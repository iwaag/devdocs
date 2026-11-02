# Step 5 report: verification of the deployed result

Checked 2026-11-02 on the deployed system. Check actor `rename-warden-check`. Screenshots and raw
output are in the ignored local notes.

## Results

| Check | Result |
| --- | --- |
| Display | dashboard and project page show `warden`, 0 occurrences of `agops`; old run `recover/run-069` opens under warden |
| Identity | one project, one workspace; repository id 138, 84 branches kept, history kept |
| Existing runs | 106 of 106 observed with old UIDs, 0 problems, situations unchanged |
| Discovery | `wfe delegate targets warden`, `wfe projects show`, published snapshots with the old project all read |
| New run | `check/run-002` on `wfe/warden/check/run-002`, from published `ab3a3a1`; number above existing; channel #866 |
| Recovery path, live | below |
| Duplicates | 0 old targets, channels or run rows; outbox rows of warden all sent; 0 new incidents on old UIDs; 0 open incidents |
| Backup | new generation `20261102T083249Z-8fc4`, 10 of 10 retired runs covered, verified at the destination |
| Retired run | restored and rehearsed in isolation from that generation, on this host and on the backup host: every check passed |
| Older generation | `backup reinstate --dry-run` from `20261102T060712Z-2623` resolves `agops:` to warden and keeps the newer record |
| Worker | `gpupc` updated to WFE `d7a3bd1` (needed: the older release cannot read `run.rename-project`, so backup-host recovery would fail); service active |
| WFE tests | 717/717 in the release check; a later test-only commit adds one, and that file passes 6/6 |

### Recovery path

`check/run-003` was interrupted by killing its attempt's process group during the `wait` node.

1. 17:29:47 killed. 17:30:00 incident `if3f64899e3`, condition `interrupted`, uid `warden/check/run-003#…`.
2. Observer posted `request-if3f64899e3` in `wf-warden.recover` (message 29037).
3. Intake bound it as `r29037`, creating `warden/recover/run-070`.
4. Assignment recipient `warden/recover`: acknowledged, performed at sequence 7 by `recovery-agent`.
5. The target's second attempt continued without repeating the sleep and completed. Only one
   continuation ran.
6. The incident closed `restored`.

Observer's `acknowledged` event carries `"late": true`, meaning it was recorded after the recovery run
had already acted. The outcome is correct, and I did not investigate this further.

## Remaining `agops` matches, by purpose

| Where | Why it stays |
| --- | --- |
| Run UIDs and `wfe/agops/*` branches of the 106 older runs | creation identity |
| `run.create` entries, frozen definitions, source snapshots, committed run documents | what happened |
| Sent content, Zulip messages and topics, Observer evidence and events, conversation text, tokens | what happened |
| Receipts such as `answer:agops/...` | deduplication keys of past work |
| Context and backup snapshots, archives, the seed in `agobserver/docs/agops/` | hashed or labelled history |
| `pj-agops-bff8c33a5c2f.git` retention repository | key recorded in the retired run's contents |
| Curator's icon file `img/agops.png` | a file name; only `icon` still refers to it |
| Registry `formerly` | resolves existing records |
| warden's README; trial references in `docs/runs.md`, `server/exec.ts`, `shared/run.ts`; tests | history and fixtures |

Current defaults, settings, stores' operational columns, launchd and paths have no match.

## Not verified

- **A pre-rename held run continuing.** None exists: all older runs are completed or cancelled. It is
  covered by a unit test only.
- **A recovery run's own failure, live.** Exclusion was confirmed on the real snapshot (all recovery
  runs, old and new) and by unit test, not by killing a recovery run.
- **Delegation relay and starting a review page on an old run.** Links and rounds were read; no new
  relay or page start was exercised.
- **`freework`.** The new-request check used `check`.
- **Live reinstate through `formerly`.** Dry run only.

## Open, not caused by this work

The watchdog reports `degraded`: 5 recovery requests not taken up, the oldest from 2026-10-19. These
are unclosed assignments on already closed incidents; the count grew from 2 to 5 before this episode.
They were left as they are, with only the recipient renamed.
