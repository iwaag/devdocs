# project_coop p1 — Step 1 report: the current cooperation map

Date: 2026-10-26. Machine-specific observations are in the ignored
`pj-agdev/.local/project-coop-p1/notes.md`.

## Revisions read

Published `main`, through `wfe projects list` (observed 2026-10-26 10:59 UTC):

| Project | Published | Developer workspace |
| --- | --- | --- |
| protoprey | `de32d81fb3` | same |
| gimmickimg | `35b73c0960` | was 8 commits behind, fast-forwarded (clean) |
| playprey | `c1112b64eb` | was 12 commits behind, fast-forwarded (clean) |

These match the plan's starting evidence, so the inputs did not move between proposal and execution.
`~/projects/protoprey` is the retired `autodev/protoprey` checkout, not the managed project.

## What each project has today

**protoprey** (gl_compatibility, Godot 4.7.2). Flow F is the forest/meadow loop: biome select → explore →
discovery → locations → predator list → the wolf's five-step predation event (`catch`, `mouth`, `carry`,
`swallow`, `stomach`), each step a still and a text. Locations are data (`data/locations/<id>/location.json`,
`index.json`); events are data (`data/preds/wolf/*.json`). 38 MB of media are tracked and listed in
`assets/ASSETS.md`. The project goals already require deliveries "by name and hash" that run without the
producer's checkout, but nothing implements that: there is no cooperation entry, acquisition tool, adoption
record or import path, and the guide names forge and playprey without saying how to reach them.

**gimmickimg** (no project guide; `freework` only). Results are experiments under `experiments/`:

- `godot/`: depth-band snow over a landscape (gl_compatibility).
- `experiments/waterfall-loop/` (freework/run-013, approved in review): the far-view waterfall of
  `assets/sample.png` animated with local MiniMax H3 (first=last-frame conditioning, the best loop of the arms),
  played as a feathered-alpha video sprite over a mushroom-free plate, plus the run-008 Wan mushroom sway as
  RGBA frames. Its Godot project is 14 MB; arms are chosen by environment variables.
- `experiments/window-loop/` (run-010): the same technique behind a window.

There is no consumer-facing description, delivery format, packaging or verification tool. The experiment
projects use fixed coordinates and environment switches meant for captures. The ComfyUI endpoint has a host
name default in tracked code (`experiments/window-loop/tools/comfy.py`, overridable by `COMFY`). The
repository's pack is 1.2 GB because the generated frames and clips of earlier experiments are tracked.

**playprey** (Forward Plus, Jolt physics, Godot 4.7.2). Workflows `onboard`, `compare`, `boast`, `freework`.
`knowledge/delivery.md` and `tools/delivery.py build|verify` define a portable delivery (format
`playprey_delivery` v1, hashes, producer commit and tree state); seven are published on the share (eagle_beakfit
and anaconda). Profiles include `wolf`, whose runtime assets are on the share
(`shared://agshare/playprey/baselines/c5d7c836/…`), but no wolf delivery exists. Embedding is explicitly
unresolved: the runtime is a whole project (`Harness` autoload, 164 global shader parameters, Jolt settings,
absolute `res://` paths, a boot scene that takes over the scene tree, a watchdog that quits).

`knowledge/delivery.md` contradicts itself and the current WFE contract: one paragraph says the runtime carries
dotfiles, the next says `game/` carries none because a share strips them. WFE `docs/shares.md` now carries user
dotfiles in inputs and outputs (workflow_editor p11).

**WFE** already provides what transport needs: `wfe projects` reads published state without checkouts;
delegation with questions, answers, revision and receipt assessment; `wfe run input` / `wfe run publish` with
`ag.wfe-input-set.v1` manifests and `.wfe-output.json`; `wfe share resolve|verify` per host; scoped context
blocks; human review rounds.

## Storage and runtime facts

- Share `agshare` is reached at a different path per host (the coordinator's backing folder on agstudio, an NFS
  mount on agpc); each host's `shares.json` maps it and `wfe share resolve` turns a `shared://` reference into a
  local path. A copy outside a WFE host has no such mapping.
- Godot 4.7.2 is installed on both execution hosts (studio macOS/Metal, gpupc Ubuntu/Vulkan with an NVIDIA GPU
  and a desktop session).
- No `cagent-resources` context block exists for protoprey, gimmickimg or playprey; the system block
  `cagent-cluster` is the only environment summary a run receives. gimmickimg's generation service is known to
  its runs through earlier run records and the `COMFY` default, not through context.

## Service checks

nctl (`status`, `relations`, `drift`, 11:03 UTC) reported agshare server and clients, the agpc WFE worker and
ComfyUI on agpc as converged (recorded state). Live checks at the same time: both workers `available` in
`wfe exec workers`; agpc reads the share and runs Godot 4.7.2. ComfyUI was not called, because the selected
case reuses existing generated material.

## Renderer probe (preliminary)

An existing playprey delivery (eagle_beakfit, onboard/run-005 o1) was copied off the share, verified and run
headed on agstudio:

- `--rendering-method forward_plus`: no errors.
- `--rendering-method gl_compatibility`: it runs and the mouth renders, but fog shaders (the haze) are refused
  ("shader type fog not supported in OpenGL renderer") and the lighting changes visibly.
- A clean protoprey clone switched to `forward_plus` passes all seven headless tests.

So switching protoprey to Forward Plus is cheap on the protoprey side, and keeping gl_compatibility costs the
producer's look. Step 3 decides with a windowed in-game trial.

## The bounded integration case

Both expressions belong to protoprey's forest:

1. **Landscape.** gimmickimg's `assets/sample.png` is byte-identical to protoprey's Mushroom Waterfall
   location image (`assets/locations/mashroom_waterfall/view.png`, md5 `06619d5c…`). The waterfall-loop result
   (first=last H3 water + mushroom sway) animates exactly that location's view when it is discovered or revisited.
2. **Predator.** The wolf predation event has a `mouth` step. playprey's `wolf` profile makes it an interactive
   first-person mouth scene inside the game; the player leaves it to continue the event.

Both reuse existing material. The trial still exercises cooperation: the waterfall needs packaging by its
producer, and the wolf needs a delivery that does not exist yet.

## Ownership map (initial)

| Information | Owner and place |
| --- | --- |
| How to request a gimmick scene, the delivery format, its verification | gimmickimg Git: a consumer entry and a delivery tool |
| How to request a mouth scene, delivery and embedding contracts | playprey Git: `knowledge/delivery.md` (reconciled), an embedding contract, `delivery.py` |
| Scene requirements, where each expression appears, authored placement/timing | protoprey Git (`data/…`) |
| Adopted delivery identity, producer revisions, hashes, where to get them | protoprey Git (a tracked adoption record) |
| Published delivery bytes | the share, as WFE run outputs |
| Share paths, endpoints | host `shares.json`, environment variables, context blocks |
| Fetched deliveries and imports | ignored folder in each protoprey copy |

## Gaps

| # | Gap | Kind | Owner |
| --- | --- | --- | --- |
| G1 | gimmickimg has no consumer entry, project guide, delivery format or verification | code + docs | gimmickimg |
| G2 | the waterfall composite is an experiment project (env switches, capture coordinates), not a reusable scene | code | gimmickimg |
| G3 | playprey's delivery doc contradicts itself and WFE on dotfiles | docs/code | playprey |
| G4 | playprey has no embedding contract: entry/exit, input and camera ownership, required project settings, path layout | code + docs | playprey |
| G5 | no wolf delivery exists | commission | playprey (trial) |
| G6 | protoprey has no cooperation entry, adoption record, acquisition/verification/import, or hosting for animated/interactive content | code + docs | protoprey |
| G7 | renderer mismatch (gl_compatibility vs Forward Plus) | decision + trial | protoprey |
| G8 | outside WFE, a copy cannot resolve `shared://`; it needs an explicit package or a remapped source | code | protoprey (consumer side) |
| G9 | gimmickimg's ComfyUI endpoint default is a host name in tracked code | env | gimmickimg |
| G10 | no resource context for the three projects | env | cagent (existing mechanism); decide in Step 4 whether the case needs it |

No WFE or executor change is indicated so far. Step 4 tests that from a fresh run.
