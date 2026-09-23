# Step 3 report — the location mechanism verified before new content

Plan: [plan.md](plan.md) step 3. 2026-09-23 14:16–14:22 UTC, by the Omni Agent; one small
autolab mission for the defect found. **Playable revision for the Developer: `autodev/protoprey`
`e2e0022`.** The Developer's creative assessment is pending (see the end); the functional
findings are below and are separate from it.

## Focused checks (fresh clone at `528605c`, then `e2e0022`)

| Check | How | Result |
|---|---|---|
| Import | `godot --headless --path . --import` | 0 error/missing/failed lines |
| First discovery over the still, one list entry per location, revisit ≠ discovery (all four), biome association, return to biome choice after a revisit, persistence across a respawn and across biome revisits, reset on a fresh F entry, forced-missing image / discovery / revisit | `tests/location_play.gd` (54 assertions) | pass |
| Wolf discovery once per run, predation in exact `catch/mouth/carry/swallow/stomach` order, replay on respawn, indicator | `tests/forest_play.gd` (untouched by the mission) | pass |
| Versions a/b/c, D, E | `playthrough`, `ui_play`, `media_play`, `loop_play` | pass |
| Real game pass | the Omni Agent's own key-driven script, **windowed** (GPU renderer), 22 screenshots (`scratchpad`, not committed) | menu → F → meadow: explore 1 with the "Nothing hunts you here yet" line, Little Burrow discovery, explore 2, Mound discovery pages 1 and 5, found list (Little Burrow, Mound), Mound revisit → forest: wolf discovery, Flower Pond discovery, found list (Wolf, Flower Pond), predation `catch` → respawn → forest again: Mushroom Waterfall discovery, found list of three, Waterfall revisit → meadow again: both still listed → `Esc` → menu → F again: `discovered_locations` empty and Little Burrow discovered afresh |
| Long text | Mound page 5 and Waterfall page 1 viewed | a paragraph fills about 60 % of the height at 26 px, wrapped at ~950 px, no overflow; revisit texts on one screen with a thin scrollbar track |
| Image framing | all four location stills viewed in the window | cover-crop of 1344×768 into 1280×720, subject intact; the meadow explore screens use the old painterly `meadow.png`, which clashes in style with the photoreal location stills (noted, not a defect of this mission) |
| Missing asset references | the forced-missing cases in `location_play` and a read of the code path (`event_player.gd` placeholder + `problems`, `main.gd` note line) | reported in the note line, no blank screen |
| Handoff contract vs. reality | `docs/ADDING_A_LOCATION.md` against the tree | matches; producer package = `<id>/{view.png,discovery.txt,revisit.txt,NOTES.md}`; integration = two folders + one index entry, no script |

## Functional findings

1. **Blank lines made blank screens** (found by probing, fixed). The paragraph splitter kept
   empty lines as steps: the Mound text rewritten with a blank line between paragraphs gave 9
   steps, 4 empty. A producer writing prose the usual way would have shipped blank pages. Fixed
   through the ordinary route — `#pj-protoprey › workplan-location-text-blank-lines`, mission
   m8476, one task, **`e2e0022`**, 1 min 20 s from Front's start post to the push — and
   re-verified on a fresh clone with CRLF and blank lines: 5 steps, 0 empty. The doc now says
   blank lines are fine.
2. Cosmetic, left as is: the event title shows the raw step key (`Mound - discovery_5`,
   `wolf - discovery`); the F title still reads "Forest and the wolf - where now?"; the
   "Wolf discovered — unlocked" label stays on over meadow screens; the meadow's explore still
   is the old painterly fallback.
3. Nothing obstructs the new-content trial: a new forest location is two folders and one
   index entry, and every screen it will use has been seen rendered.

## Preview of step 4's capability check (done here because it needed no agent)

SwarmUI lists `flux2_dev_fp8mixed` among its models, so forge's existing
`agforge image generate --model flux2_dev_fp8mixed` was tried once (1344×768, 8 steps):
**"All available backends failed to load the model"** — SwarmUI cannot drive the Flux2
architecture on this backend, while ComfyUI itself holds every node and model of the
Developer's workflow (step 1). So forge's current image tool cannot execute the Developer's
route; step 4 provides the adapter over the ComfyUI graph.

## The human's assessment — pending

The Developer has not played `e2e0022`. What only they can say: whether paging a
five-paragraph discovery reads well; whether the fixed discovery order (wolf, then Flower
Pond, then Mushroom Waterfall on the next visit; Little Burrow, then Mound) and the meadow
line are what they intend; whether the found list as "revisit" for both predators and
locations is acceptable for now. Their words go into step 6's report when given.

```
cd ~/projects/protoprey && git pull      # or a fresh clone of autodev/protoprey at e2e0022
godot --headless --path . --import
godot --path .
```
Menu `6`, then `1` meadow or `2` forest; "Keep going", Next, Done are `1`; pick a found item
by its number; `Esc` back; `6` again starts a fresh run.

## Cost of the step

| Role | Runs | Cost |
|---|---:|---:|
| Front `front` (fix request, start, report; close-out not yet counted) | 3 | $0.34 |
| autolab `superdirector` | 1 | $0.13 |
| autolab `supercoder` | 1 | $0.16 |
| **Total** | 5 | **$0.63** |

Plus one local SwarmUI attempt (failed to load, no GPU time) and the Omni Agent's windowed pass.
