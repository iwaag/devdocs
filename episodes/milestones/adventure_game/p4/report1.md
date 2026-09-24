# p4 step 1 — Baseline and the request text

2026-09-24, 12:44–12:55 UTC. Read-only; nothing posted, nothing restarted.

## Baseline ("before")

| What | Value |
|---|---|
| `nctl status` | ok: Nautobot 3.1.3 authenticated, 1 celery worker / 0 pending, dumps fresh (3.6 h), five submodules clean |
| launchd | all eleven jobs loaded and running (agfront, agautolab + gateway, agforge + request service, agobserver, archsage, cagent zulip + api, comfy-notifier, agentroom relay); `observation-refresh` is the periodic one-shot |
| pyagag | `0ef3c61` in agfront, agautolab, agobserver, agforge (installed `direct_url.json`) |
| agautolab | `3a67d63` (expected) |
| agfront | `99ad106` (expected) |
| agobserver | code `74c881f` (expected); the folder's newest commit is `b9877cd`, its introduction only |
| relay `/healthz` | mirror `live`, 0 resyncs; Observer monitor `ok`, cycle 212, **5 tracked / 0 open incidents**, 0 pending judgments |
| relay `/cost` today | 392 runs, $37.71 (all `claude_code`, reported) — the day's robust_workflow trials |
| newest message id | #11282 (autolab's mirror) — every p4 post is newer |

Run records per role at the start (the step 4 diff is against these numbers):

| agent | role → last run |
|---|---|
| agfront | front 1007, routine_run 64, present 18, argue 29, character_talk 30 |
| agautolab | superdirector 275, supercoder 432, entrance_front 29, front 23, director 12, coding 11, run 22, window 19, answer 2 |
| agforge | front 69, assetrun 41, generator 63, runcreate 7, entrance_front 4 |
| agobserver | intake 20, observe 53, triage 29 |

## Inputs

- `agrefs revision protoprey-refs` → `protoprey-refs@a7b4763` ("change name"), as planned.
- `bg.png` 1344×768 carries ComfyUI `prompt` and `workflow` chunks; `concept.png` 1672×941 has none.
- `autodev/protoprey` `main` at `6330a3a`, autolab's project folder clean and even with origin.
- The Developer's play clone is at `6330a3a` with the stray untracked `protoprey/` folder; left
  alone (it is the Developer's clone).
- Only this Omni session is active: devdocs, pj-agdev and its submodules have no commits or dirt
  since the plan commit `4e3a191`.

## What the request says (and why)

[request.md](request.md), for the Developer to paste. Choices made in drafting:

- **Scope F only**: the scene replaces menu F's biome choice; E keeps its own. The trigger that
  must hold is the end of predation (`phase = "pick"` after `predation` in
  `scripts/forest_flow.gd`); whether the first choice and the return after a location revisit
  use it too is left to autolab, with "say what you chose".
- **bg.png is the image**; concept.png is named reference-only, echoing `human_advice.md`.
- **Rotation is left to the implementer**, with the Earth-is-baked-in fact stated so the choice
  is informed. The disc-cut / pan / shader hints in the plan are not in the request — they are an
  implementer's call, and handing them over would be the Omni Agent designing the work.
- Biomes: those in `BIOMES`; "do not invent the other thirteen".
- State survives the respawn, resets on a new game; tests headless; the usual commit/push.

## DEM log

[dem_log.md](dem_log.md): format and the nine cause classes; empty.

## Next

Step 2 needs the Developer: paste `request.md` (edited as they like) into `#front`, then act as
the requester in the conversations. The Omni Agent only reads until asked.
