# Step 3 report: rehearsal of the data transition

Rehearsed on isolated copies: stores taken with `.backup`, workspaces cloned, a local bare repository
as the remote, no Gitea credentials, no transport process, and a separate session folder. No job ran
and nothing was sent. Machine-specific evidence is in the ignored local notes.

## Results

| Check | Result |
| --- | --- |
| Runs after migration | 106 of 106 readable, all current project `warden`; 0 observation problems |
| Replay | `wfe run check` usable for all 106 |
| History | for the 105 copy-backed runs, the earlier entries, definition and storage are identical, with exactly one new entry |
| UIDs | none changed; run situations unchanged |
| Notification scan | 0 rows queued (outbox stays 12898), 0 old targets or run rows, 114 targets still provisioned with their ids; no orphan delivery, event or IO row |
| Intake | a re-handed old post: `duplicates: 1`, no new request. A new Observer post became `recover/run-070` on `wfe/warden/recover/run-070` |
| Observer | 64 of 64 acknowledged assignments match their recovery run; no finding identity appears or disappears; all 69 recovery runs stay excluded from recovery |
| Delegation | 19 links: no problems; a parent resolves its child through the old stored workspace |
| Retired run | `recover/run-036` restored from retention, `usable`, branch kept, origin `pj-warden` |
| Reviews, presentations | rounds of `release-check/run-006` read; presentation views read from the moved folder |
| New run numbering | `recover/run-070`, `freework/run-002` (above existing) |
| Interruption | `kill -9` during records (47 of 106 done), then during stores (exec applied, notify not): the rerun converged, each run has exactly one rename entry, and a further rerun changes nothing |

WFE `npm run check`: type check, 717/717 tests, both builds. Observer `pytest`: 122/122.

## Defects the rehearsal found (all fixed before the live change)

1. A tree-only run, `correction-check/run-001`, had no copy. There are 106 runs, not the 105 in the
   plan. It was outside the migration and would have kept the old project. It is now included, and
   its changed record is committed and published with the workspace.
2. The workspace still held the old `project.yaml`, so a scan provisioned the old folder and 11
   channels. The migration now follows the published branch and refuses unless its id is the new one.
3. The observation of a child compared a hand-built UID and reported "another run now".
4. The review UID check and `roundsUsing` compared the current project. Both now accept the creation
   project.
5. A restored tree took its origin from the hashed contents list, which is the pre-rename URL. It now
   uses the manifest's URL.

Defect 1 also shows a limit of the inventory in report 1: it counted copies, not runs.

## Not covered here

- The Gitea and Zulip renames: their effects need the real services. The Zulip script's read-only
  check found all 114 ids and an administrator credential.
- Restoring an older backup generation. An isolated restore uses that generation's own registry; a
  live reinstate through `formerly` is only type-checked so far. Step 5 covers what it can.
- The delegation relay and a review's page start were read, not exercised.
