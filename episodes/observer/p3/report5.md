# observer p3 step 5 — sessions apart from execution; obsolete assumptions retired (2026-10-14)

pj-agdev `3e482e0`. The running Observer process is still the old one until the cutover in step 7.

## Sessions

The observation contract already keeps them apart (step 2): `process.log.session` is the harness session id read from the attempt's log, beside the attempt id and the process. Observer attaches monitoring obligations to the run and incidents to attempts:

| Fact | Rule | Test |
| --- | --- | --- |
| Run = the work | tracked by `uid` until a recorded terminal outcome or an incident disposition | step 3 tests |
| Attempt = one execution interval | an incident's episode; its first suspicion belongs to it | `test_a_fresh_session_and_two_intervals_sharing_one_session_are_two_episodes` |
| Session = retained conversation, optional | recorded as evidence, never an identity; two intervals with one session id are two episodes | same test |
| Process restart within an attempt | same episode, same first suspicion, no new obligation | `test_a_process_restart_within_an_attempt_neither_opens_an_incident_nor_resets_its_age` |
| Idle retained session | may wait legitimately; judged from the run's record (question, external wait with deadline), never from the session's existence | classification (step 3) |

**Attempt time budget versus session lifetime.** Today `wfexec` stops an attempt after `timeout_seconds` (4 h), and each attempt is one process with a fresh session. When agent_session p1 reuses a session across attempts:

- the budget stays per execution interval (an attempt), never per session;
- waits keep their own deadlines (`wfe run wait --deadline`, step 2);
- the attempt should record the session it resumed (`attempt.begin` settings), so the observation reads it from the record rather than from the log;
- the foreground-wait rule in the `wfrun` guide stands: keeping a session does not keep a background job alive.

Delegation (parent/child run, result waits) will add references; no conversation ancestry is inferred meanwhile.

## Retired from the active Observer path

Removed with their tests (11 test files): the conversation monitor `monitor.py` (2227 lines: `#front` discovery, `agag.trace` over the mirror, Front asks, conversation rescue rules, tracked/held/retired files), its `triage` role and guide, `health.py` (legacy serving probes via `agag.health`), `review`/`review_status` (Zulip developer reviews of monitor incidents), `hold`/`disposition`/`withdraw` (records on `#front` requests; holds and cancellations are WFE records now), `trial` (fixture triage). No compatibility adapter remains.

Kept:

- The watch entrance (`listener`, `intake`, `worker`, `observe`, `notify`, `anchor`, `destination`, `store`) and `argue`, as a library; not started. `is_incident_topic` moved into the watch intake.
- pyagag's shared libraries (`agag.trace`, `agag.health`, the mirror): other agents use them.
- `wftriage` (step 3) is the reusable bounded local-model judgment.

**Old local state.** `python -m agobserver.retire_legacy [--dry-run]` inventories the old incident records, tracked/held/review files, watches, triage replays, `monitor-health.json` and `health.toml`, and moves them to `.local/archive/observer-p3-<time>/` with `retirement.json` ("replayed: false"; an archived record says what the old monitor last recorded, not that its work finished). Dry run on this host: 7 incidents (none open), 16 watches (none open), 9 other files. It runs once at the cutover.

**The process.** `com.agdev.agobserver` (`devenv/launchd/com.agdev.agobserver.plist.in`, `agobserver/service/workflow.sh`) runs `agobserver.wfmonitor`; it replaces `com.agdev.agobserver-zulip`. nodeutils/nctl read an agent's liveness from `agag.status.v1` at `AGAG_STATUS_PATH`; the monitor now writes it after each look whose observation came back (tested), so Observer stays observable without a Zulip listener.

## Evidence

- agobserver suite: 101 passed (workflow monitor 22, recovery path 7, watch library and argue 72).
- `python -m agobserver.wfmonitor --once` (the real entry point, scratch `--state`, source `wfe`, live registry, recovery enabled, judgments off): source live, 1 run observed (`respected-hold`, jevlab `experiment/run-001`), no incident, no post. It needs only `wfe`, its store and the Zulip client; no Front, relay or listener.

## Remaining for deployment (step 7)

The agentroom relay still evaluates the old `monitor-health.json` (and its ops board shows it); the watchdog is relocated to a process of its own in step 7, and the relay's evaluation is removed then.
