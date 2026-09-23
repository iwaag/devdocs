# Adventure game p3 — phase report

Source: [braindump.md](braindump.md). Plan: [plan.md](plan.md). Steps:
[1](report1.md) inputs and tasks · [2](report2.md) locations from the human's examples ·
[3](report3.md) verification · [4](report4.md) forge's location · [5](report5.md) integration ·
this report. Executed 2026-09-23 13:00–15:00 UTC by the Omni Agent with Front, autolab and
forge. Task texts: [task_locations.md](task_locations.md),
[task_forest_location.md](task_forest_location.md).

## The answer to the braindump

The braindump asked whether forge and autolab can add content of the same kind as the human's
hand-made examples, and whether a location forge makes can be added by autolab. **Both
happened once, through the ordinary channels**: autolab built discoverable, revisitable
locations from the Developer's four examples at `protoprey-refs@6e017fb`; forge, given the same
references and the format autolab had defined, produced a fifth forest location
(`fallen_log_hollow`: one Flux2 render at the references' settings, five paragraphs of
discovery, two of revisit, a provenance note) in nine minutes on its first attempt; autolab
integrated it with **no engine change and no asset repair** — two folders and one index entry.
The Developer has not yet played the result, so whether the fifth location is *as good as*
their four is not answered here.

## Reference and production revisions

| Repository | Revision | What |
|---|---|---|
| `developer/protoprey-refs` (human) | `6e017fb` "new todo" | the four locations, biome art, `human_advice.md` with the Flux2 section, the ComfyUI workflow |
| `autodev/protoprey` | `528605c` → `e2e0022` → `7ece9d7`, `ea2be13` → **`6330a3a`** | locations (m8298); blank-line fix (m8476); forge's location (m8627); tests read the index (m8671) |
| `autodev/protoprey-direction` | `fafc924` | third adoption entry |
| `autodev/mediagen-localize` | `31c0ba1` | `flux2_scenery/` — the Developer's Flux2 graph as a runnable, verified capability (m8519) |
| agfront | `6c45d94` | guide: a post starts a task, say the id; never resolve a fresh topic |

Work conversations: `#front › front-protoprey-p3-locations-20260923` (#8286–#8509),
`front-protoprey-p3-flux2-capability-20260923` (#8512–#8585),
`front-protoprey-p3-forest-location-20260923` (#8572–#8700); `#pj-protoprey ›
workplan-locations` (m8298), `workplan-location-text-blank-lines` (m8476),
`workplan-add-fallen_log_hollow` (m8627), `workplan-location-tests-from-index` (m8671);
`#pj-mediagen › workplan-flux2-scenery-capability` (m8519); `#agforge-agstudio1 ›
assetplan-protoprey-forest-location` (a8589) and `assetrun-protoprey-forest-location-a8589`.

## Verification results

- Fresh-clone checks by the Omni Agent at every accepted revision: import clean; six headless
  suites green at `6330a3a`; images and texts byte-identical to the reference snapshot (human
  locations) and to forge's zip (the fifth); `git diff e2e0022..ea2be13 -- scripts` empty.
- Windowed passes (the Omni Agent's key-driven script): all 22 screen types of the F flow at
  `528605c`, and the fifth location's discovery, list and revisit at `ea2be13`. autolab's own
  windowed screenshots in task 3 of m8298.
- Defects found and fixed in-system: blank lines between paragraphs became blank pages
  (`e2e0022`); the tests hardcoded the location list, so the first addition needed a test edit
  and the doc's promise was false (`6330a3a`, verified to fail on order and registration
  mismatches and to pick up a throwaway location with no edit).
- Left as found: raw step keys in titles (`discovery_5`), the meadow's painterly explore still
  beside photoreal locations, the stale "Forest and the wolf" title, the wolf label over the
  meadow; a second beetle in forge's image; forge's discovery text at 2,457 characters.

## The human's assessment

**Pending.** The Developer has not played `528605c`, `e2e0022` or `6330a3a`. What they can say
and nobody else can: whether the paging of five paragraphs reads well; whether the fixed
discovery order and the meadow line are what they want; and, the phase's real question,
whether Fallen Log Hollow stands beside their four in viewpoint, scale, atmosphere and prose.
The hand-over is at the end of [report5.md](report5.md).

## Production effort

| | |
|---|---|
| Wall time, relay to accepted integration | 13:07 → 14:55 UTC (1 h 48 min), of which 24 min was one stall |
| Location feature (m8298, m8476) | 69 + 8 min, 5 + 1 tasks |
| Capability (m8519) | 9 min, 2 tasks |
| forge's location (a8589) | 9 min, 1 render kept of 1 attempt, 3 min 28 s GPU |
| Integration (m8627, m8671) | 5 + 7 min |
| Agent runs and cost | 79 runs, **$16.56** (Front 50 / $6.27, autolab 26 / $9.71, forge 3 / $0.57) |
| Human corrections | none (no human edit, no human generation, no human message during the phase) |
| Omni Agent interventions | relay of five requests; nine acceptances; two review corrections (registration as data; file names in the contract); one repair (un-resolving the mission topic and closing its twin); three nudges to Front to post a start; verification on fresh clones and in a window |
| Gameplay code change to add the fifth location | **none** |

## What the trial proves, and what it does not

**Proved once:** the reference → forge → autolab → playable path, with identity kept at every
hop (`protoprey-refs@6e017fb:<path>` in forge's plan and notes, `a8589` and the run topic in
`location.json`), the human's originals untouched, and the additions data-only. A missing
capability was supplied in-system in nine minutes and used by forge without a guide change.
forge read the actual images and prose, matched the settings from the PNG metadata, judged
its own output and named its flaws.

**Not proved:** quality in the Developer's eyes (pending); production at scale — one
location, one attempt, one biome; whether forge's prose holds up over many locations without
converging on one voice; a request that needs image-to-image or a rejected attempt; anything
about the meadow's missing predator or a second predator.

## The observed bottleneck, and the smallest next experiment

The bottleneck was not generation and not integration; it was **Front's handling of starts and
closes**: an accidental resolve seconds after opening the mission, a start post reported but
never made (24 minutes of nothing), a wait for a report on an unstarted task, and a go-ahead
round trip before every first post. Every one of these was invisible to Front and found by the
Omni Agent reading the task topic or the listener log. The guide now says what a start is and
what a resolve is; the smallest useful next experiment is **the same production cycle with no
Omni Agent in the loop** — the Developer posts the request in `#front` or the Project Room
themselves, accepts from what autolab and forge report, and the phase measures whether Front
gets four tasks started and closed without a nudge. If it does, the next scale experiment is
five locations in one forge request, to see whether the prose and the pictures stay distinct.

## Guidance updated

- agfront `front` guide (this phase): a task or run starts only with a post in its own topic;
  report the message id or post it; never resolve a fresh topic; a resolve is a rename.
- `docs/ADDING_A_LOCATION.md` in the game (by autolab): blank lines tolerated; registration is
  all the tests need.
- `localize/flux2_scenery/README.md` (by autolab): how to run the Developer's route here.
- Not changed: autolab's and forge's guides (nothing in the phase called for it), the
  `README_DEV.md` references section (p2's text still holds).
