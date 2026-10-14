# observer p3 step 2 — workflow observations through WFE (2026-10-14)

agdevworld `b40f430` (source; not yet released — deployment is step 7).

## What was added

- **`wfe observe [--json] [--workspace <id>] [--run [<ws>:]<wf>/<run>]… [--terminal]`** and **`GET /api/observe`** (`server/observe.ts`, contract `workflow_editor/docs/observation.md`, schema `ag.wfe-observation.v1`). One module serves both; Observer will not read SQLite or copy layouts.
- **Run identity**: `uid = <project>/<workflow>/<run>#<creation>`, creation = the first 16 hex digits of the SHA-256 of the canonical `run.create` entry, beside workspace, creation time, request receipt and run branch. The same display identity in two registrations is listed twice with the problem `ambiguous` naming the other creation; nothing is merged.
- **Discovery** uses the existing resolvers: every registry workspace, `runRefs` (project tree and p9 copies), `runSpace`/`readCopy` for where the live record is. A missing, damaged or still-creating copy is a source state (`copy-missing`, `copy-damaged`, `copy-creating`), the run stays listed with `unavailable.record`, and its queued job is still shown. Completed/cancelled runs with nothing outstanding are left out by default; `--run` lists a tracked run whatever its state; `--terminal` lists everything.
- **Consistency**: record, the executor's rows (one SQLite read transaction, `ExecStore.runRows`), record again; a sequence or open-attempt change re-reads (up to 3), and a record still changing is reported `consistent: false` with the problem `changing`.
- **Separate evidence**: executor heartbeat and slot (`ExecStore.queueView`), attempt liveness (`kill 0`), process-group members and CPU (one `ps` per look), harness activity (log last write, turns), and new **log facts** (`server/activity.ts` `logFacts`, from the log's head and last 1 MiB): the session id, the **open tool call** with its declared timeout (Bash `timeout`, else the harness default 120 s; background commands are not open calls), the harness's own end of the session, and background tasks the harness reported stopped. Workflow progress is the run agent's last report in its own name.
- **Waits**: `waiting {reason, responsible, since}` per run — `person:<name>` (open question, `stopped` hold), `executor` (answer recorded and not taken up, queued job, ready work without a job), `external:<ref>`, `agent:<name>` (executing), `recovery` (`interrupted`/`unknown`). Queue position and the work ahead (the slot's attempt, earlier jobs) by uid. **`wfe run wait … --deadline <time|20m>`** records an external wait's expected latest time (`node.wait.deadline`, additive), exposed per waiting node; it is a statement, not a timer.
- Publication problems exclude checkpoints superseded by a later published one; integration rows by state; `recoveries` lists `run.recover` entries (the operation arrives in step 4).

The observation reconciles nothing, queues nothing, creates no copy, writes no record and posts nothing.

## Evidence

- `test/observe.test.ts` (7 tests): an executing attempt in its own copy with a real process group, an open Bash call with `timeout: 600000` and the job queued behind it (position 1, ahead = that attempt by uid), the call's result closing it; an exit with a node running → `interrupted`, `human: false`, `responsible: recovery`, the session's end and one stopped background task — the motivating signature; a person's resume, a question, an answer not taken up (`executor`, nothing queued by observing), the resumption queued as the API queues it, an external wait with a deadline, a person's stop (`human: true`); a moved copy tree → `copy-missing` with the run and its job still listed and a 10-minute-old heartbeat → executor `unavailable`; two registrations with `run-001` → `ambiguous` on both, a cancelled one left out, listed with `--run` or `--terminal`; a record changed once between reads (2 reads, consistent) and on every read (3 reads, `changing`); the CLI (`node cli/wfe.ts observe --json`) and `GET /api/observe` agreeing on identity, sequence, situation, hold, waiting, attempts and source.
- Full suite: 294/294; type check and production build pass.
- The real log of the motivating attempt (`amv6sl63n-accd7c`) read by `logFacts`: session `a2d9830c…`, `ended: success/end_turn`, `backgroundStopped: 1`, no open call. The healthy later attempt `amv6tjofm-e9a6b2`: `backgroundStopped: 0`.
- Read-only look at the live registry from the source checkout: one run listed by default — jevlab `experiment/run-001`, location `copy`, consistent, `stopped`, `human: true`, `person:iwaag`; executor available (heartbeat 1.9 s). 0.33 s per look; 0.31 s with `--terminal` (13 runs, none with a failed publication).

## Notes

- Turns in a one-shot CLI process are counted from the log start up to 8 MiB per call (`activityOf`); the open call, session end and background facts come from the head and tail only, so they do not depend on the log's length.
- `run.create`'s receipt is null for the existing jevlab runs (they were created before receipts were carried into the entry); identity relies on the creation digest, not on the receipt.
