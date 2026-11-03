# improve_wfrun p4 step 6: deployment and the ordinary user path

Date: 2026-11-03. Plan: [plan.md](plan.md). Host records: `pj-agdev/.local/improve-wfrun-p4/` (`notes.md`, `deploy/`, `shots/`),
`pj-clusterintent/nctl/.local/improve-wfrun-p4/`.

## Deployed and read back

| Item | Revision | Read back by |
| --- | --- | --- |
| WFE, coordinator | agdevworld `1053a73` | `GET api/version` |
| WFE, gpupc | agdevworld `1053a73` | Git on the worker |
| Executor, studio | agautolab `1f20225` (the code of `96b5a32` plus a test fix) | the checkout launchd runs |
| Executor, gpupc | agautolab `96b5a32` | Git on the worker; `wfe-worker` active |
| warden | Gitea `8619d68` | `wfe projects show warden --refresh` |
| Observer | pj-agdev `706e1d1`, restarted | its log |

- Exec store version 5 is applied. `96404c2` was activated with no attempt open, so no running work had to be handled.
- Settings changed outside Git: `workers.json` (coordination, lane names), `agautolab/.local/wfexec.toml` (`lanes =
  ["coordination"]`), `agobserver/.local/workflow.toml` (`exempt`). The earlier versions are in `deploy/`.
- The 21 stale `evidence-fixture` jobs were deferred by the agent's first turn, until 2026-11-04 05:44 UTC.

## The ordinary path

Room `p4-check-room` (category check, actor `p4-check`):

1. `p4-check` asked the conductor at 06:57:40 to have `p4trial` freework make 4 Godot images, naming no node.
2. Warden admitted the conductor beside two busy attempts (decision 62). The conductor delegated.
3. The delegated request went through the ordinary Zulip intake; warden moved the child to gpupc (decision 67).
4. The result reached the room at 07:01:46 (#616). The 4 PNGs and the script are on the published branch of `p4trial`.

I confirmed the files are on the branch and that two of them are 256 × 256 PNGs. I did not look at the images.

- **Overlap:** the trials above, on the deployed release.
- **Responsive warden:** turns during busy attempts, median about 20 s.
- **Agent-selected alternative worker:** decisions 59 and 67.
- **Deferral and reconsideration:** decisions 57 → 59 and 71 → 73.

### Browser

| View | Seen |
| --- | --- |
| Home, Execution nodes | every node, queued jobs by admission ("21 deferred"), load |
| Run view of a deferred job | `Deferred · By warden, until …` |
| Run details | node provenance, moves with actor and reason, the latest decisions |
| Room | a message sent through the UI, answered by the conductor |

**Attribution problem:** I sent #617 from the room UI and it was recorded as `iwaag`, because the UI's default sender is that
name. #618 in the room corrects it. It was not the Developer's message.

The room also triggered the separate conductor issue from `opinion.md`: after #617 the conductor replied and ended with its
ready node unrecorded. Recovery continued it in 1 min 37 s. This phase did not change that issue.

## To look at it yourself

- `http://agstudio.local:8093/wfe/`: Home, Execution nodes.
- `http://agstudio.local:8093/hangout/?conv=p4-check-room`: the room, left open.
- `http://agstudio.local:8093/wfe/#/ws/p4trial/run/bench/run-002`: Detail → Run details: a deferral, then a move.
- `bin/wfe coordinate view` and `bin/wfe coordinate decisions` in the area.

There is no dedicated coordination page in the browser. The queue-wide view and the decision list are CLI only.

## Effects outside the trials

- **Owner DMs were sent:** one for the coordination process I killed (15:09 JST), one executor-down (15:32 JST).
- **The executor's heartbeat lagged during my builds:** down, then available, at 14:37, 14:42, 15:16, 15:21, 15:30 and 15:35 JST, each
  during a release build or full test run. This is the known effect of heavy local work. The keeper thread did not prevent it.
- `coordinate/run-012`, killed on purpose, is cancelled, and its incident closed as `cancelled`. No incident is open now.
- A stray `report.md` in warden's workspace (`devdocs/runs/recover/run-082/`, Nov 2, untracked, different from the tracked file)
  blocked a rebase. I moved it to `pj-warden/.local/p4-aside/run-082/` and left it there.
