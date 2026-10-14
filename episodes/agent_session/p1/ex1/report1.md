# Agent session p1 ex1 — step 1: launch ownership

Date: 2026-10-14. Not deployed yet (step 4).

## The defect

`wfexec.launch` spawned the harness, wrote the prompt and then called
`wfe exec started`. If that call (or the prompt write) failed, `tick` recorded
`exec finished --exit-code 127` and forgot the process, which was still alive
and working: the attempt was ended and the slot freed while a writer ran. If
the executor itself died between the spawn and `started`, the process kept
working, unregistered; after two minutes `reconcile` ended the `launching` row
as `unknown`, and a resume could start a second writer next to it.

## The fix

- **Launch gate** (agautolab `wfexec`). The executor spawns
  `python -I -c <gate> <fd> <argv…>` in its own process group. The gate reads
  from a pipe whose write end only the executor holds. It `exec`s the harness
  (same pid, same group) only when it reads `go`. The executor writes `go`
  only after `wfe exec started` has registered the pid. If the executor dies
  first, the pipe closes, the gate reads EOF and exits 75 without starting
  any agent work. A row that is still `launching` therefore cannot have a
  working process, so the two-minute `unknown` rule is sound again.
- **Ownership from the spawn.** `self.current` is set right after the spawn,
  before the process is registered. A failure after that point (registration,
  the gate write, the prompt) goes to `abandon`: SIGTERM, a bounded wait,
  SIGKILL, a bounded wait (5 s each), then a check that the group is empty
  (`killpg(pgid, 0)` after reaping the leader). Only then is
  `exec finished --exit-code 127` recorded.
- **Unconfirmed termination.** If the group cannot be confirmed gone, the
  executor keeps the process and the attempt. It returns `launch-unconfirmed`,
  retries the stop on every tick and claims nothing. Its reconcile passes
  `--holding <attempt>`, which is new in WFE (`ExecStore.reconcile(holding)`):
  an attempt the caller owns keeps the slot, whatever its row says.
- **Recording the end.** `finish()` keeps the attempt current when
  `exec finished` fails, so the next tick retries. This is how a normal end
  already behaved; it now covers abandoned launches as well.
- The prompt is now written after the gate opens, so a prompt larger than
  the pipe buffer cannot block an unopened gate. A command that does not
  exist fails before anything is spawned.
- Unchanged: the existing process-group cleanup after a normal end (SIGKILL
  to the group once the leader is gone), stop/cancel, the time budget, and
  interrupted-run recovery (`unknown`/`interrupted` holds and `run.recover`).

## Tests (stub processes and injected failures; no model calls)

`uv run pytest -q tests/test_wfexec.py -k "launch_failing or unconfirmed or dying_before or early_crash or restarted_executor"`
gave **6 passed in 27.6 s**. Three of the tests are new:

| Test | Shows |
| --- | --- |
| `test_a_launch_failing_after_the_spawn_stops_the_process_before_the_attempt_ends[started]` | `exec started` refused: when `exec finished` is called, the group is already gone. No agent work ran (marker file absent). The run is held `interrupted` and the attempt row is `ended`. |
| `…[gate]` | Same, with the gate write failing after registration. |
| `test_an_unconfirmed_stop_keeps_the_process_and_the_slot` | Group not confirmable: `launch-unconfirmed`, the attempt stays current. Reconcile with `--holding` leaves the aged `launching` row alone, so the slot stays held. Once the group is confirmable, the attempt is finalized. |
| `test_an_executor_dying_before_registration_leaves_no_writer` | A real executor subprocess is SIGKILLed while its `started` call is pending (the fake `wfe` blocks there). The gate exits by itself and the marker stays absent. The restarted executor is `idle` while the row is fresh. Once the row is aged, it is `unknown`. A person resumes, and exactly one agent run ever happens. |

The other two are the existing restart and early-crash cases, rerun because
they cross the changed launch path. `tsc --noEmit` for the workflow editor
is clean.

## Commits

- agdevworld `b331400`: `wfe exec reconcile --holding`, docs/runs.md *Launch ownership*.
- agautolab `abbf62d`: launch gate, ownership, `abandon`/`finish`, tests.
