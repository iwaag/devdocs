# Agent session p1 ex1 — step 3: focused verification and report correction

Date: 2026-10-14.

## The required regressions

All of them use stub processes and injected failures. There are no paid
model calls, no long sleeps, and no RTS or broad suite rerun. Waits are
bounded polls on records, files or process groups. The executor test that
"dies" is a real subprocess that gets SIGKILL while its `started` call is
pending.

| Fast regression | Test | Result |
| --- | --- | --- |
| Spawn succeeds, `exec started` fails | agautolab `test_a_launch_failing_after_the_spawn_stops_the_process_before_the_attempt_ends[started]` (and `[gate]`) | When `exec finished` was called, the child group was already gone. No agent work ran. The run is held `interrupted`. |
| Executor disappears before registration | `test_an_executor_dying_before_registration_leaves_no_writer` | The gate exited by itself and did no agent work. The restarted executor stayed `idle` while the row was fresh, ended it `unknown` once aged, and after a person's resume, exactly one agent run ever happened. |
| Harness reports another session ID | `test_a_reported_session_mismatch_stops_the_attempt_and_nothing_continues_it`; WFE `mismatch (p1 ex1) …` | The attempt was stopped (group gone), both ids were kept, and the hold names the mismatch. Resume, recover and claim's `sessionFor` refuse with the `--retire` command. |
| Attachment write fails once | `test_a_failed_attachment_record_is_retried` | Tried again; the record ends attached and the run waits normally. |
| Explicit retirement after mismatch | `test_retiring_a_mismatched_session_permits_a_new_one_and_keeps_the_recovery_budget`; WFE `mismatch (p1 ex1) …` | Each permitted attempt selected a new session. The third recovery was refused by `recover-budget` after two retirements. The retirement lifted no hold. |

Additionally, `test_an_unconfirmed_stop_keeps_the_process_and_the_slot`
covers a group that cannot be confirmed gone: the attempt is kept and the
slot stays held. WFE `mismatch under fresh` covers the `fresh` side.

## What was run

Only the affected cases, each once after the last change:

- agautolab, step 1 and 2 boundaries: **7 passed in 33.8 s**.
- agautolab, the existing p1 session cases that cross the rewritten
  `attach()`: **6 passed in 38.3 s**.
- WFE `node --test test/sessions.test.ts`: **7/7**. `tsc --noEmit`: clean.
- agobserver `-k recorded_session`: **1 passed**.

The 88-second executor suite and the full project suites were not rerun.
The release script runs its mandatory checks once, in step 4.

## Report corrections (p1)

I reconciled the totals against the retained measurements
(`measure.json`): improve-fresh/run-001 $2.746, improve-run/run-001
$4.914, improve-run/run-002 $3.158, improve-fresh/run-002 $3.241. They add
up to **$14.058**: fresh $5.987 and run $8.072. The trial driver's own
running total ended at **$14.06** (`pair2.out`).

- **report5.md:** "Used: $11.06" now reads **$14.06**, with a correction
  note. The per-run and per-attempt figures were right and are unchanged.
- **report6.md:** its trial total said "~$15" for steps 3–5, which was built
  on the wrong figure. It now reads about **$17.6** (step 3 ≈ $1.19, step 4
  ≈ $2.3, step 5 $14.06).
- **report5.md and report.md:** compaction and per-node session splitting
  were described as "what would make `run` cheap". They are now untested
  improvement candidates, not established savings. No compaction occurred
  in any trial.
- **report.md:** a correction note was added. The `fresh` default and the
  small-sample qualification stay as they were. The comparison figures
  ($8.07 vs $5.99) were right.
- **README_DEV:** quotes only the correct comparison figures and makes no
  compaction claim, so it is unchanged.
