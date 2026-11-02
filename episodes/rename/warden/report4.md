# Step 4 report: the coordinated rename

Applied 2026-11-02, window 17:22:15 to 17:24:28 JST (about 2 minutes). Addresses and raw logs are in
the ignored local notes.

## Order and results

| # | Operation | Result |
| --- | --- | --- |
| 0 | Release, before the window | WFE `d7a3bd1` built and activated (release check passed); service and `bin/wfe` agree |
| 1 | Pause | watchdog, Observer, executor, slack-front, backup job, service booted out; 0 open attempts; service not answering |
| 2 | Migration input | 7 stores (`integrity_check` ok), registry, settings, 105 manifests and records, context folder, refs |
| 3 | Gitea | repository renamed `pj-warden`, id 138 unchanged, 84 branches unchanged; definition `51bfb98` published on `main` |
| 4 | Migration | workspace moved to `pj-warden`; 105 manifests; 104 tree remotes; 106 run records; stores; `zulip.json`; context folder; 6 presentation configs; 67 session folders |
| 5 | Tree-only record | `correction-check/run-001` committed and published as `ab3a3a1` |
| 6 | Zulip | folder and 113 channels renamed by id, 0 failed, 0 missing (done before step 4, read back after) |
| 7 | Observer | settings switched; 7 incidents rebound; recipient updated on 10 unclosed assignments |
| 8 | Incoming reference | curator `boast/projects/data.js` entry renamed, `f475161`; its icon file keeps its name |
| 9 | Read-only checks before resuming | 106 runs readable, all current project `warden`; 0 problems; 19 delegation links resolved; scan queued 0 (outbox stays 12898); 0 old targets; 114 provisioned |
| 10 | Resume | service, executor, Observer, watchdog, slack-front, backup job running |

Store rows changed are in the journal, `.local/migrations/rename-agops-warden.json`.

## Differences from the plan

- Zulip was renamed before the data transition, not after it. The rename script reads ids from the
  store, and the order avoids any scan seeing new keys with old channel names. Nothing ran in between.
- The GPU worker holds no copies of this project, so no worker file changed. Its pinned release is
  covered in report 5.

## Effects outside the window

My `npm run check` runs and rehearsal copies delayed the executor's heartbeat several times between
16:30 and 17:20. The watchdog logged the executor as down each time, and Observer opened incident
`i501d348c82` at 16:33, which sent the owners a DM (closed 16:34 as resumed). That was load from this
work, not a fault. Restarting Observer also repeated an already known `degraded` notice; see report 5.

GitHub was not pushed by this step. The Gitea redirect from the old repository name remains, which
Gitea creates by default. No code uses it.
