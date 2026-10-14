# Agent session p1 ex1 — step 2: a mismatched session is stopped and refused

Date: 2026-10-14. Not deployed yet (step 4).

## The defect

When the harness's init event named a session other than the one the attempt
was launched with, `wfexec.attach` recorded it (`mismatch: true`), logged a
line and let the attempt go on working in that conversation. A later claim
under `run` resumed by the intended id as if nothing had happened. A failed
`exec session --attached` call set `attached = True` anyway ("said once"),
so the harness's report was dropped.

## The fix

**Executor** (agautolab `wfexec`):

- `attach()` reads the init event once and keeps the reported id
  (`Running.reported`).
- If the reported id differs from the intended one, the attempt is ended by
  the host. `abandon()` (step 1's path, now shared) sends SIGTERM, then
  SIGKILL, and waits until the group is confirmed gone. The end is then
  recorded with the real exit status plus
  `--interrupted "session mismatch: launched in session X, the harness
  reported Y"`. A group that cannot be confirmed gone keeps the slot, as in
  step 1. The return value was renamed `stop-unconfirmed` (step 1 called it
  `launch-unconfirmed`), and a failed launch now ends with its real status
  and `--interrupted "launch failed: …"` instead of exit code 127.
- A failed attachment record stays pending and is retried on every tick. The
  end report also carries `--attached <id>`.

**WFE** (agdevworld):

- `wfe exec finished --interrupted <reason>` ends the attempt `interrupted`
  with that reason, whatever the exit code. The run is held.
- `--attached <id>` records a reported id before `attempt.end`. If that
  record fails too, the id is written into the end's detail.
- `unverifiedSession()`: the current session under `run` with `mismatch`.
  `run.resume`, `run.recover` (reducer, code `session-mismatch`) and claim's
  `sessionFor` all refuse it. The reason names both ids and the attempt, and
  gives the next step: look at the run and what the attempt did, then
  `wfe run session <run> --retire --reason "<what you found>"`.
- This uses the existing retirement route. Retiring lifts no hold and needs
  no live process. It is not a person's control, so it resets no recovery
  budget. The retired entry keeps `mismatch` as evidence. Under `fresh`
  nothing reuses the session, so only the hold applies.

**Observer** (pj-agdev `agobserver`): the recovery request now says when the
current session is UNVERIFIED. It names both ids, states that continuing is
refused until the session is retired, and gives the command. `wfe observe`
already carried `current.mismatch`.

## Tests (stub processes and injected failures; no model calls)

| Test | Result |
| --- | --- |
| agautolab `test_a_reported_session_mismatch_stops_the_attempt_and_nothing_continues_it` | The stub announces `other-<id>` and then sleeps. The attempt is stopped and its group is gone. Both ids are kept (`mismatch: true`). The hold is `interrupted` and names the mismatch. `scope` is still pending, so no work was done in the unverified conversation. `run resume` and `run recover` are both refused with `session-mismatch` and the `--retire` command. |
| agautolab `test_retiring_a_mismatched_session_permits_a_new_one_and_keeps_the_recovery_budget` | Three mismatching attempts. After each, Observer retires the session, the hold stays, and Observer recovers. Every permitted attempt got a new session id. The third recovery is refused by `recover-budget`: two retirements reset nothing. |
| agautolab `test_a_failed_attachment_record_is_retried` | The first `--attached` record is refused. It is tried again, the record ends with `attached == id`, and the run waits normally. |
| WFE `mismatch (p1 ex1): … refuse the session until it is retired` | `finished --interrupted --attached` records both ids and holds the run. Resume and recover are refused, and `sessionFor` refuses with the run-specific command. After retirement, recovery is permitted. The next claim is a new session that `replaces` the retired one, and `mismatch` stays on the retired entry. |
| WFE `mismatch under fresh …` | An exit 0 that the host interrupted still holds. Resuming is permitted, and the next attempt gets a new session. |
| agobserver `test_recorded_session_is_evidence_and_reaches_the_recovery_request` | Extended: the request names an UNVERIFIED session with both ids and the retirement command. |

Commands and results:

- `uv run pytest -q tests/test_wfexec.py -k "mismatch or failed_attachment or launch_failing or unconfirmed or dying_before"`:
  **7 passed in 33.8 s**. These are the step 1 and step 2 boundaries.
- `-k "run_policy or fresh_policy or missing_session or changed_guide or early_crash"`:
  **6 passed in 38.3 s**. These are the existing p1 session cases, which
  cross the rewritten `attach()`.
- `node --test test/sessions.test.ts`: **7/7**.
- `tsc --noEmit`: clean.
- agobserver `pytest -k recorded_session`: **1 passed**.

A job cannot reach claim with an unverified session through the normal
paths: the run is held, and resume and recover refuse. Claim's refusal was
therefore tested on `sessionFor` itself; a held run's job is dropped earlier,
for the hold.

## Commits

- agdevworld `75cdae7`: `finished --interrupted/--attached`, `unverifiedSession`, refusals, docs/sessions.md *A reported id that differs*, tests.
- agautolab `043c082`: mismatch stop, attachment retry, tests.
- pj-agdev `a9e71e2`: agobserver request text and test, plus both pointers.
