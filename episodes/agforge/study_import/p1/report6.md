# study_import p1 — step 6: into operation

Date: 2026-09-20.

## Re-verification, short

- `localize/README.md` § "Re-verification and where problems go" (autolab,
  `78c428d`): rebuild into a scratch directory with `--force`, run
  `judge.py`, hash-check against `out/`, then update the INDEX row's state
  and date. Written per capability; `hud_icons` is the first.
- `agforge/README_DEV.md` § "Re-verifying a capability": `agforge knowledge
  list` first (state and last-verified date against the change), then the
  capability's own proof, then route. forge's own code: `uv run pytest -q`.
- No monitoring was built; the trigger is "a model, workflow, dependency or
  environment changed, or a run failed on something called verified".

## Routes

Recorded in both READMEs and in the run guide (forge has no chat tool, so
its report *names the kind* of problem and the requester routes it):

| kind | goes to |
|---|---|
| environment: library, service, model missing or unreachable | cagent (`cagent-agstudio1`) |
| implementation: a localised script or its README is wrong | a `workplan-` in `#pj-mediagen` (autolab) |
| general finding missing from `main/` | the same channel, as a mediagen tip naming the subject |
| forge's code or guides | agforge repository, tests |

## Deployed and announced

- agforge `089609b` (pj-agdev `b6233c9`); listener on `c9a3c83`+guides,
  guides and the intro are re-read per request so the last two commits
  needed no restart. Introduction re-posted to `#agents` with a new "What I
  know, and what I only list" section.
- `devdocs/README_DEV.md` forge section: the knowledge paragraph.
- `pj-agdev/.local/devenv.md`: where the two sources are checked out, the
  transcript paths, and the environment gaps found (no background-removal
  model on ComfyUI, no libcairo/ImageMagick/ffmpeg on the Mac).
- Repositories committed and pushed: devdocs, pj-agdev, agforge,
  `autodev/mediagen-localize` (by autolab). `main/` and `publish/` untouched.

## Phase report

`report.md` beside this file.
