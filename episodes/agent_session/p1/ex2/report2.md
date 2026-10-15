# Agent session p1 ex2 — step 2: a compact, reliable continuation

Date: 2026-10-15. Source:

- agdevworld `6d8d1b5` (on `399e2b2`);
- agautolab `1f1ca20`;
- pj-agdev `d00a653`.

Captured prompts and outputs are in the ignored notes
`pj-agdev/.local/agent-session-p1-ex2/step2/`. They come from `capture.ts`,
a p1-shaped run: default documentation policy, the real node catalog, plan →
implement → result check, and the same scripted answers as p1.

## What changed

**WFE (`server/continuation.ts`, `server/exec.ts`).** The two prompt paths
are now separate. `wfe exec claim` returns `brief`, the whole "This
attempt" text a new session gets, as before. It also returns
`continuation`, the compact text an ordinary resume gets; this is null for
a new session and for a reconstruction.

The continuation contains:

1. **Why the attempt runs and how the previous one ended.**
   - A previous attempt that did not end normally adds a line: its effects
     are uncertain, so files (`git status`/diff) and the record are checked
     before any command is repeated.
   - A recovery's finding, a person's resume and the project's commits
     since the copy's base are kept as before.
2. **The delta, with an explicit seq boundary.** One line per record entry
   after the session's previous attempt's `attempt.begin` that the previous
   attempt did not write (by the entry's `attempt` id). The executor's
   bookkeeping is left out, and the list is capped at 30 lines, with a
   refresh command for the rest. Holds, resumes, recoveries,
   cancellations, decisions, adjustments by others, answers, instructions
   and presentation changes all appear here.
3. **The compact state at seq N**:
   - hold or cancellation;
   - nodes ready, running, waiting (for whom), failed or blocked;
   - presentations not stopped;
   - the revision in force;
   - decisions.
4. **Pending input in full, selected by state.**
   - Answers recorded and not taken up: exact text, question id, answer
     index, author, and the question they answer.
   - Instructions not acknowledged.
   - Delegation facts not taken up.

   No delivery cursor is involved. An input sent in a prompt whose attempt
   failed is still pending, so the next continuation repeats it until it is
   taken up.
5. **Only new documentation-step instructions.** These are steps running,
   ready or waiting now that were not active when any attempt of this
   conversation began since its last whole bootstrap. Earlier attempt
   states are computed with the reducer's `replay`.
6. **A closing line**: what is unchanged and not repeated, the snapshot
   seq, `wfe run state <run> --since <seq>`, and `wfe run show` for the full
   record.

The continuation leaves out the working-copy and workspace paths, the
catalog, the documentation policy and steps already briefed. It does not
ask the agent to read the history.

**`wfe run state <run> [--since <seq>]`** is the compact read. It prints
the same state and pending input, plus one line per entry after a seq.
`wfe run show` and `--history` are unchanged and remain the full record
and a diagnostic.

**The whole brief (new and replacement sessions).**

- The original input (and `original-input.md`) is named on every attempt,
  not only on the start.
- The current `plan.md` is named, with the question it was agreed in.
- Pending answers are given in full.

These are shared correctness fixes: both policies get them.

**Compaction.** At claim, a resume of the current session checks the logs
of every attempt since the session last got the whole bootstrap (its first
attempt, or a reconstruction). Either of these triggers a reconstruction:

- a `compact_boundary` event in any of those logs;
- a log that is missing or unreadable, because then a compaction cannot be
  ruled out.

The attempt then resumes the same conversation with the whole bootstrap.
The reason is recorded before launch as `settings.session.reconstruct`,
which is validated and allowed only on a resume. Cache expiry alone changes
nothing.

**Executor (agautolab `wfexec.py`).**

- An ordinary resume gets the attempt-identity header and the claim's
  `continuation`. "Begin by reading … `--history`" is gone from it.
- A reconstruction gets a header ("the run's instructions again", with the
  recorded reason), then the contract, guides and whole brief.
- A new session is unchanged. That includes its `--history` reading
  instruction, so the `fresh` arm keeps p1's reconstruction behaviour apart
  from the shared brief fixes.
- A session found missing before launch still switches to `new` and gets
  the whole bootstrap.

**wfrun guide (the common contract).**

- The tool list names `wfe run state`; `--history` is no longer presented
  as the way to read the state.
- The node-boundary check is `wfe run state <run>`.
- A new fact says what a resumed attempt is given: what it was told before
  still applies; a document is reread when it changed or the agent is
  unsure of it, not because an attempt began; a compacted conversation gets
  the instructions again; finding a summary where the beginning was means
  reading the state, the request and `plan.md`.
- Changing the guide changes the contract hash, so existing retained
  sessions are replaced at their next claim, as p1's rules say.

**Docs**: agdevworld `docs/sessions.md` *Continuation* (rewritten),
`docs/runs.md` (operations table, node boundaries).

## Before and after (p1-shaped run, same answers)

| Prompt | p1 (characters) | p1 ex2 (characters) |
| --- | --- | --- |
| Continuation after the plan answer | 3,546 + a `--history` read (3.8 k) | **1,204** (answer included; no read needed) |
| Continuation after the correction | 2,938 + `--history` (3.6 k) | **1,304** |
| Continuation after the acceptance answer | 3,235 + `--history` (1.2 k, tailed) | — (same shape, ≈1.3 k) |
| New-session brief ("This attempt") | 3,158 | 3,016 (+ input/plan lines on later attempts, + pending answers) |
| `wfe run state` at the correction | — | 686 |
| `wfe run show --history` at the same point | 5,163 | 5,163 (unchanged; on demand only) |

The p1 numbers are from the retained transcripts; the ex2 numbers are from
`capture.ts`. The continuation after the correction, verbatim apart from
times:

```text
Run: rts / improve/run-001 (record devdocs/runs/improve/run-001/run.json).
Why this attempt: resume: answer 0 to q2 recorded
The previous attempt amv994txh-730138 ended: exited.

Recorded since your previous attempt began (after seq 7), not by it (1):
  17 question.answer by Trial Person (question q2): 見出しを「選択中」にして、兵種ごとの行は数の多い順に並べてください。

State at seq 18: in-progress.
Nodes: 3 completed; ready: (none); running: (none).
Waiting: human-check — question q2; next move: autolab (answer recorded, not yet taken up).

Answers recorded and not yet taken up (1), exact text; take each up before acting on it (wfe run take-up <run> <question>):
- q2 answer 0 from Trial Person (scripted trial input) (recorded by Trial Person) via cli at …; your question from human-check: "表示を確認してください。"
    見出しを「選択中」にして、兵種ごとの行は数の多い順に並べてください。

Unchanged and not repeated: the contract, the guides, the node catalog (wfe run nodes improve/run-001) and the documentation policy you were given. This is the record at seq 18; wfe run state improve/run-001 --since 18 shows what is recorded after it, wfe run show improve/run-001 the full record.
```

The executor puts its identity header (≈440 chars, unchanged from p1) in
front of this text.

## What did not change

- p1/ex1 session identity, fencing and ownership rules, which are recorded
  before launch:
  - attempt fencing;
  - `session-mismatch` refusal;
  - the launch gate;
  - one writer;
  - replacement on a configuration change, on retirement, or when the
    conversation is missing from the store.
- Holds, recovery budgets and write-time refusals (`instructions-pending`,
  `attempt-stale`).
- Records, plans and reports. Nothing durable was reduced.

No compaction, model change or session splitting was added.

## Tests run for this step

- WFE: `node --test test/continuation.test.ts test/sessions.test.ts
  test/instructions.test.ts test/docsteps.test.ts` → 34 passed. Step 3
  maps the new cases to the required boundaries.
  - The p1 session tests' claim helper now creates the attempt log, as the
    executor does. Without a log, a resume is correctly a reconstruction.
- agautolab: `pytest -k "prompt_kinds or resumes_one_explicit or
  fresh_policy or missing_session or changed_guide"` → 5 passed. This runs
  the real executor against the working-tree WFE, with a stub harness.
- `tsc --noEmit` clean.

## Note on activation

The executor reads the wfrun guide from its checkout at every claim. The
guide naming `wfe run state` was therefore live from the agautolab commit,
before the WFE release that adds the command. At that time
`wfe observe` showed only jevlab's held run and nothing queued. The release
follows step 3.
