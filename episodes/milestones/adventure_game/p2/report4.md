# Step 4 report — one scene produced from the human's examples

Plan: [plan.md](plan.md) step 4. 2026-09-22 15:20–15:49 UTC (2026-09-23 00:20–00:49 JST).
**Technical verification done; the human's evaluation is pending** — the plan's Done
condition ("the human can compare one playable scene with their originals and say where
the interpretation succeeds or fails") is theirs to close.

## The human's selection and examples

The Developer chose the scene themselves and published it as `protoprey-refs@a3c8196`
("first advice"): the forest biome's **wolf** — a discovery event and the five-step
predation event (catch → mouth → carry → swallow → stomach). What each example
establishes was stated in their own `human_advice.md`: the aesthetics (immersive,
bodily, second-person; the predator as a proud living being; fear, humiliation,
endurance, acceptance, awe — without dwelling on great pain; anatomical realism with
fiction woven in), the phase structure of a predation event, the base loop, the MVP
scope (forest only, wolf only, image+text units in an ordered event file), and a
request to confirm that the images' embedded generation parameters are readable. Six
texts and five images (1344×768, SwarmUI `oneObsession_v22`) are theirs. Nothing was
invented for them.

## The workflow, as it ran

| Time (UTC) | Who | What |
|---|---|---|
| 15:20 | Omni Agent → Front | `#front › front-protoprey-p2-wolf-event-20260923` #8077: relay of the Developer's material, the reference identity, the scene, the yardstick, "plan only" |
| 15:21 | Front → autolab | `#pj-protoprey › workplan-wolf-forest-scene` #8080, carrying `protoprey-refs@a3c8196`, the paths, the yardstick, and two ambiguities Front found by reading the reference itself (`hdiscovery.txt`/`rest.jph` typos, `todo.md` truncated) |
| 15:24 | autolab | plan #8085 (m8084): read the reference whole through `agrefs`, wrote `direction/REFERENCES.md`, confirmed EXIF readable, chose JSON event files and a parallel flow beside v0.1.0, resolved the typos, judged `todo.md` a superseded draft, and asked **five questions** before starting |
| 15:31 | Omni Agent → Front | #8103: answers (below); start |
| 15:32–15:38 | Front/autolab | task 1: assets and event files (`a6f9d14`, direction `676e680`), GOAL note |
| 15:38–15:44 | Front/autolab | task 2: `event_player.gd`, `forest_flow.gd`, menu entry F, `tests/forest_play.gd` (`ab6edef`) |
| 15:44–15:49 | Front/autolab | task 3: dead-link and exact-order assertions, README/VERIFY (`69b79b4`); delivery #8188 |

Twenty-nine minutes from the relay to the delivery; three tasks; no forge request (every
asset already existed in the reference, as autolab said).

**Interpretation and ambiguities surfaced before production** — the plan's requirement
— happened: Front caught the two file-name typos and the truncation by reading the
reference; autolab explained its interpretation (parallel flow, JSON, "explore/meet"
collapsed to direct triggering) and asked about (1) the content-policy conflict between
`GOAL.md`'s round-1 rule and `stomach.txt`, (2) menu placement, (3) meadow/wetland,
(4) whether EXIF should be read in-game, (5) discovery re-entry.

**Answers** (#8103). Items 2–5 were routine implementation choices, answered by the Omni
Agent as the requester's stand-in: new entry F beside D/E; meadow/wetland visible but
disabled with a note; no in-game EXIF reader; discovery once per run, then the wolf in
the predator list triggers predation. Item 1 is about the intended experience: the Omni
Agent read the Developer's own later material (`human_advice.md`, the texts) as
superseding the round-1 "no digestion shown / no sustained fear" rule *for this scene*,
within the limits the advice itself sets, and asked that the reversal be recorded
explicitly and marked as a reading to be confirmed by the Developer. **It was recorded
as "the Developer's own reading … relayed via Omni Agent"** in `GOAL.md` and
`direction/REFERENCES.md` — Front's relay dropped the "to be confirmed" qualifier. The
Developer should confirm or overturn it (see *Handed to the human*).

## What was delivered (`autodev/protoprey` `69b79b4`, tag stays `v0.1.0`)

- **Reused byte-for-byte**: the five images and six texts, copied into
  `main/assets/preds/wolf/` and `main/data/preds/wolf/`.
- **Transformed**: the advice's phase structure into `discovery.json` and
  `predation.json` (`id`, `kind`, `predator`, ordered `steps[{key,image,text}]`); the base
  loop into `forest_flow.gd` (biome choice → explore → discovery once → predator list →
  predation → biome choice), and a generic `event_player.gd` after `loop.gd`'s shape.
- **New**: menu entry F, the disabled biomes, `tests/forest_play.gd`, README/VERIFY
  sections, the dated content-policy note in `GOAL.md` and its `direction/` twin.
- **Untouched**: `loop.gd`, D and E (v0.1.0's identity is preserved; two tests bump a
  button count by one).

`direction/REFERENCES.md` records the adoption: source, full commit, date, why, what was
read, the EXIF check, the typos and how they were resolved, the mission.

## The Omni Agent's own check

Fresh clone into the scratchpad at `69b79b4`:

| Check | Result |
|---|---|
| `godot --headless --path . --import` | 0 error lines |
| `tests/forest_play.gd`, `playthrough`, `ui_play`, `media_play`, `loop_play` | all pass (`failed=false` / every path reaches the menu) |
| Wolf assets vs. the `agrefs` snapshot of `a3c8196` | all 5 images and 6 texts `cmp`-identical |
| `predation.json` order | catch, mouth, carry, swallow, stomach; `mouth` and `carry` share `carry.jpg` as the advice says |
| `godot --path . --quit-after 90` (windowed, real renderer) | exits 0 |
| Provenance | `GOAL.md` note, `direction/REFERENCES.md`, README "Known limits" pointer, each task text and the delivery name `protoprey-refs@a3c8196:<path>` |

Not checked by anyone: how the scene reads at play — whether the text over the image
carries the advice's intended experience. That is the human's part, and the plan keeps
it apart from the technical verification above.

## What the mechanism proved here

- A human-authored reference travelled request → Front → autolab plan → three tasks →
  delivery with its identity intact, and both Front and autolab **opened the originals**
  (Front found the typos itself; autolab decoded the EXIF).
- The originals were consumed unchanged; every derivative lives in the project's
  repositories; the departure from the earlier rule is written down, not silent.
- forge and archsage were not needed for this scene (no new asset, no research) — their
  routes exist (step 3) but are untested on a real request in this phase.

## Cost of the step

| Role | Runs | Cost |
|---|---:|---:|
| Front `front` (relay, planning, 3 task supervisions, delivery) | 15 | $3.22 |
| autolab `superdirector` (plan + re-plan) | 2 | $1.25 |
| autolab `supercoder` (3 tasks) | 7 | $3.74 |
| **Total** | 24 | **$8.21** |

Human effort: the Developer wrote six texts and generated five images with their own
SwarmUI, wrote `human_advice.md`, and pushed once — the bulk of the phase's human time,
and exactly what the braindump expected ("人月コストは他のプロセスよりはるかに大きくなる").

## Handed to the human

```
cd ~/projects/protoprey && git pull        # or a fresh clone of autodev/protoprey
godot --headless --path . --import
godot --path .
```
Menu `6` (F.): forest → explore → discovery → explore → predator list → wolf → the five
steps → biome choice. `Esc` back.

Two things only you can say:
1. Where the interpretation succeeds or fails against your originals (tone, framing,
   pacing, interaction) — in your words, here or in `#front`.
2. Whether the content-policy reading in `GOAL.md` (your advice supersedes the round-1
   "no digestion / no sustained fear" rule for this scene) is yours. If not, step 5 is
   where it gets revised.

## Addendum (2026-09-23) — the Developer's answer to item 2

The Developer answered at once: **the round-1 rule itself is inappropriate**, not only
for this scene — 「生々しい残酷描写はしないが、恐怖や消化という目の前の現実ははっきりと明記する必要があります」
("no graphic, cruel depiction, but fear and digestion — the reality in front of the
player — must be stated clearly"). Relayed to Front (#8191) as the Developer's own
words; Front opened a doc-only task 4 in the same mission; autolab replaced the
scene-scoped exception with the project-wide rule (Japanese original and translation)
in `GOAL.md`, added a follow-up entry in `direction/REFERENCES.md` that supersedes the
"Omni Agent's reading" entry while keeping it as history, and reworded the README
pointer. Commits `main` `f9f8d40`, `direction` `3bbb341`; no code or asset touched.
One transient glitch: autolab's plan-tracker edit failed with "Nothing to change" on a
no-op PATCH and it retried on Front's nudge (four extra runs). Item 1 (the play
evaluation) is still the Developer's to give.
