# p8/ex1 captures: Simple and Detail

**Source.** agdevworld `5d4519c`, built locally. These are not captures of the installed release, which is `8cda5c9`.

**How they were made.** `node checks/displayShots.ts --out <this folder>`. The script builds `checks/states.ts`'s private project, registry, area and service on a loopback port. It adds a second project with a long Japanese and English name. Each screen is captured right after it opens, in each mode, at **1440 × 1000** (`<screen>-<mode>.png`) and **1280 × 800** (`<screen>-<mode>-1280.png`). `index.json` lists every file with its screen, mode and size.

**Synthetic parts:**

- **Working run.** The working run's host facts (`/exec`) report an alive attempt. A fixture has no executor process.
- **Disconnected.** The private service was stopped for that capture.

Everything else comes from records written through the server's own operations. No production data was read.

[step2/](step2/) holds step 2's first captures of Project and completed Run.

## Screens and their p8 counterparts

| Screen | Simple | Detail | p8 (same state, 1440 × 1000) |
| --- | --- | --- | --- |
| Dashboard | [dashboard-simple](dashboard-simple.png) | [dashboard-detail](dashboard-detail.png) | none captured in p8 |
| Project | [project-simple](project-simple.png) | [project-detail](project-detail.png) | none captured in p8 |
| Project, long Japanese/English name | [project-jp-simple](project-jp-simple.png) | [project-jp-detail](project-jp-detail.png) | — |
| Definition, nothing selected | [def-simple](def-simple.png) | [def-detail](def-detail.png) | [def-improve](../../image_after/def-improve.png) |
| Definition, node selected | [def-selected-simple](def-selected-simple.png) | [def-selected-detail](def-selected-detail.png) | [def-node](../../image_after/def-node.png) |
| Definition, long names, selected | [def-longtext-simple](def-longtext-simple.png) | [def-longtext-detail](def-longtext-detail.png) | [def-longtext](../../image_after/def-longtext.png) |
| Definition, large graph | [def-large-simple](def-large-simple.png) | [def-large-detail](def-large-detail.png) | [def-large](../../image_after/def-large.png) |
| User nodes | [nodes-simple](nodes-simple.png), [selected](nodes-selected-simple.png) | [nodes-detail](nodes-detail.png), [selected](nodes-selected-detail.png) | none captured in p8 |
| Run: working | [run-working-simple](run-working-simple.png) | [run-working-detail](run-working-detail.png) | [run-working](../../image_after/run-working.png) |
| Run: open question | [run-question-simple](run-question-simple.png) | [run-question-detail](run-question-detail.png) | [run-plan](../../image_after/run-plan.png) |
| Run: two open questions | [run-questions-simple](run-questions-simple.png) | [run-questions-detail](run-questions-detail.png) | [run-branch](../../image_after/run-branch.png) |
| Run: question in Japanese | [run-question-jp-simple](run-question-jp-simple.png) | [run-question-jp-detail](run-question-jp-detail.png) | [run-longtext](../../image_after/run-longtext.png) |
| Run: live review | [run-review-simple](run-review-simple.png) | [run-review-detail](run-review-detail.png) | [run-review](../../image_after/run-review.png) |
| Run: answer awaiting take-up | [run-answered-simple](run-answered-simple.png) | [run-answered-detail](run-answered-detail.png) | [run-answered](../../image_after/run-answered.png) |
| Run: accepted, cleanup pending | [run-accepted-simple](run-accepted-simple.png) | [run-accepted-detail](run-accepted-detail.png) | [run-accepted](../../image_after/run-accepted.png) |
| Run: completed | [run-completed-simple](run-completed-simple.png), [selected](run-completed-selected-simple.png) | [run-completed-detail](run-completed-detail.png), [selected](run-completed-selected-detail.png) | [run-completed](../../image_after/run-completed.png) |
| Run: completed, no decision | [run-nodecision-simple](run-nodecision-simple.png) | [run-nodecision-detail](run-nodecision-detail.png) | [run-nodecision](../../image_after/run-nodecision.png) |
| Run: held | [run-held-simple](run-held-simple.png) | [run-held-detail](run-held-detail.png) | [run-held](../../image_after/run-held.png) |
| Run: failed | [run-failed-simple](run-failed-simple.png) | [run-failed-detail](run-failed-detail.png) | [run-failed](../../image_after/run-failed.png) |
| Run: cancelled | [run-cancelled-simple](run-cancelled-simple.png) | [run-cancelled-detail](run-cancelled-detail.png) | [run-cancelled](../../image_after/run-cancelled.png) |
| Run: historical | [run-history-simple](run-history-simple.png) | [run-history-detail](run-history-detail.png) | [run-history](../../image_after/run-history.png) |
| Run: disconnected | [run-disconnected-simple](run-disconnected-simple.png) | [run-disconnected-detail](run-disconnected-detail.png) | — |

Each screen also has a `-1280` version in both modes.
