# study_import p1 — step 3: finding the way to make the icons

Date: 2026-09-20. Mission `m7314` in `#pj-mediagen › workplan-hud-icons-method`,
five tasks, autolab throughout (11:18Z–11:45Z, about 27 minutes wall
time, 38 generated images). The Omni Agent wrote the request, accepted each
task after looking at the sheets itself, and made one decision (below).
Result: `localize/` at `d410fed`, pushed; `hud_icons` row **verified
2026-09-20**.

## What was tried

| method | outcome | measured |
|---|---|---|
| (a) Pillow tile + **drawn** fruit | **winner** | corner alpha 0 at 512/128/64/32; margin 10.0 %, radius 22.5 % at 512; fruit 65 % of canvas on all five; every fruit nameable at 32 px (Omni Agent's eyes, `task4/review_drawn.png`); 0.06 s per icon, 0.29 s per set |
| (a) Pillow tile + **generated** fruit (`perfectdeliberate_XL`, flat background, keyed in Pillow) | works for apple and banana; strawberry, orange and grapes need retries and stay blobs at 32 px | 28 images for 5 usable cut-outs (5.6 per icon), 7 hand-edited prompt rounds; "orange" alone drew a woman's face; backgrounds never flat, so the key is "bright, low-saturation pixels connected to the border" |
| (b) whole icon generated (one prompt per fruit) | **rejected** | alpha 255 at every corner in all 10; margin 0 or 95–240 px; radius 3–36 %; hue spread 7° in round 1, uncontrolled in round 2; "HUD" in the prompt draws grids and phones |
| (c) SVG rasterised | not run | `cairosvg` has no libcairo on this Mac; no rasteriser built |
| (d) `flux1-kontext` replacement | not needed | the winner's replacement is a spec edit and generates nothing |

## What is re-runnable

`localize/hud_icons/`: `build.py` reads `spec.toml` (fruits, shared hue with
per-fruit override, size, radius, margin, fruit fraction, sheet sizes,
seed for the generated route) and writes `<fruit>.png`, `sheet.png` and
`manifest.json`; `fruits_drawn.py` is the fruit art, one function per
fruit; `judge.py` measures alpha, margin, radius and fruit box at the
four sizes and writes a contact sheet; `generated.py` + `spec_generated.toml`
keep the generated route runnable as the secondary alternative. Only
entries whose parameters or art changed are rebuilt. Needs `pillow`
(drawn), plus `numpy scipy` (generated, judge) and `requests` (generated).

Proved in task 4 (`.local/evidence/hud_icons/task4/proof/`): grapes → pear
is one spec line and leaves apple, banana, orange, strawberry byte-identical
(a forced full rebuild gives the same hashes); hue 210 → 150 is one line and
measures 150° on the tiles; a per-fruit hue changes only that icon; size 512
→ 256 is one line. Run from a foreign working directory with
`uv run --with pillow python <abs>/build.py --out <dir>` (the way a forge
run will call it), checked by the Omni Agent after the push: 0.48 s, same
files.

## Decisions and findings kept

- **Fruit at 65 % of the canvas (≈81 % of the tile), not 65 % of the
  tile** — the Omni Agent's call in task 4, for legibility at 32 px; the
  README says so. The step 1 assumption said "of the tile"; the accepted
  set is what was reviewed.
- One shared hue for all five (blue) is the reading of "single-colour
  tile" that autolab took and the requester did not contradict; a
  per-fruit hue is one spec line if wanted.
- Failures are recorded in `hud_icons/README.md` under what was tried,
  what relaxing the requirement would buy, and alternatives (Recraft and
  Bria named as paid cloud options, not called). Tips that generalise
  (`perfectdeliberate_XL` never gives a flat background; "HUD"/"icon"
  wording draws UI chrome; a single fruit noun can draw a person) are listed
  by `main/` subject for a later mediagen tip, not written into `main/`
  here.
- Hand-repeated steps autolab named as tool candidates: the
  edit-prompt → generate → sheet → look loop, the per-fruit raw picking,
  the sheet build. The drawn route has none left.

## Who did what

autolab did every experiment, script and commit; cagent was not needed
(nothing installed, the missing libcairo and matting model were recorded
rather than requested); forge was not involved yet. The Omni Agent
reviewed each sheet by eye and decided the fruit fraction — **judging a
render for agent autolab: hand-off candidate** (autolab measured; the
"nameable at 32 px" verdict came from the Omni Agent).

## Observations on the mission route

- autolab resolved every task topic itself right after posting its report,
  before the Developer's acceptance; the acceptance for task 2 then bounced
  off the resolved topic and had to be carried in task 3's post. The
  introduction promises to wait for acceptance. Recorded for agautolab.
- The `hud_icons` README's "Candidate methods" and "Background" sections
  from the planned state remain above the findings; fine for a reader, but
  the file is now long.
