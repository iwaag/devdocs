# Adventure game p4 — one request with no Omni Agent in the loop

Source: [braindump.md](braindump.md). Previous evidence: [p3 report](../p3/report.md),
[robust_workflow p3](../../../robust_workflow/p3/report.md) and
[p3 ex1](../../../robust_workflow/p3/ex1/report.md).

## Goal and working approach

p3 proved the reference → forge → autolab → playable path once, with the Omni Agent relaying,
accepting, nudging and repairing. `robust_workflow` p1–p3 (+ex1) then removed the start relay,
added `agentchat trace`, Observer's request monitor, recorded mission acceptance, receipts and
per-mission working copies. **p4 measures whether that holds on one real request with the Omni
Agent out of the loop.** The Developer is the requester end to end; the Omni Agent observes,
verifies after the fact, and counts every Deus Ex Machina (DEM) event. p4/exN repeats the same
measurement on the next request; a phase that surfaces defects hands them to the next
`robust_workflow` phase instead of fixing them mid-trial.

- The request is the one already written in `protoprey-refs@a7b4763` `todo.md`: **a scene, shown
  after a predation event ends, where the player chooses the biome to respawn in — the Earth seen
  from space, rotating.** Resources: `scenes/modes/biome_view/images/bg.png` (1344×768 Flux2 still
  with ComfyUI metadata: Earth lower-left, sun upper-right, starfield) and `concept.png` (1672×941
  concept mock: "Choose Your Spawn Biome", sixteen biome bubbles, no metadata). The new
  `human_advice.md` section says a concept image is guidance, not a spec to reproduce.
- Breaking phase, private environment: no backward compatibility, no approval rounds for
  routine choices. The only prohibition that matters: **the Omni Agent does not post, accept,
  nudge, un-✔, edit or deploy during the request, unless the Developer asks for it.** Every such
  ask is a DEM event and is recorded with its cause.
- Reads are free: `agentchat trace`, the ops board (`http://localhost:8090`, `/?view=project`),
  the relay's `/healthz` and `/cost`, listener logs, `listener.sqlite`, mirror. None of them buys
  a run.

## step1 — Baseline and the request text

- Check the environment read-only: `nctl status` from `pj-clusterintent/nctl`, the launchd
  listeners, the running revisions (expected: pyagag `0ef3c61`, agautolab `3a67d63`, agfront
  `99ad106`, agobserver `74c881f`), relay `/healthz` (mirror `live`, tracked requests, open
  incidents — 5 tracked / 0 open at planning time). Record the numbers; they are the "before".
- Confirm the inputs: `agrefs revision protoprey-refs` → `a7b4763` (`5a18e23` added the images and the advice; `a7b4763` renamed the mock to `concept.png`); `autodev/protoprey` `main` at
  `6330a3a`; the Developer's play clone `~/projects/protoprey` has a stray untracked `protoprey/`
  folder (an old nested copy) — harmless, delete or ignore.
- Write the DEM log format up front (`dem_log.md`): time, who asked, what the Omni Agent did,
  cause class. Classes from the ex1 candidate list: rescue follow-up, missed completion record,
  change contamination, duplicate work, silent stall, returned change, false claim, missing
  tool/permission, other.
- Draft `request.md`: the request in the Developer's words, ready for them to paste. Keep it one
  request. Suggested content: the scene and its trigger (after predation, before the next explore;
  today that is `phase == "pick"` in `scripts/forest_flow.gd`, and the E-loop in `scripts/loop.gd`
  has its own biome return — say whether both or only F), the reference paths, "the concept image
  is reference only", the playable biomes are those in `BIOMES` (meadow, forest; wetland listed
  but not playable — do not invent the other thirteen), keep the found list and discovery state
  across the respawn, tests stay headless-runnable. Say which of the two images to use and leave
  how to "rotate" to the implementer.

Done: baseline recorded, `request.md` and `dem_log.md` in this folder, nothing posted.

Hints: bg.png is a still with Earth baked in. Rotation options an implementer can pick: cut a
circular Earth disc from bg.png and rotate it as a `Sprite2D` over the same still (the disc
center is roughly x 180 / y 640 at 1344×768, radius ≈ 620, mostly off-screen — a slow rotation
of a partly visible disc reads as a turning planet); a slow parallax pan; or a shader. A
transparent Earth sprite from ComfyUI is not available (no background-removal model on the
GPU node); if forge is asked for one, that is a second agent in the loop and a fair part of the
trial, but not required. Window is 1280×720, still 1344×768: the existing `still_rect` fit
handles it. A pulled PNG needs `godot --headless --path . --import` before it shows.

## step2 — The Developer runs the request; the Omni Agent watches

- The Developer posts the request in **`#front`** (a `front-protoprey-p4-…` topic, or the Project
  Room's request door). Not directly as a `workplan-` topic in `#pj-protoprey`: that bypasses
  Front, and Observer's request monitor discovers `#front` only, so a stall there would be
  nobody's to find.
- From then on the Developer does the requester's part in the conversations where it is asked:
  accepts each task in the task's own topic (that starts the next task), answers Front's
  questions, and records the mission's close — Front should run `agentchat accept <mission>
  --evidence <post>` itself; if it does not, `accept.flag` in the `workplan-` topic is the same
  operation for a person. Give Front and Observer their own time: Observer's detection is about
  6–7 min for a missing start or an undelivered answer, and a recovery request then goes to Front,
  not to the Omni Agent. Ask the Omni Agent only when nothing has moved past that, or when a
  tool refuses.
- The Omni Agent logs, and does nothing else: timestamps of each hop (request → plan → task
  start → checkpoint → accepted → integrated → completed → done), `agentchat trace` output at
  each stall the Developer reports, Observer `incident-…` topics and health-record entries,
  `[change]` notes in the task topics (the mission works in
  `.local/missions/protoprey/m<id>/`, `main` moves only at close-out — judge from the notes, not
  from Front's summary), and every DEM event with its class.
- If the Developer sends feedback after the first delivery, it is a second cycle of the same
  request and stays inside the measurement.

Done: the mission is `done` on record (or stopped on record), and every hop, stall and DEM
event has a timestamped line.

Hints: `agentchat read` hides `[selfnote]`s; read them with `ZulipClient.topic_history` in a
venv or `mirror.sqlite` read-only. `agentchat read --since` prints even when empty. A Front
line saying it resolved or accepted something is not evidence — check the ✔ state and the
`[state]` notes. Two Omni sessions on one account are invisible to everybody — make sure only
this one is running (`git log` before touching anything). Known Front quirk: a stale "not
integrated" carried from a previous message (ex1 follow-up 2) — log it as a false claim, do not
correct it.

## step3 — Verify the delivery independently

- After the mission is `done`: fresh clone of `autodev/protoprey` at the accepted commit,
  `--import`, the headless suites (`tests/*.gd`), and a windowed pass of the new scene from a
  predation end into meadow and forest, then the found list and a location revisit (discovery
  state must survive the respawn). Check the E-loop still runs if it was in scope.
- The Developer plays it and judges the scene in their own words; the Omni Agent records that
  separately from the functional result.
- Findings go three ways, and only one of them to the Omni Agent's hands: functional defects and
  creative feedback → the Developer posts them (a new cycle or a new request, measured);
  workflow defects → the DEM log and step 4; nothing is patched by the Omni Agent in the game.

Done: functional result, creative verdict and workflow observations recorded apart.

## step4 — Measure, classify, decide the loop

- Runs and cost per role from `.local/agent/<role>/run-NNNN.json` under agfront / agautolab /
  agforge (or the relay's `/cost`), wall time per hop, stalls and their detection time, Observer
  incidents, receipts written, DEM count by class. Put p3's numbers beside them (79 runs,
  $16.56, 1 h 48 min with a 24-min stall, nine acceptances and five relays by the Omni Agent).
- Verdict rule, stated before the numbers are in: **stable** = zero DEM events on the normal
  path, every stall that happened found by Observer and recovered in-system, mission `done`
  recorded without a human follow-up. Anything else names the cause class that broke it.
- Decide the next episode from the verdict: stable → `p4/ex1` with the next request (the
  Developer's next `todo.md` entry, or p3's scale test: five locations in one forge request);
  not stable → `robust_workflow` p4, whose braindump is this phase's DEM log and the reproduced
  failure as a fixture first (the p2 strict-xfail pattern). Alternate until the exit condition:
  three consecutive requests at zero DEM.

Done: one table, one verdict, one named next episode.

## step5 — Report

- `report.md` with the numbers, the verdict, the accepted revisions (`protoprey-refs@a7b4763`,
  `autodev/protoprey` commit, direction entry), work conversations by message id, and the
  Developer's assessment. Per-step reports as they happen.
- Update `README_DEV.md` / `.local/devenv.md` only where the phase found something a later
  reader needs. Guides: no rule added on one observation — an observed defect goes to
  `robust_workflow`, which fixes the system first.
- Commit and push devdocs; nothing in the game repositories is the Omni Agent's to push.

Done: the phase's claim — the workflow ran one real request without a Deus Ex Machina, or
exactly where it needed one — is evidenced and the loop's next step is named.
