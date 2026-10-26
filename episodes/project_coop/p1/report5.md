# project_coop p1 — Step 5 report: the cooperation trial

Date: 2026-10-26. Requests were made through the normal request entrance (`wfe request freework --workspace
protoprey --braindump …`), authored "Omni Agent (for Developer)".

## Trial 1: the two moments (protoprey freework/run-001)

The braindump asked for a flowing Mushroom Waterfall and an experience inside the wolf's mouth, and said that other
projects may already have work for this. It named no project, tool or route.

Timeline (UTC): request 11:40:11 → first attempt 11:41:28 → nodes `gimmick`, `mouth` (delegate), `wire` added and both
delegations dispatched by 11:44:02 → gimmickimg result 11:46:22 → playprey result 11:47:34 → adoption and wiring in the
second attempt 11:48:48–11:51:44 → integrated into protoprey `main` 11:52:10. No question was asked; nothing was
answered or repaired by Omni Agent.

| What | Record |
| --- | --- |
| Discovery | The talk concluded "Delegating deliveries to gimmickimg and playprey". Its delegation requests quote gimmickimg `docs/delivery.md` (scene `mushroom_waterfall`, `tools/delivery.py build --scene …`), playprey's profile list and `knowledge/delivery.md` (tissue GLB), and protoprey's `docs/cooperation.md`; i.e. the Step 2/3 published revisions |
| d1 → gimmickimg freework/run-014 | built, verified and published `shared://agshare/runs/gimmickimg/freework/run-014/o1` (60 files, 6.1 MB); answered the consumer's question that `sample.png` is byte-identical to the game's still; limits stated (seam judged on stills, hand-polygon alpha); judged satisfied |
| d2 → playprey freework/run-004 | built from a clean tree at `63bf63c4`, verified and published `…/playprey/freework/run-004/o1` (1553 files, 91 MB), including the tissue GLB; ran it as its own project with a capture; reported controls, the end condition (`finished` on Esc; no timeout, the consumer owns one), properties, and a content check against protoprey's rules; judged satisfied |
| Adoption | `tools/deliveries.py adopt` in the run's copy: protoprey `5bad20f` (waterfall, location wiring) and `8575bc0` (wolf, mouth step wiring, `project.godot` settings), report `2a0fb35` |
| Run's own checks | import, headless start, every test headless; it stated that nobody had played either moment in a window |

The parent asked the producers itself, read their reports where they arrived (the receipt assessment showed no
omissions), and disposed both as satisfied. The children's catalogs expect a presentation for a person; neither child
presented anything, and neither did the parent. The Developer's review is therefore the handoff of Step 6.

## Trial 2: revision and replacement (protoprey freework/run-002)

Follow-up braindump with `--predecessor freework/run-001`: try the never-delivered second first=last waterfall loop
in place of the current one, keep the current one recorded, keep dimming and placement.

- New delegation d1 → gimmickimg freework/run-015, which committed recipe revision 2 (`8f4f03c`: only the waterfall
  clip changes, seed 2) and published `…/gimmickimg/freework/run-015/o1` with a numeric comparison of both loops.
- protoprey `f33a989`: `adopt` replaced the record (revision 2, manifest `a36cf416…`); revision 1 (run-014 o1,
  `4bee9857…`) is in `replaced`; `location.json` (`dim` 0.55) is untouched. The report tells the person how to switch
  back (adopt the run-014 source again) and where the raw clips are.
- Mechanism used: a new delegation from a successor run. The predecessor had completed, so the same-child
  `delegate revise` did not apply; the producer's own recipe history records the revision.

## Verification (Omni Agent)

| Check | Evidence |
| --- | --- |
| Discovery and commissioning | run-001's own nodes, requests and dispositions above |
| Delivery integrity | `deliveries.py` refused unless every hash matched (it refused a package carrying macOS `._*` files, below); `wfe share verify` of both `.wfe-output.json`: verified, every hash matches |
| Actual integration | Developer workspace after `install`: `tests/cooperation_play.gd` windowed with screenshots — the flowing waterfall dimmed under the discovery text, the extra choice "Look around inside the wolf's mouth", the mouth scene full-window, Esc back to the same step; repeated with revision 2 |
| Fresh-copy reproduction | clean clone of protoprey `main` (`f14427a`) on agstudio: `install` through `wfe share resolve`, import, headless start with no errors, all 8 Godot tests and 5 Python tests pass; `git status` clean afterwards (the embed reproduced the committed `project.godot` exactly) |
| Producer independence | the relocated copy below had no `wfe`, no share access by the tool, and no producer checkout; the game reads only `external/`; no generation service was used by the consumer at any point (shared production services were not stopped) |
| Environment portability | agpc (Ubuntu, Vulkan Forward+ on the NVIDIA GPU): source archive and `deliveries.py pack` package copied to local disk; PATH with only Godot and Python; `install --from <package>`; import with no errors; all tests pass; windowed run on the desktop with screenshots; no absolute agstudio paths in data, settings or installs |
| Revision and replacement | trial 2 above; revision 2 installed and checked in the workspace; previous identity retained |
| Existing behaviour | every earlier protoprey test passes in workspace, fresh clone and agpc copy (one assertion was widened in Step 3 to accept the animated scene in place of a still) |

## Observations and limits

- **Look not judged by a person.** No run presented the moments for review; my shots confirm mechanism and placement
  only.
- **Renderer/host differences.** On agpc the mouth view was heavily blurred compared with macOS at the same point
  (probably playprey's time-dependent wet-screen effect or a Vulkan difference); not investigated.
- **Transport artifacts.** macOS `tar` adds AppleDouble `._*` files; the consumer refused the package as unlisted
  content, and the cooperation test then passed through its fallback branch. A package made with
  `COPYFILE_DISABLE=1 tar --no-xattrs` installed cleanly. Strictness was kept on purpose.
- **Rebuilding revision 1** from gimmickimg `main` is no longer possible (the recipe moved to revision 2); revision 1
  survives as its published bytes on the share, which is why adopted outputs must be retained.
- **Producer warnings** carried into the game log: playprey's "band side_r" tooth-authoring line.
- **Interventions:** none in the trial path. Omni Agent wrote the two braindumps, and moved the package to agpc only for
  the portability check.
