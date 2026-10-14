# Agent session p1 — step 4: waits, recovery and Observer

Date: 2026-10-14. Source: agdevworld `d317b0f`, pj-agdev `d1ce1d5`
(agobserver), agautolab `c08cb5d`, pyagag `4dde895`. Evidence (driver,
per-phase logs, attempt logs, measurements) is in the ignored notes
`pj-agdev/.local/agent-session-p1/step4/`.

## Semantics (unchanged where they already held)

- **A wait ends the attempt.** A question or a hold ends the process and
  releases the slot, under either policy. The conversation waits in the
  harness store; no process is kept alive to preserve it. An answer queues
  the next interval through the existing mechanism, once per answer.
- **The budget is per interval.** The 4 h `timeout_seconds` applies to each
  active attempt. The age of a retained session is not a timer anywhere.
- **Recovery** works as in observer p3. Under `run`, the continued attempt
  resumes the retained conversation (when its configuration is unchanged and
  it is stored). The brief carries the recovery's finding ("look at your copy
  and the record before repeating anything") and the current record state.
  Replacing or retiring a session resets no recovery budget; only a person's
  control does.
- **Failed resume.** A resume that launched and then failed counts as an
  interrupted attempt (non-zero exit → `interrupted` hold → Observer →
  recovery). It is not a reason for an immediate fresh retry. A conversation
  found missing *before launch* is replaced at once (nothing ran in it). One
  that goes missing after launch is caught by the next attempt's store check
  and replaced then.
- **Unsupported harness vs lost data.** These are distinct. `run` on an
  incapable profile is refused at claim (step 2). Missing conversation data
  on a capable host is a recorded replacement.
- **Foreground waits** are unchanged. Session persistence keeps no child
  process alive (the wfrun guide's foreground-wait rule stands). Future
  external or delegate waits keep their own identities and deadlines.
- **Persistent processes** were not introduced. Waits are not served by a
  live process, and no idle state, input channel or Observer precedence change
  was needed.

## What changed for observation

- `wfe observe` now reads the session from the record:
  - per run, `session`: `{policy, current {id, began, attempts, attached,
    mismatch}, count, ended[≤3] {id, state, at, by, reason}}`;
  - per attempt, `process.session`: `{id, resumed}`.

  The log's `session` stays as supporting evidence. Activity, open tools and
  end events are still read from the attempt's own log. A resumed claude
  invocation does not replay earlier events into the new log, so they stay
  attempt-local.
- Observer keeps `session` and `process.session` in each incident's evidence.
  Incidents stay per attempt: two intervals of one session are two episodes
  (observer p3's test). A new test checks that the recorded session is in the
  evidence and that the incident key is the attempt.
- The recovery request names the session policy, the session the ended
  attempt ran in (new or resumed), and the current session. Under `run` it
  adds how to retire the conversation (`wfe run session <run> --retire`) when
  it should not be carried on (for example, a resume keeps failing, or the
  conversation holds a wrong belief).
- agobserver `docs/workflow.md` *Sessions* now states these facts in place of
  the p3 forward note.

## End-to-end trial (real harness, private area)

Setup: claude_code 2.1.283 / `claude-sonnet-5`. Workflow: talk (ask the
language) → do (write `greeting.txt`). `keep` = `run` policy, `anew` =
`fresh`. After each phase the driver ran Observer's own `wfclassify.classify`
over the area's `wfe observe --json` (executor staleness disabled, because
the trial executor runs in bursts).

| Phase | What happened | Result |
| --- | --- | --- |
| **Wait A, run B, resume A** | A asked and ended (session 9315bad2, no process). B was requested, asked, answered and completed meanwhile in its own session f5670c30. Then A was answered and resumed. | Observer: A `awaiting-person`, no recovery, alive=false, session from the record. A's second attempt resumed 9315bad2 (2 attempts, 1 session). Both runs completed. The slot was never held by a waiting run. |
| **Interruption (`run`)** | keep/run-003: attempt 2 (resumed 68ba5cdd) was SIGKILLed after its first tool calls. | Hold `interrupted` (signal). Observer: `interrupted`, family `attempt`, episode = the killed attempt, recover=true. A recovery was recorded (`wfe run recover … --by trial-recovery`). Attempt 3 **resumed the same session** (3 attempts, 1 session). It first ran `wfe run show --history && git status && ls`, found the answer not yet taken up and no file, then took it up and completed. The killed command was not replayed. |
| **Interruption (`fresh`)** | anew/run-001: same kill. | Same hold and finding. The recovered attempt started a **new** session. The record shows 3 sessions, the first two ended with "fresh policy". Completed. |
| **Missing storage** | keep/run-004: after attempt 1 the trial moved the transcript out of the claude store. Then the answer. | The executor's store check found it missing and recorded the replacement before launch: `864287a9 replaced: missing on agstudio…; nothing ran in it`, new `e95c9150`. The new session got the full bootstrap and completed. No resume of a missing id was attempted. |
| **Human hold** | keep/run-005: attempt 1 asked; the person stopped the run during the wait; then answered. | The answer did **not** resume it (idle, hold `stopped`, still 1 attempt). Observer: `respected-hold`, no recovery. After the person's resume, attempt 2 resumed the same session dc7ed5c4 and completed. |
| **Cancellation** | keep/run-006 cancelled while waiting; an answer was posted afterwards. | Nothing was launched (still 1 attempt). The answer is recorded as before; cancellation refuses further attempts. The session record stays `open` on the cancelled run, which is harmless: no attempt can begin. |

Across all phases there was no duplicate writer (one slot, one process per
run at a time), no lost answer, no false stall for a waiting retained
session, and no bypass of a human hold. Automated coverage of the same paths:
the agautolab wfexec stub tests (early crash + executor restart, missing,
retire) and WFE `sessions.test.ts` (observation during the wait and the
resume).

**Measurements from this trial** (reported client-side estimates; resumed
rows are cumulative deltas; `measure.py` in the ignored notes):

- New-session first request: about 51.8 k tokens (20.2 k written, 31.6 k
  read).
- Resumed first request: 56.5–60.2 k tokens, of which only 1.0–1.7 k are
  written.
- Second attempts: resumed $0.127 / $0.169 / $0.129, against $0.235–0.260
  for new sessions doing the same remaining work (a replacement, and a fresh
  attempt after an interruption).
- A killed invocation saves no totals: its own spend is unavailable and also
  absent from the session's next cumulative figure.

Trial total: about $2.3 reported.

These are indicative only; step 5 is the matched comparison on a real RTS
task.
