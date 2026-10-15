# Agent session p1 ex2 — step 1: what a continuation receives and reads

Date: 2026-10-15. Sources read:

- the executor's `prompt()` / `continuation()` (agautolab `f66162a`,
  `src/agautolab/wfexec.py`);
- WFE's `brief()` and its helpers (agdevworld `394def3`, `server/exec.ts`,
  `shared/catalog.ts`);
- `wfe run show` (`cli/run.ts` `describeRun`);
- the wfrun guide;
- the shared harness argv (pyagag `4dde895`).

Evidence comes from p1 step 5:

- the 16 attempt stream logs;
- the two retained session transcripts, which still hold the exact prompts
  the agent received.

The analysis scripts and their output (`audit.py`, `transitions.py`,
`pair1.json`, `transitions.txt`, `continuation-breakdown.txt`) are in the
ignored notes `pj-agdev/.local/agent-session-p1-ex2/step1/`.

## Before-context breakdown (p1 step 5)

**New session, first request: 53.7 k tokens.**

| Part | Size | Supplied |
| --- | --- | --- |
| Harness system prompt, tools, skill listing | ≈31.6 k tokens, a cache read from the first request on | by the harness, shared across sessions |
| Common contract (wfrun guide) | 17,790 chars | once per session, in the first user message |
| Project guide `agent/review.md` | 4,026 chars | once per session |
| "This attempt" (identity, request, catalog, documentation policy, step instructions) | 3,158 chars | once per session |

**Resumed session, first request: 75.9 k / 147.6 k / 176.0 k tokens (pair 1).**
Only 1.3–1.9 k tokens of each is newly appended:

| Transition (pair 1, retained) | Context at end of previous attempt | First request of next attempt | Appended |
| --- | --- | --- | --- |
| attempt 1 → 2 | 73,923 | 75,859 | +1,936 |
| attempt 2 → 3 | 146,302 | 147,568 | +1,266 |
| attempt 3 → 4 | 174,267 | 175,966 | +1,699 |
| pair 2: 1 → 2, 2 → 3, 3 → 4 | 72,891 / 93,442 / 126,047 | 74,789 / 94,727 / 127,675 | +1,898 / +1,285 / +1,628 |

The appended part is the continuation prompt, 2.9–3.5 k characters, plus a
few hundred tokens the harness adds itself on resume:

- a skill listing (≈1.3 k chars);
- deferred-tool and MCP-instruction deltas.

Those harness additions are not ours to change. Everything else is history
the model necessarily still reads: the earlier attempts' tool results,
edits and reasoning. It grew by 70 k tokens in the retained implementation
attempt alone.

**The continuation prompt, by section** (characters; p1's six
continuations):

| Section | Chars | Changes externally? | Verdict |
| --- | --- | --- | --- |
| Header: new attempt id, earlier ids ended, processes gone, memory is context | 444 | the id, yes | **keep**: identity and fencing facts |
| Identity: run, agent, working copy path, project workspace path | 680 | no, fixed for the run's life | **omit**: keep only the run reference |
| Why / previous outcome / waits / ready | 209–217 | yes | **keep**, as the compact state |
| Node catalog (meaning + each template) | 1,030 | no, frozen at creation | **omit**: `wfe run nodes` on demand |
| Documentation policy | 249 | no, frozen | **omit** |
| Step instructions for `_plan`, already given in attempt 1 | 856 | no | **omit when already briefed in this session**; include steps newly ready |
| Presentations / definition revision | 246–543 | yes | **keep**, compacted into the state |
| "Begin by reading … `wfe run show <run> --history`" | 74 (+ what it fetches) | — | **remove** (below) |
| **Missing:** the answer that launched the attempt | 0 | yes | **add**, as exact text with ids and author |

About 2.0 k of each 3.0–3.5 k-character continuation repeats material that
cannot have changed.

## What agents read after resumption

From the retained attempts (`transitions.txt` lists every call that repeats
one made in an earlier attempt of the same session):

- **The full run, every attempt.** Each retained attempt began with
  `wfe run show <run> --history` (3.8 k, 3.6 k (`| tail -40`) and 1.2 k
  (`| tail -15`) chars), as the prompt told it to. It needed this read
  because the prompt only said "resume: answer 0 to q1 recorded": the
  answer's words were nowhere else. Without `--history` the output is still
  cumulative: `describeRun` lists every node, every question with every
  answer (400 chars each), artifacts, files and the last five history
  entries. Fresh sessions read the same command: 2.8 k → 4.2 k → 8.6 k →
  10.3 k chars across improve-fresh/run-001's four attempts. It grows with
  the run.
- **No re-reading of stable documents in retained sessions.** The resumed
  attempts did not reread `braindump.md`, `plan.md`, the definition, the
  contract or help pages they had already read. Help reads were for new
  subcommands (`take-up`, `adjust`, `decide`). Fresh sessions reread
  `plan.md`, the definition, reports and help, which is the reconstruction
  they exist for.
- **Repeated source reads are verification, not waste.** For example,
  `game/main.js` was read again in attempts 2 and 3 after edits and before
  the correction. Removing them would remove verification of actual state.
  They stay.

## Tool responses

- Write commands (`start`, `complete`, `take-up`, `ask`, `ack`) answer in
  two or three lines with the seq and what is ready or waiting. They are
  already compact.
- `wfe run show` is the only oversized response for its purpose. An agent
  that wants "what changed and what is pending" gets the whole record.
- No other command dumped history.

## Session maintenance

- **Compaction**: none occurred in p1 (largest context 176 k). The harness
  emits `compact_boundary` and a synthetic summary. The contract, guides and
  original requirement sit in the first user message, which is exactly what
  a summary may drop. Nothing in WFE or the executor notices a compaction
  today: a later continuation would assume the conversation still holds them.
- **Cache expiry**: pair 2's 65-minute wait rewrote the whole 95 k prefix.
  That is a property of the cache, not of the conversation. No bootstrap is
  needed, and none will be added.
- **Configuration replacement**: a changed contract, guide, agent, profile or
  backend already starts a new session with the full bootstrap (p1). Unchanged.
- **Harness-injected files**: no CLAUDE.md exists above the run copies (checked
  for `~/.claude`, the workspace root and `pj-agdev`). The trial sessions
  inherited the account's skills and claude.ai MCP listing in both arms.

## Optimizations to implement (step 2)

1. **The continuation carries the pending input itself.** Every answer
   recorded and not taken up goes in, with its exact text, question id,
   answer index, author and the question it answers. The same applies to
   instructions not acknowledged and delegation facts not taken up. The
   selection is by state, not by a cursor. Input stays in every
   continuation until it is taken up, which bounds the repetition and loses
   nothing to a failed launch.
2. **A delta since the session's previous attempt began.** One line per
   record entry after that attempt's `attempt.begin` seq that the attempt
   did not write itself (holds, resumes, recoveries, cancellations,
   decisions, adjustments, answers, instructions, presentation changes). It
   is capped, with a refresh command for the rest. The snapshot names its
   seq.
3. **A compact current state**: hold, nodes ready, running, waiting and
   failed, open presentations, the definition revision in force, decisions.
4. **No stable material**: identity paths, catalog, documentation policy,
   and step instructions already briefed in this session are left out. A
   step that becomes ready or running for the first time in this session
   still gets its instructions.
5. **No mandatory history read.** A compact read, `wfe run state <run>
   [--since <seq>]`, prints the same state, pending input and delta. The
   continuation and the guide's node-boundary check point to it.
   `wfe run show` stays the full record, and `--history` stays a
   diagnostic.
6. **Interrupted previous attempt**: the continuation says that effects are
   uncertain and names what to check (files, `git status`/diff, the record)
   before repeating a command.
7. **Compaction detection**: a resume whose session compacted, or whose
   compaction cannot be ruled out (an unreadable attempt log), gets the
   whole bootstrap in the same session: an explicit reconstruction. The
   decision is recorded on the attempt.
8. **New and replacement sessions** keep the whole bootstrap. Their brief now
   always names the original input and the current `plan.md` (today only
   the start attempt names the input). Pending input is given in full there
   too. These are shared correctness fixes and apply to both arms.

## Correctness checks retained

- Attempt fencing, `session-mismatch`, launch ownership, single writer (ex1).
- Write-time refusals: `wfe run start` and the final completion refuse while
  an instruction is unacknowledged; a stale attempt's reports are refused.
- Holds, cancellation and recovery budgets are decided by the record, never
  by the prompt.
- Configuration change → new session → whole bootstrap.

## Newly appended duplication vs old history

Per continuation, the avoidable part is small:

- ≈2.0 k chars of stable prompt material (≈0.5–0.6 k tokens);
- one mandatory full-record read (1–3 k tokens, growing with the run).

That is about 1.5–3.5 k tokens against first-request contexts of 75–176 k.
Each step of the attempt then reads it again from cache. The carried
history, which p1 identified as the cost driver, is not addressed by
removing duplication: it is what a retained session *is*. Step 4 measures
whether the optimization changes the outcome. This audit predicts at most a
few percent of a retained run's input.
