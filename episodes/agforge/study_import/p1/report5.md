# study_import p1 — step 5: the ordinary request route, end to end

Date: 2026-09-20, 11:46Z–12:01Z. Everything below went through
`#agforge-agstudio1` as the Developer: `assetplan-…` → forge's plan →
a post in the `assetrun-…` topic forge opened → delivery. forge ran on
`a7297bc` for the first request and `c9a3c83` (transcripts kept) from the
replacement on. No Omni Agent hand did any of the work; the Omni Agent
posted the requests, downloaded the zips and looked at the sheets.

## Requests and results

| request | plan | run | checked |
|---|---|---|---|
| **five fruit icons** (`assetplan-hud-fruit-icons`, a7402) | found `localize/hud_icons` by itself, cited it as "verified here 2026-09-20", named `build.py`/`spec.toml`/`judge.py`, chose the drawn route, listed what it did *not* use and why; registration line: `knowledge: mediagen@c415b0c, localize@d410fed` | 33 s, 7 files, presigned zip + `[S3KEY]` | 5 × 512² RGBA; the five PNGs are **byte-identical to the localize build** (`b6c398…`, `f6e641…`, `748b08…`, `febb29…`, `3779fe…`); sheet has the 512/128/64/32 rows; corners transparent |
| **replace grapes → pear** (same topic) | re-plan cites the README's "only changed entries are rebuilt" and finds `pear()` already in `fruits_drawn.py`; plans to take the delivered four as the baseline and never regenerate them | 41 s, 8 files incl. `check_report.md` | apple, banana, orange, strawberry byte-identical to the first delivery (checked here too); `grapes.png` absent; pear alpha mask hash equal to the other four; readable at 32 px (sheet viewed) |
| **tile hue → green** (same topic) | one spec value; plans to rebuild the blue reference to prove the fruit pixels are unchanged; flags the pear-on-green contrast risk before running | ~2.5 min, 8 files | hue 120° at every sample, gradient 0.58→0.46 unchanged, masks identical, fruit pixels byte-identical between the blue and green builds; **contrast table per fruit** and an honest "pear is a pale blob at 32 px", with three options offered and none applied |
| **new set, new topic** (`assetplan-hud-icons-set2`, a7450): cherry, lemon, watermelon at 256 px, dark slate tile | says none of the three fruits exist, that a fruit is one function, copies the scripts into its own `hud2/` and never writes into the knowledge folder | 81 s, 4 files | 3 × 256² RGBA; the new fruits were **drawn by forge in code**, readable at 32 px; `localize/` git status clean afterwards |
| **hard request** (`assetplan-hud-photoreal`, a7478): photoreal studio-lit 3D fruit, reliable at 32 px | answers "yes, with one caveat about reliably": what it can guarantee by code (size, alpha, identical tile, shadow, scale, 32 px check) vs what it cannot (photoreal look, five matching), citing the localised evidence (28 images for 5 fruits, grapes/orange specks), the checkpoint licence, and fallbacks (paid Recraft/Bria, or the drawn set) | **not run** — a ~40-image, minutes-long attempt was not needed to prove the point of this row | the plan is the answer; the run topic stays open, unposted |

## What the knowledge access looked like from inside

The run transcripts (`.local/agent/{generator,assetrun}/run-NNNN.jsonl`,
kept since `c9a3c83`) show the calls: `agforge knowledge show
localize/hud_icons/README.md`, `agforge knowledge path
localize/hud_icons/build.py` (in `uv run --with pillow python "$(…)"`),
`… judge.py`. The plans read the README's "verified" state and the
recorded task 4 decision (fruit 65 % of the canvas) and repeated them
back. Precedence held: the chat's "keep the other four exactly as
delivered" overrode the spec's grapes entry; the shared `spec.toml` was
never edited.

Costs: planning generator ≈ $0.14 / 30 s each, runs ≈ $0.14–0.4 each
(`.local/agent/*/run-NNNN.json`); the drawn route itself is 0.3 s.

## Tests

`uv run pytest -q`: 262 passed. Added to `tests/test_knowledge.py`: the
registration line carries the stamp and the run workspace gets
`knowledge.md` from the same record (the plan flow and the run flow
wired through their real fixtures). Async is untouched and untested here:
image work is synchronous and no request used `pending.json`.

## Left over

- **A fruit forge draws in a run stays in the run.** Set 2's `cherry()`,
  `lemon()`, `watermelon()` live in `.local/agentws/r7457/generator/hud2/`
  and not in `localize/hud_icons/fruits_drawn.py`. There is no route from a
  forge run back into `localize/` yet; step 6 names where such a finding
  should go (a workplan in `#pj-mediagen`).
- The green request could not have re-used the delivered four *files*
  without rebuilding; it rebuilt them from the same spec instead and
  proved equality by hash. That is the editable-source reuse the plan
  asked for, not a file reuse.
- `assetplan-hud-photoreal` is planned but not run; the requester can
  post in `assetrun-hud-photoreal-a7478` to try it. The three topics are
  left unresolved for the developer to read.
- Omni Agent stand-ins this step: none in the work; **looking at each
  sheet and deciding the delivery was good** was the Omni Agent's, which
  is the operation-room acceptance a person would do — not a hand-off.
