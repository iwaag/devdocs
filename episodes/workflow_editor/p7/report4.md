# Workflow editor p7 — step 4: presentation server lifetime

Commit: agdevworld `25aa19f` (shared with step 3).

## Changes

- **Host store** `server/present.ts` → `<area>/.local/present/present.sqlite` (`user_version` 1).
  - One row per instance: id `pi-…`, workspace, run, presentation, revision, host, pid, start signature (`ps -o lstart=`), port, bind address, state (`starting|serving|stopping|stopped|failed`), detail and log.
  - The owner token (0600) and serve config live in `.local/present/instances/<id>/`.
  - Configuration is `<area>/present.json` (`bind` default `127.0.0.1`, `ports` default 8300–8399, `publicBase`, `answerBase`), validated. A loopback `publicBase` is refused, so loopback is never advertised as remote.
- **Start** (`Presenter.start`).
  - The server process is spawned `detached` (its own session and process group) with stdio to its log, then `unref`'d. It runs from the real path of the release that started it, so a later release switch does not change a running instance.
  - Only one start per presentation:
    - an instance observed serving is returned (and recorded available again if the record disagrees);
    - a start in progress (< 30 s) is reported as `starting`;
    - an instance observed gone is marked and replaced;
    - an instance in an unclear state (alive without identity, or identity without the recorded process) is refused with the stop command.
  - The insert is a transaction, so two concurrent starts cannot both create instances.
  - **Available** is recorded only after `/.wfe-present/identity` returned this instance and `/` and the entry returned 200. Start-up failures (non-zero exit, hash mismatch exit 3, port exit 4, no answer within 10 s) record `failed` with the log tail. They kill only their own child and are retryable.
- **Stop** (`stopInstance`, `stop(<p>|all)`).
  - The shutdown request carries the owner token.
  - Then `SIGTERM`, then `SIGKILL`, but only if the pid's start signature equals the recorded one and its command line names the instance.
  - A stop is **verified** when the process is gone and the port no longer answers with the instance's identity. Only then is `stopped` recorded. An instance found gone records `stopped` (found gone).
  - An unverifiable stop records `stop-failed`, stays open (the run cannot complete) and is retried by the same command. Repeating a stop is harmless.
- **Status and recovery.**
  - `view`/`wfe present status` shows what the record says beside the observation (alive with the recorded signature, and answering with its identity). The link is given only when both hold.
  - `reconcile` marks gone instances in the host store and lists open presentations that are not served. Records change only through `start` (a new instance of the same revision) or `stop` (records `stopped`).
  - Nothing restarts by itself.
- **Cancellation.** `ExecStore.cancel` records `run.cancel` first, then stops every open presentation of the run (`Presenter.stop(…, 'all')`), independent of the graph's successors. The result is returned (`cleanup`) and shown by the CLI and API; failures stay `stop-failed` beside the cancellation. The reducer refuses `available` after cancellation and `wfe present start` refuses a cancelled run.
- **Pause versus cancel.** `run.stop` (held, resumable) does not touch presentations: a paused review keeps them. The policy is recorded in docs/presentations.md.
- **Interfaces.** `wfe present start|stop|status|reconcile`; HTTP `GET runs/<wf>/<run>/presentations`, `POST …/presentations/<p>/start|stop` (`by` required); run view *Start again* / *Stop* / *Retry stop*.

## Verification (`test/present.test.ts`, real processes)

| Required behavior | Test |
| --- | --- |
| An agent exit during a human wait does not stop the server | The "agent" is `/bin/sh` in its own process group. It runs `wfe present start` through the CLI and then waits. The test sends `SIGKILL` to the whole group, as wfexec does. The agent is gone, the server pid is alive, the URL still returns 200, and the record says `available` |
| Repeated start returns the existing instance | the second start is `reused` with the same id; one row |
| Restart recovery distinguishes the claim from the observation | after `SIGKILL` of the server the record still says `available`, the view says `live: false`, `gone`. `reconcile` notes it, and the next `start` makes a new instance that serves |
| Startup failure | a tampered bundle file: the start is refused (`differs from the manifest`), `failed` is recorded, and no instance is left `starting`/`serving` |
| Shutdown failure, unrelated processes unaffected | the store is pointed at an unrelated `sleep` (pid reuse) and the real instance's identity is made unverifiable (wrong token and signature). Stop records `stop-failed`, neither process is signalled, and completion is refused (`presentation-open`). After the identity is restored, the retry stops it, verified, and the run completes |
| Repeated stop safe | a second stop of a stopped presentation does nothing |
| Cancellation path | `ExecStore.cancel`: cleanup ok, process gone, presentation `stopped`, run `cancelled`; restart refused, and `available` refused (`run-cancelled`) |

The full suite is 184/184. The stop test takes ~27 s because it waits out the unverifiable stop's grace periods.

## Limitations

- The start signature has one-second resolution (`ps lstart`). Combined with the command line naming the instance id, it makes accidental matches implausible, but it is not cryptographic proof of identity. The shutdown token is the primary path.
- After a host reboot the store rows read "found gone" only when someone runs `status`, `reconcile`, `start` or `stop`. No daemon watches instances. wfexec's minute reconcile was not extended to presentations; the run view and status show the truth when asked.
- Instances are per host (`host` column). Another host's rows are reported, never stopped.

## Next

Step 5: recorded adjustments of a run's future work.
