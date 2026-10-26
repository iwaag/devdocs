# project_coop p1 — Step 6 report: deployment, handoff and the standard

Date: 2026-10-26.

## Published state

`wfe projects list` after the last integration: protoprey `ce8e7b3`, gimmickimg `90af5c3` (then `feec195`, docs),
playprey `6c825fb8`; developer workspaces fast-forwarded to them. Trial work (protoprey run-001/002, gimmickimg
run-014/015, playprey run-004) was integrated by WFE's own mechanism. No WFE, executor or other system component
changed, so nothing was redeployed.

## Prepared for review

- protoprey `ce8e7b3` adds a review shortcut: `godot --path . -- --jump=location:<id>` or
  `-- --jump=predation:<predator>:<step>` (README). Checked windowed: both targets open with no errors.
- **agstudio:** a separate review clone (so in-system runs keep their workspace) with the deliveries installed and
  imported, and three Finder launchers: the game, the waterfall, the wolf's mouth. Each was started and showed no
  errors. Locations are in the ignored `pj-agdev/.local/devenv.md` (*project_coop p1 on this host*).
- **agpc:** a review copy installed with `PROTOPREY_SHARE_ROOTS=agshare=<agpc mount>` and no `wfe` (the remapped-root
  portability mode; the package mode was checked in Step 5), import clean, cooperation test passing, and three desktop
  icons.
- No web service is involved; the result is a native game.

## Environment context

cagent refreshed its `cagent-resources` blocks twice more: context/run-009 made them project-specific and dropped
the stale "code default" sentence; that refresh also dropped the ComfyUI base URL, which context/run-010 restored
with its observation age. gimmickimg runs now get the endpoint from context and must set `COMFY`.

## The standard

- `devpolicy/styles.md` *Project cooperation* (`69db240`): producer entry and delivery contract, consumer cooperation
  entry and adoption record, how bytes are obtained and checked, environment facts outside tracked code, replacement.
  Stated as the style's recommendations, grounded in this case.
- `devdocs/README_DEV.md` *Project cooperation* (`84a075c`): where each piece lives, the trial, defects found, limits.
- Agent guidance: no change to the shared wfrun guide. The trial run found and used everything from the project
  guide and the producers' README sections, so a shared instruction has no demonstrated value yet.
- Project lessons stay with their owners: playprey `knowledge/delivery.md` (tissue GLB, embedding), gimmickimg
  `docs/delivery.md` *What the first deliveries taught* (`feec195`), protoprey `docs/cooperation.md`.
