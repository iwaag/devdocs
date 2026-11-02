# Step 7: deployment and direct verification

## Deployed

| What | Revision | Verified by |
| --- | --- | --- |
| Coordinator service and `wfe` | agdevworld `66bdad5` (previous `d7a3bd1`), 22:18:45 JST | `api/version`, `wfe version`; release build 735/735 |
| agpc worker | wfe `66bdad5`, executor `a9b82f8` unchanged | playbook ok=22 failed=0; `git rev-parse` on agpc |
| Studio executor | Unchanged | Claimed and ran the check runs |
| pj-agdev pin | `83035a2` | |

No store schema changed. Drafts add one folder. No running attempt was open at activation. Machine details and how to go back: `pj-agdev/.local/workflow-editor-p20/notes.md`.

## The original placeholder case

Actual inventory at execution: 21 registered projects, same as the baseline. Through the deployed service, the `placeholder` node shows in the published `freework` of **21 of 21**, with checkouts 1 to 83 commits behind and none synchronized (`check/placeholder.log`). Browser: ponderer, 83 behind (`shots-deployed/7-ponderer-placeholder.png`).

## Migration

Inventory of all 21 checkouts: **no workflow edit exists only in a checkout**, so nothing was imported and no draft was manufactured. Compared by hand for the 4 dirty checkouts: rts-vs-bot and ppt hold `.DS_Store` files, warden an untracked run folder, protoprey a modified `project.yaml`. That `project.yaml` edit was not touched and is outside this inventory, which covers workflow files. No existing draft format existed to migrate; run evidence was not changed.

## End to end on the deployed service, real executor

Test project `wfrun-trial`, whose registered checkout stayed at its first commit `9e6f4a8` throughout, now 70+ behind.

| Step | Result |
| --- | --- |
| Edit in the GUI, Save draft | Draft `d-00e004084683` r2 |
| Try draft | `stamp/run-010` completed; records that draft, r2 and its sha256; the published definition stayed different |
| Apply | `e6cdfe082c`; graph shows it, published read agrees |
| Ordinary run from the project page | `stamp/run-011` at `e6cdfe082c`, with the applied definition |
| Agent edits the workflow, accepted, integrated | `stamp/run-013`, integration 579 → `ec0454d726`. The open graph showed it no later than the journal's integrated time (-188 ms: the push comes before the journal's entry) |
| Earlier run after all of this | `stamp/run-010` still has its own definition |
| Retired run | `gimmickimg freework/run-014`: definition read from its record |
| CLI | `wfe draft start|save|apply` applied `613ac83bb2`; checkout unchanged |
| Another LAN PC (agpc) | Chrome there: graph and placeholder in rts-vs-bot, project list, no page errors. Viewing only; editing was not done from it |

The first script (`deployed-part1.log`) stopped because it accepted a run before the attempt had ended, a defect of the script. `stamp/run-012` was integrated anyway, without an open-graph measurement, so `deployed2.mjs` repeated it with `run-013`.

## Load after release

`wfe intake status` 3.8 s, one sample (3.2 s recorded in improve_wfrun p1). Polling figures are in report 6, from a scratch service. No recovery run was created during this work: the latest, `warden recover/run-104`, is from 20:40 JST, before it began. The whole check ran at low priority for that reason.

Seen and left alone, not from this work: the watchdog reports observer `degraded` (5 recovery requests not taken up, the oldest 14 days old), and 21 `warden evidence-fixture` start jobs have been queued since 2026-10-27.

## Developer check

1. Open <http://agstudio.local:8093/wfe/#/ws/ponderer/wf/freework.yaml>. The Placeholder node is there, labelled Applied.
2. If you like, open <http://agstudio.local:8093/wfe/#/ws/wfrun-trial/wf/stamp.yaml>, press Edit, give your name, change something, press Save and apply. The graph shows it, and new runs use it.

Other links are in the ignored notes.
