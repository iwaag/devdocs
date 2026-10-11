# p6 step 8 — real runs and lightweight final review

Plan: [plan.md](plan.md) §Step 8. Reviewed on 2026-10-11 JST at the person's request after they completed the trials.

## Real runs

| Run | Outcome | Human interaction |
| --- | --- | --- |
| `study/run-001` | All nodes and final report completed | Plan and finish answers recorded through the browser |
| `study/run-002` | All nodes and final report completed | Plan answer through Zulip; finish answer through the browser |
| `study/run-003` | All nodes and final report completed | Both plan and finish answers through Zulip, in `request-bestpractice` |
| `create/run-001` | All eight authored/generated nodes completed; result explicitly accepted | Plan answer through the browser; artifact-location request and final acceptance through Zulip, in `request-modernppt` |

The three study runs accumulated slide-generation research, a follow-up search, and explanatory-slide design knowledge in `study-ppt`. The create run used that knowledge to produce a 19-slide Japanese presentation on explanatory-slide practices and LLM-agent slide-generation research/projects.

## Create evidence

- Source: `devdocs/runs/create/run-001/deck.md` (Marp Markdown).
- Presentation: `deck.pptx`, 3,638,732 bytes; SHA-256 `277ed32fddea64013b11fe48b11701bd12057163eaccfe61e5248b1ecdda4ac7`.
- Preview: `deck.pdf`, 922,444 bytes; SHA-256 `39212724da9833c78407c7b69d875e9c9f9c7afa9d6deb3f158fbe0d98f65ebe`.
- Per-slide previews: `slides.001.png` through `slides.019.png`.
- Exercised procedure: `wedo-ppt/procedures/marp-with-chrome-for-testing.md`.
- Approval: q3 answer 0, Zulip message **15960**, from iwaag: “オッケーです、完了してください。” The result decision records that source and the PPTX/PDF hashes; these match the current files.
- Run completion: **23:15:56 JST**; the last attempt ended normally and its checkpoint was published. Final completion messages were sent to the request topic (**15964**) and run topic (**15966**).

The initial request was received at approximately 22:50:45 JST, execution began at 22:50:48, and the planning question was posted at 22:56:39 (**15952**). The earlier apparent silence was the planning interval; receipt, execution, and eventual question delivery are present in the logs.

## What was independently checked

This was a lightweight review, not a repeat of the implementation suites or research verification:

- Read all four run records: every node completed; no open attempt, hold, stop request, or cancellation remains.
- Checked create's confirmation source and recalculated both deliverable hashes.
- Checked PPTX ZIP integrity and counted 19 slide XML files.
- Inspected preview slides 4, 10, and 17: no obvious clipping, overlap, or unreadable text in those samples. The executing agent's report says it inspected all 19; the final reviewer did not repeat that full inspection.
- Downloaded the commit-addressed PPTX, PDF, and final report from Gitea without credentials: HTTP 200, with the expected PPTX/PDF byte sizes.
- Checked publication: `pj-ppt` at `f5c0f99`, `study-ppt` at `e9c153b`, and `wedo-ppt` at `293227c`, each with zero commits ahead of its upstream. The only untracked project file was a Finder `.DS_Store`.
- Checked durable transport state: all **51** inbox records were `done`; all **43** notification deliveries were `sent`.

Published evidence: [create final report](http://agstudio.local:3000/developer/pj-ppt/src/commit/f5c0f99/devdocs/runs/create/run-001/report.md), [PPTX](http://agstudio.local:3000/developer/pj-ppt/raw/commit/f5c0f99/devdocs/runs/create/run-001/deck.pptx), [PDF](http://agstudio.local:3000/developer/pj-ppt/raw/commit/f5c0f99/devdocs/runs/create/run-001/deck.pdf).

## Limits against the original trial plan

- `study/run-003` demonstrates a real study conversation entirely through Zulip. Create demonstrates Zulip receipt, result discussion, acceptance, and completion, but its plan answer has `via: browser`. A create run conducted exclusively through Zulip from planning onward has not been demonstrated.
- No live run contains an additional-instruction event during running work. Boundary handling has the isolated test evidence in [report5](report5.md) and [report7](report7.md), not the planned live exercise.
- Study runs have permission-to-finish answers but no explicit result decisions. Their execution completion is valid; the notifier's “Not accepted yet” wording reflects that distinction.
- The PPTX contains flattened slide images. Edit `deck.md` and regenerate to change slide text; native PowerPoint text editing is not delivered. This limitation was disclosed in the plan question and accepted.
- Preview verification is Marp/Chromium rendering. The deck was not opened in PowerPoint. The independent review did not revalidate the research claims or reproduce generation.

No blocking issue was found in this review. The core study/create trial goal is delivered; the two live-validation gaps above remain explicit rather than being counted as completed checks.
