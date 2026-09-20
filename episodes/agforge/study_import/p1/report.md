# study_import p1 — phase report

Braindump: `braindump.md`. Theme: `test_theme.md`. Plan: `plan.md`. Steps:
`report1.md` … `report6.md`. Date: 2026-09-20, one session (about 2 h of
wall time from step 1 to step 6, of which autolab's two missions took ~31
min and forge's five requests ~15 min).

## Adopted way

**Knowledge → localisation → forge finds it → delivery → change → reuse**,
each part on an existing route:

1. mediagen `main/` stays general and publish-ready (`c415b0c`).
2. A new study-pattern folder `localize/` (`autodev/mediagen-localize`,
   `d410fed`, no `publish/`) holds what runs here: an INDEX of
   *capabilities* with a state, one folder each with scripts, a README that
   stands alone, failures and alternatives; host values in ignored
   `.local/`. Opened and filled by autolab through two ordinary
   `workplan-` missions (`m7275`, `m7314`).
3. forge reaches both through `agforge knowledge list|show|search|path`
   (`agforge` `a7297bc`…`089609b`): everything listed, nothing pre-selected,
   reachable under the roles' existing `Bash(agforge:*)` grant on any
   harness. The plan cites `<source>/<path>` and its verified state; the
   record carries `[selfnote][knowledge] mediagen@<rev>, localize@<rev>`;
   the run gets `knowledge.md` and its transcript is kept.
4. The `test_theme.md` request went through `assetplan` → `assetrun` five
   times: the five icons, one fruit replaced (four files byte-identical),
   the tile recoloured (fruit pixels byte-identical, contrast honestly
   reported), a new set in a new topic with three fruits forge drew itself,
   and a hard request answered with what can and cannot be guaranteed.

## Method for the icons

Pillow-drawn tile + Pillow-drawn fruit from one `spec.toml` (`build.py`,
`fruits_drawn.py`, `judge.py`). Measured: corner alpha 0 at 512/128/64/32,
margin 10 %, radius 22.5 %, fruit 65 % of the canvas (≈81 % of the tile —
the one decision the Omni Agent made, for legibility at 32 px), 0.06 s per
icon. Rejected with numbers: whole-icon SDXL generation (opaque corners,
uncontrolled margin and radius); generated fruit + keying kept as the
secondary route (28 images for 5 usable cut-outs, three fruits blobs at
32 px). Not run: SVG (no libcairo), Kontext (not needed).

## Limits and residue

- **No route back from a forge run into `localize/`**: fruits forge draws
  for a request (cherry, lemon, watermelon) stay in its workspace. The
  route is named (a `workplan-` in `#pj-mediagen`), not automated.
- Contrast on a recoloured tile is reported, not fixed; the requester
  chooses among the options forge lists.
- The generated route depends on a checkpoint that never gives a flat
  background and on hand-tuned prompts; no matting model is installed
  (recorded for cagent, not requested).
- Toolset documents still travel by copy and the `speech` toolset is an
  empty body; `list` shows the state of *knowledge*, not of toolsets.
- autolab resolves its task topics before the acceptance lands
  (agautolab, not this episode).
- Photoreal request planned, not run; the three forge topics are left open
  for the developer.

## Omni Agent stand-ins (hand-off candidates)

- Judged each rendered sheet by eye and decided the fruit fraction (for
  autolab, task 4): the measurement was autolab's, the "nameable at 32 px"
  verdict the Omni Agent's.
- Wrote the two mission texts and the five requests, and accepted
  deliveries — the requester's role, not an agent's.

## Evidence

Conversations: `#pj-mediagen › workplan-localize-workspace`,
`workplan-hud-icons-method`, their `work-m7275` / `work-m7314` channels;
`#agforge-agstudio1 › assetplan-hud-fruit-icons`, `assetplan-hud-icons-set2`,
`assetplan-hud-photoreal` and their `assetrun-` topics. Files (host-local):
`localize/.local/evidence/hud_icons/`, `agforge/.local/agentws/r7409`,
`r7457`, `agforge/.local/agent/*/run-NNNN.{json,jsonl}`.
