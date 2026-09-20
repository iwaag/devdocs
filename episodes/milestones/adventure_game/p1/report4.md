# Step 4 report — the first playable build, v0.1.0

Plan: [plan.md](plan.md) step 4. 2026-09-20 15:22–16:46 UTC. Research round 1 ended
at 15:22; the human's decision to build came at 15:27 (in the Omni Agent's chat, relayed
as #7712); the build was delivered at 16:46 — **84 minutes** from decision to a tagged
build, 15 of them spent on the basis fix below.

## The decision, and the gap it exposed

After round 1 the human said (translated): *a human shrinks, gets eaten and comes back —
there is no scientific basis to check in that one spot. Build the MVP.* No ecology check,
no round 2. The Omni Agent relayed it to Front in
`#front › front-protoprey-handover-20260920T1500Z` (#7712), marked as a relay.

Front could not act: **only the `argue` role carried `agproject`**, and the argue was
closed; `agentchat send` refuses a resolved topic; the `front` role reported it had no
tool to open a project (#7714). The argue route had been the only door because argue p1
built it there. With the human's go-ahead the basis was changed
(agfront `4538f9e`, pj-agdev `247533c`): `front` and `desk` carry `Write` and
`Bash(agproject:*)`, the front guide says when to use it (a decision the developer
actually stated, here or in a closed argue), the origin lines read "conversation" when
the home is not an argue; 181 tests; listener restarted 15:43Z. **Omni Agent basis
work, not a stand-in**: the decision stayed the human's, the setup stayed Front's.

## What the agents did from there

1. **Front** wrote `GOAL.md` from #7526, #7552, #7589, #7712 and round 1, opened
   `#pj-protoprey` (#7718, `[AUTO] … opened from conversation front/…`), asked autolab for
   setup (#7720); autolab answered in 47 s (`main/GOAL.md` at `1852539`, `direction/` and
   `devlog/` provisioned with the workspace, `README_PROJECT.md` pointing at channel,
   topic, conversation, argue and the study).
2. **Front** opened `workplan-first-build`; **autolab** planned mission **m7732** in
   Godot 4 (installed here; Twine, Ink and Ren'Py were not) with five tasks and
   `start.flag`, and Front started each task as the previous closed, accepting each
   delivery on the evidence autolab gave.
3. **Task 1** (2 min): a Godot skeleton, three text-only eaten-scene variants as JSON
   (implied / cut-away / direct-but-bloodless), a path test and a key-press UI test.
4. **Task 2** (14 min): assets. autolab asked forge in three `assetplan-` topics.
   - The eaten still (a7798) came back in 4 min.
   - **Sound**: forge declined — its only audio tool is music generation, so it could not
     promise a music-free ambient loop, a seamless seam or a 3–6 s cue; it wrote
     `idea.md` naming what would enable it (DSP libraries). autolab synthesised both
     files itself (`make_sounds.py`: a 60 s stereo loop and a 5 s hush).
   - **Biome stills** (a7836): forge's first run produced nothing — its generator role
     was **blocked on a shell approval** for the `zsh intermediate/gen.sh` it had written
     (only `agforge:*` is granted). autolab asked for a rerun with direct
     `agforge image generate` calls; the second run delivered forest floor and wetland
     shore in 2 min.
5. **Task 3** (7 min): scene D, the direct variant with still, ambient loop, hush at the
   catch, fade, mute and volume keys, a visible note when an asset fails to load.
6. **Task 4** (24 min, one stop): the three-cycle loop with a biome choice at each return
   and a carried "memory" line. autolab stopped because **the a7560 meadow still was
   never imported into the project** — it had been forge's delivery to the argue, judged
   by the human from the download link, and lived nowhere the project could reach. Front
   asked the Omni Agent whether to use a placeholder; the answer was no, ask forge.
   **forge could not re-deliver its own a7560 result** (the upload had expired and the
   run's zip is not something it looks for), so it made a new meadow still (a8007) in the
   same style. `ASSETS.md`, `README.md` and `VERIFY.md` say so.
7. **Task 5** (4 min): `README.md` (start command, Godot version, fresh-clone import,
   controls, known limits), version `0.1.0` in `project.godot` and the menu, annotated
   tag `v0.1.0`, `VERIFY.md` with what was checked and what was not.

## The Omni Agent's own check of the delivery

Fresh clone of `autodev/protoprey` at `v0.1.0` (`205037d`) into the scratchpad:

| Check | Result |
|---|---|
| `godot --headless --path . --import` | 0 error lines, 6 assets imported |
| `tests/playthrough.gd`, `ui_play.gd`, `media_play.gd`, `loop_play.gd` | all four end with `failed=false` / every path reaches the menu |
| `godot --path . --quit-after 90` (windowed, real renderer) | opens on Metal, exits 0 |
| Assets | 3 stills 1920×1080, `ambient_meadow_loop.wav` 60.0 s stereo, `eaten_hush.wav` 5.0 s |
| Texts | the direct variant's seven nodes read as written: awe, no wound, no process; the loop's three biome openings differ |
| Stills viewed | the eaten still is a vast dark shape descending over grass toward a low sun, no creature detail, no gore; the meadow fallback is a ground-level meadow |

Not checked by anyone: how the sounds sound, how the layout reads at play, whether the
three stills sit together. That is the human's part.

## Cost of the step

| Role | Runs | Cost |
|---|---:|---:|
| Front (setup, mission driving, forge questions) | 31 | $3.68 |
| autolab (planner + 5 tasks + reruns) | 23 | $4.21 |
| forge (3 plans, 4 runs, 1 refusal) | 14 | $1.61 |
| **Total** | 68 | **$9.50** |

Running total for p1 so far: step 1 $0.65, step 2 $2.87, step 3 (research) $8.41,
step 4 $9.50 — **$21.43**.

## Findings

- **A closed argue is a dead end for a later decision.** Fixed for the `front` and
  `desk` roles; the argue guide's own gap (closing while its question to the human is
  unanswered) remains.
- **Assets a human accepts in an argue are not thereby in any project.** forge's
  delivery is a download link that expires; nothing moves it into a workspace, and forge
  does not re-deliver from its own record. The same still had to be made twice.
  Candidate: forge keeps and re-serves its results by request id, or a project's asset
  request names the argue delivery to import.
- **forge's sound capability is music only**; it said so instead of delivering the
  wrong thing, and autolab's fallback was a script. Whether the synthesised sounds serve
  is the human's judgement; a DSP toolset for forge is a candidate if they do not.
- **forge's generator writes shell scripts it is not allowed to run.** The approval
  block cost one run and one rerun; the retry with the granted command worked. Either the
  guide says "call `agforge` directly" or the grant includes the interpreter.
- **Front and autolab both prefix their own mention on top of the listener's**
  (`@**Front** @**Front**`), so the doubling seen in step 1 is in the consumers'
  guides or in `agag.reply`, not in one agent.
- **Mission closure**: the five tasks are `✔`, but `workplan-first-build` and the
  mission stay open; Front reported the delivery to the requester's conversation, not
  into the mission topic, and nobody wrote acceptance. The human's evaluation in step 5
  is the natural acceptance; the Project Room shows the mission with 5/5 completed.
- autolab wrote every piece of text in the game. The study's technique catalogue is
  visible in the variants (implied / cut-away / direct), which is the study reaching
  production — through the plan and the task text, not through the sage.

## Reached

A build the human can start, with its start command, controls, known limits and
version, verified to start and hold its assets by autolab and again by the Omni Agent;
its research, assets and code are traceable: argue → study round 1 → GOAL → m7732 →
`v0.1.0`.

## Handed to the human

```
git clone http://<internal git>/autodev/protoprey.git && cd protoprey
godot --headless --path . --import
godot --path .
```
Menu: `1`–`3` the text variants, `D` the still-and-sound scene, `E` the three-cycle
loop (`M` mute, `-`/`+` volume, `Esc` back). Godot 4.7.2. Or play from autolab's own
clone under `agautolab/.local/projects/protoprey/main`.
