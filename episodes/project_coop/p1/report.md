# project_coop p1 — final report

Date: 2026-10-26. Step reports: `report1.md` … `report6.md`. Machine-specific details (addresses, paths, launchers)
are in the ignored `pj-agdev/.local/project-coop-p1/notes.md` and `pj-agdev/.local/devenv.md`.

## Result

ProtoPrey now plays two expressions from other projects inside the game:

- **Mushroom Waterfall** (gimmickimg): when the location is discovered or revisited, the waterfall flows and the
  mushrooms sway under the text, in place of the still.
- **The wolf's mouth** (playprey): at the `mouth` step of the wolf's predation, choice 2 hides the game and puts the
  player inside the wolf's mouth (click to look, WASD/Q/E to move, Esc releases the mouse, Esc again returns to the
  same step).

Both were obtained by protoprey's own run through the normal request entrance, without intervention, and work without
either producer's repository, run copy or services once the delivered bytes are installed.

**To look at it:** on agstudio, the three launchers in the review folder (game / waterfall / wolf's mouth); on agpc,
the three `*-p1` desktop icons. In any copy: `python3 tools/deliveries.py install`, import twice, then
`godot --path . -- --jump=location:mashroom_waterfall` or `-- --jump=predation:wolf:mouth`.

## What was built

| Repository | Change |
| --- | --- |
| playprey `63bf63c4` | delivery format v2 with an embedding contract; `delivery.py embed|unembed` (path rewrite, settings merge, restorable record); embeddable entry `scenes/embed.tscn` with `finished(reason)`; Harness embedded mode (no host argv, no watchdog, failures reported); the generated tissue GLB is delivered; docs reconciled with WFE's dotfile transport; README "For other projects" |
| gimmickimg `2e0a52f`, `826fdea`, `feec195` | relocatable gimmick player (`player/gimmick/`), recipe format, `tools/delivery.py build|verify` (format v1), README "For other projects", `docs/delivery.md`, project guide; ComfyUI endpoint only from `COMFY` |
| protoprey `9a8e8ff`, `91821dc`, `ce8e7b3` | `docs/cooperation.md`, adoption record `data/deliveries.json`, `tools/deliveries.py` (adopt/install/status/pack/remove; share, remapped root or explicit package; every hash checked), location `gimmick` and step `interactive` wiring with fallbacks, Forward Plus renderer, `--jump` review shortcut, tests `cooperation_play.gd` and `test_deliveries.py` |
| protoprey (by its runs) `5bad20f`, `8575bc0`, `f33a989` | adoption of gimmickimg r1 → r2 and playprey wolf r1, wiring and `project.godot` settings |
| devpolicy `69db240` | `styles.md` *Project cooperation* |
| devdocs `84a075c` | `README_DEV.md` *Project cooperation* |

No WFE or executor change was needed; delegation, run outputs, share resolution, scoped context and project reads
covered the case.

## Ownership as built

| Information | Where |
| --- | --- |
| Methods, lessons | producers' knowledge (`knowledge/`, `docs/delivery.md`) |
| What a producer offers, delivery and embedding contracts, tools | producer Git (README section, delivery doc, `delivery.py`) |
| Scene requirements, wiring, authored tuning (`dim`, labels, properties) | protoprey data |
| Adopted identity, producer revision, hashes, source, replacements | protoprey `data/deliveries.json` |
| Delivered bytes | `agshare` run outputs (gimmickimg run-014 o1, run-015 o1; playprey run-004 o1) — keep them |
| Fetched and installed bytes | ignored `.local/deliveries/`, `external/` in each copy |
| Share paths, generation endpoint, host tools | host configuration, executor environment, cagent's `cagent-resources` context, `COMFY` |

## Trial evidence (report5)

| Check | Result |
| --- | --- |
| Discovery and commissioning | protoprey freework/run-001 read the producers' entries and its own cooperation doc, delegated to gimmickimg (run-014) and playprey (run-004), adopted and wired both; 11:40 → 11:52 UTC, no question, no repair |
| Delivery integrity | the consumer checks every hash; `wfe share verify` of both outputs passes; a package carrying macOS `._*` files was refused |
| Actual integration | windowed checks in the Developer's workspace: animated location, interactive choice, mouth scene, return to the same step |
| Fresh copy | clean clone: install, import, all 8 Godot and 5 Python tests pass, tree unchanged |
| Producer independence | relocated copy with no `wfe`, no producer checkout, no generation service |
| Portability | agpc (Linux, Vulkan Forward+, NVIDIA): from an explicit package and from a remapped share root; tests and windowed run pass |
| Revision and replacement | protoprey freework/run-002 obtained gimmickimg revision 2 (run-015) and adopted it; revision 1 kept in `replaced`; tuning untouched |
| Existing behaviour | all earlier tests pass (one assertion widened for the animated scene) |

## Defects found by cooperating (fixed)

- playprey deliveries lacked the generated tissue GLB; wolf, cheetah and horse could not start from a delivery.
- playprey's Harness watchdog would have quit a host game after 600 s, and profile errors quit the host.
- gimmickimg's first player drew the video at its native size and squared the texture in its alpha shader (caught by
  comparing with the experiment's capture).
- protoprey would have gone to the menu on the embedded scene's first Esc.
- playprey's delivery doc contradicted itself and WFE about dotfiles.
- gimmickimg's tools carried a host name for the generation service.

## Limits and open points

- **Nobody has judged the look** of either moment; no run presented them for human review (the children's catalogs
  expect it, and the parents did not ask). The handoff above is the Developer's review.
- **agpc renders the mouth much blurrier** than macOS at the same point; not investigated (likely playprey's
  time-dependent wet-screen effect or a Vulkan difference).
- **Renderer:** protoprey needs a Forward Plus–capable GPU now; gl_compatibility reach (OpenGL-only, web) is given up.
- **Embedded identity:** the runtime's printed `config_hash` differs from the delivered one because paths are
  rewritten; `embed.json` keeps the delivered identity.
- **A copy that has the playprey settings but has not run `install`** logs three autoload errors and plays without
  the mouth scene.
- **Revision history:** gimmickimg's recipe moved to revision 2; revision 1 exists only as its published output.
- **Correction mechanism:** the replacement used a new delegation from a successor run; same-child
  `delegate revise` was not exercised, because the first run had completed.
- **cagent context** needed three refreshes to be project-specific and to keep the endpoint; its blocks rely partly on
  Omni Agent's reported agpc observation.
- One trial each; the standard is a recommendation grounded in this case, not a proven procedure.

## Recovery

- Drop an adoption: `python3 tools/deliveries.py remove <name>` (restores playprey settings) and commit; or revert the
  adoption commits. Go back to waterfall revision 1: `adopt mushroom_waterfall --source
  shared://agshare/runs/gimmickimg/freework/run-014/o1`.
- Renderer back to gl_compatibility: revert that line in `project.godot` (the wolf mouth then loses its fog and
  changes lighting).
- Producer changes are ordinary commits in their repositories; earlier deliveries keep working because they carry
  their own tools.
