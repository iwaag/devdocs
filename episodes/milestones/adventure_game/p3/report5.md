# Step 5 report — forge's location integrated and demonstrated

Plan: [plan.md](plan.md) step 5. 2026-09-23 14:40–14:55 UTC. **Playable revision:
`autodev/protoprey` `6330a3a`** (content at `7ece9d7`, tests at `ea2be13` then `6330a3a`).
Forge's `fallen_log_hollow` plays through discovery and revisit beside the human's four
locations. **The Developer's creative evaluation is pending** and is recorded as such.

## The workflow, as it ran

| Time (UTC) | Who | What |
|---|---|---|
| 14:40 | Omni Agent → Front | [task_forest_location.md](task_forest_location.md) B2 with the actual key, URL and request id (#8620) |
| 14:41 | Front → autolab | `#pj-protoprey › workplan-add-fallen_log_hollow` #8623; plan m8627, one task; **Front started the task itself** (#8638) before being asked |
| 14:42–14:45 | autolab | fetched the zip, stored the files, wrote `location.json` from `NOTES.md`, registered the id, ran the tests — **both failed** — fixed the tests in a separate commit, pushed `7ece9d7` + `ea2be13` |
| 14:46 | Omni Agent | fresh-clone check; accepted; asked for one follow-up (#8657) |
| 14:47–14:53 | Front / autolab | `workplan-location-tests-from-index` m8671: tests derive their expectations from `data/locations/index.json`; doc corrected; `6330a3a` |
| 14:55 | Omni Agent | fresh-clone check of all six suites; accepted |

Five minutes from the relay to the pushed integration; seven more for the follow-up.

## What integration needed — the trial's measurement

| Question | Answer |
|---|---|
| Engine (script) change | **None.** `git diff e2e0022..ea2be13 -- scripts` is empty. |
| Asset repair | **None.** `view.png`, `discovery.txt`, `revisit.txt` stored byte-for-byte (checked against forge's zip on a fresh clone); `NOTES.md` kept beside the data unchanged. |
| Data added | one folder under `assets/`, one under `data/` (with `location.json` written from `NOTES.md`), one id appended to `index.json`, one row in `ASSETS.md`. |
| Test change | **Yes, at first.** Both suites hardcoded the forest as exactly two locations (7 and 21 FAIL lines after registration); autolab edited the expected lists in a separate commit and reported that `docs/ADDING_A_LOCATION.md`'s "covered automatically" claim was wrong. The follow-up m8671 made both suites read the index (per-biome lists, found-list counts, discovery steps per visit derived from each text with the game's own paragraph rule), verified by autolab that they still fail when the index and the behaviour disagree (order reversed: 15 and 44 FAILs; a location dropped from the index: `location_play` fails) and that a throwaway new location is picked up with no test edit. |
| Anything else | Godot's import step: a pulled PNG has no `.import` until `godot --headless --import` runs — the Omni Agent's first windowed pass showed "still missing (view.png)" for that reason alone. The README already says to import on a fresh clone; a pulled *new* asset needs it again. Noted for the guidance, not a defect of the format. |
| Brief error | the request said the new location comes on the forest's third visit; autolab found and stated the second (wolf + Flower Pond on visit 1, Mushroom Waterfall + Fallen Log Hollow on visit 2). The Omni Agent's brief was wrong, autolab was right. |

So: **adding a forge-made location required no gameplay code change**, and after m8671 it
requires no test change either. The first implementation supports repeated additions; what it
did not support on the first try was the *tests* keeping up, which is now fixed.

## Played, and seen

The Omni Agent's windowed key-driven pass at `ea2be13` (after import): visit 1 wolf → Flower
Pond → predation; visit 2 Mushroom Waterfall → **Fallen Log Hollow**, five paragraphs paged over
forge's still (the log, hollow, web, beetle and light shaft dimmed under the text as the human's
stills are), the found list `Wolf, Flower Pond, Mushroom Waterfall, Fallen Log Hollow`, `4` →
the revisit text, `Done` → biome choice. Screens saved in the scratchpad, not committed. The
new location is indistinguishable in presentation from the human's; the difference, if any, is
in the writing and the picture, which is the Developer's call.

## Human evaluation — pending

```
cd ~/projects/protoprey && git pull      # or a fresh clone of autodev/protoprey at 6330a3a
godot --headless --path . --import       # needed again after pulling the new PNG
godot --path .
```
Menu `6` → `2` forest → `1` (wolf) → `1` (Flower Pond, page `1`×5) → `1` (wolf predation, page
to the end) → `2` forest → `1` (Mushroom Waterfall, `1`×5) → `1` (Fallen Log Hollow, `1`×5) →
found list → `4` for its revisit. Compare with your own four: viewpoint and scale of the image,
the prose's body-first walk-in and its open question, the revisit's familiarity. Two known
departures forge named itself: a second beetle at the hollow's mouth that the text does not
mention, and 2,457 characters against your ~3,000.

No human edit, no additional generation attempt, and no Omni Agent edit of any delivered file
occurred in this step. Omni Agent interventions: the relay, two acceptances, one follow-up
request, and the fresh-clone verifications.

## Cost of the step

| Mission | Role | Runs | Cost |
|---|---|---:|---:|
| integration m8627 + follow-up m8671 | Front | 8 | $0.86 |
| | autolab superdirector | 3 | $0.39 |
| | autolab supercoder | 2 | $1.18 |
| **Total** | | 13 | **$2.43** |
