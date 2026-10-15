# Agent session p1 ex2 — final report

Plan: [plan.md](plan.md). Step reports:

- [report1](report1.md): audit;
- [report2](report2.md): continuation contract;
- [report3](report3.md): boundaries;
- [report4](report4.md): RTS pair;
- [report5](report5.md): phase boundaries;
- [report6](report6.md): deployment.

p1's own results ([../report.md](../report.md),
[../report5.md](../report5.md)) are unchanged.

## Outcome

- **A retained session's continuation is compact and complete.** It
  carries what the conversation cannot know:
  - the record entries since its previous attempt began;
  - the compact state;
  - every pending answer, instruction and delegation fact, in full.

  It repeats nothing stable and asks for no history read.
- **The cost picture is unchanged.** In one RTS pair the resumed attempts
  read no history and appended 1.7–1.9 k-char continuations, about half of
  p1's. Whole-run cost was still `run` $3.02 against `fresh` $1.84. Most of
  that gap is one extra correction attempt in the `run` arm. The duplication
  removed is a few hundred tokens per resume, against 57–151 k tokens of
  history carried into every step.
- **Recommendation: keep `fresh` as the default.** `run` stays selectable.
  This is one pair, so the result is a diagnostic, not an average. The
  evidence did not support a change.

## Audit inventory (step 1)

| Source | Finding | Action |
| --- | --- | --- |
| Harness system/tools/skills | ≈20–32 k tokens, a shared cache read; the harness adds a skill listing and deltas on resume | out of scope (harness) |
| Bootstrap (contract 17.8–21.5 k chars, project guide 4 k, brief 3.2 k) | once per session | unchanged for new sessions |
| p1 continuation (2.9–3.5 k chars) | ≈2.0 k stable chars repeated: paths, catalog, policy, step instructions already given. **No answer text.** | stable parts omitted; answers added |
| "Begin by reading `wfe run show --history`" | a cumulative read every attempt (1.2–3.8 k chars retained, 2.8–10.3 k fresh), needed only because the answer was missing | removed from continuations; `wfe run state` added |
| `wfe run show` without `--history` | still cumulative (every node, question and answer) | kept as the full record; not the routine read |
| Write-command responses | already 2–3 lines | unchanged |
| Re-reads in retained attempts | none of plan, request or definition; source re-reads were verification | kept |
| Compaction | not detected anywhere; the bootstrap lives in the first user message | detected at claim, which then reconstructs explicitly |
| Cache expiry | the conversation is intact | no action, by design |

## Implemented (step 2)

- **WFE** (`server/continuation.ts`, `server/exec.ts`, agdevworld `6d8d1b5`):
  - The claim returns a `continuation` for an ordinary resume. It contains:
    - why the attempt runs, and how the previous attempt ended, with an
      "effects uncertain, check before repeating" line after an
      abnormal end;
    - recovery, resume and project-commit facts;
    - the delta after the previous attempt's begin seq, excluding the
      attempt's own entries, capped at 30 lines;
    - the compact state at seq N;
    - pending input in full, selected by state;
    - documentation-step instructions not yet briefed in this
      conversation;
    - a refresh line.
  - `wfe run state <run> [--since <seq>]`.
  - A resume of a session with a `compact_boundary`, or an unreadable log,
    since its last bootstrap gets the whole bootstrap, recorded as
    `settings.session.reconstruct`.
  - New-session briefs name the input and the agreed `plan.md` on every
    attempt and carry pending answers.
- **Executor** (agautolab `1f1ca20`): the continuation for an ordinary
  resume; the whole bootstrap with a stated reason for a reconstruction.
  The wfrun guide names `wfe run state` and states what a resumed attempt
  is given.

## Before / after

| | p1 retained | ex2 retained |
| --- | --- | --- |
| Continuation after the plan answer | 3,546 chars + `--history` read | 1,682 chars (answer included) |
| Continuation after the correction | 2,938 chars + `--history` | 1,828 chars |
| Continuation after acceptance | 3,235 chars + `--history` (tail) | 1,884 chars |
| Tokens appended at resume | +1.27–1.94 k | +1.28–1.53 k (the rest is the harness's own additions) |
| History reads in resumed attempts | 3 | 0 |

## Tests

- WFE `test/continuation.test.ts`, 7 cases: ordinary resume; several inputs
  and a failed launch; input after the snapshot and the write-time
  refusal; hold, resume, decision and cancellation; documentation steps;
  compaction and an unreadable log; output size on a long history.
- `sessions.test.ts`: 7.
- Release gate `npm run check`: **344/344**.
- agautolab `tests/test_wfexec.py`: **21 passed**, including prompt kinds
  and a replacement prompt carrying the request and the answer.

The full mapping is in [report3](report3.md).

## Trial (step 4, one pair, deployed revision)

| | fresh | run-optimized |
| --- | --- | --- |
| Attempts / steps | 3 / 85 | 4 / 99 (one session) |
| Cached reads / writes | 5.04 M / 135 k | 10.24 M / 130 k |
| Output / active time | 30 k / 517 s | 43 k / 939 s |
| Reported cost | **$1.84** | **$3.02** |
| Implementation attempt | $1.02 (new, 43 k start) | $1.03 (resumed, 57 k start) |
| Correction | handled in the implementation attempt: its design already met it | separate attempt, real repair (r2), $1.31 |
| Final attempt | $0.39 (20 steps, reads the 9.6 k-char history) | $0.39 (11 steps from 146 k) |
| Quality | 4 stats for both sides; 39/39 tests; accepted | 4 stats for both sides, kept through the correction; 34/34; accepted (re-check from an added node was recognised by meaning) |

Total: $4.86 of the $15 cap. There was no compaction, no cache expiry and
no stale-state failure. Costs are list-price estimates on a subscription,
not bills.

## Accounting

| Item | Size in this pair | Status |
| --- | --- | --- |
| Total carried history | resumed steps started at 57 k, 105 k and 146 k against 43 k fresh; 10.2 M against 5.0 M cached reads | inherent to `run`; not reduced by deduplication |
| Avoidable duplication (stable prompt material + mandatory history read) | p1: ≈2 k chars + 1.2–3.8 k chars per resume. ex2: none in the continuation, no history reads | removed |
| Cache expiry | none occurred (answers were immediate); p1 measured the 65-minute case | unchanged; not recreated |
| Extra work | the `run` arm's correction attempt ($1.31); the agents' designs diverged | work variation, not policy |

## Phase boundaries (step 5)

A plan → implementation boundary was designed, not implemented:

- a policy value `phase`;
- one recorded replacement at the first claim after `_plan` completes;
- a handoff that adds every taken-up answer and acknowledged instruction
  verbatim to the bootstrap.

Estimated from this trial, it would save ≈1 M cached reads on a run of this
shape. It is **untested**, and nothing here claims a saving for it. The
evidence needed is listed in report5.

## Versions

- Deployed:
  - agdevworld `a642e27` (previous `399e2b2`);
  - agautolab `1f1ca20` code, `9382b1b` tests (executor restarted 17:11
    JST);
  - pyagag and agobserver unchanged.
- Harness: claude_code 2.1.283, `claude-sonnet-5`.

## Remaining limits

- **One pair, one task, diverging designs.** It cannot establish an
  average effect. A negative or inconclusive cost result is valid, and this
  is one.
- **Harness additions on resume are outside WFE's control.** In this
  trial these were a skill listing, a "response was cut off" nudge, and a
  stale background-task notification from the previous attempt.
- **The delta is bounded noise.** It lists presenter observations and
  answers that also appear in full in the pending section.
- **Compaction is detected only between attempts.** A compaction inside an
  attempt relies on the harness summary, the guide's fact ("read state,
  request and plan.md if you find a summary"), and the write-time
  refusals.
- **The environment differs from p1.** The trial ran without the Omni
  session's `CLAUDE*` environment, so its absolute figures are not
  comparable with p1's. Within the pair they are.

## Deus Ex Machina note

None. No in-system agent's work was done by the Omni Agent. The trial inputs
were scripted and labelled.
