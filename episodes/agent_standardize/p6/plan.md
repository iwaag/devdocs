# agent_standardize p6 plan — asset requests as ordinary project work

Goal: retire autolab's special asset route (marker, label, `asset_gate`,
the `assetplan-` answer handler) and let a plan that plainly says "ask
agforge for X" run as ordinary work: the task's supercoder talks to forge
through forge's entrance, the way Front did in p2–p5. **Images and music
only** — video is out of scope this phase.

Decisions already made ([braindump.md](braindump.md) + discussion):

- **Method 1**: autolab learns about other agents from the harvested intro
  list, same as agfront. No hub (method 2).
- **Who/what is decided at planning time, the conversation happens at
  execution time.** The superdirector writes the delegation into the plan
  (which agent, what to request, what comes back); the supercoder of that
  task opens the `assetplan-` topic, answers forge, fires `assetrun-`,
  receives the URL and integrates the asset.
- Delegation is its own task. The plan guide should steer "obtain the
  asset" into a standalone task whose deliverable is the asset URL posted in
  its topic; the next task reads it from there. This bounds the waiting
  blast radius to one task (p5 lesson: a run that ends mid-wait leaves
  nobody watching).
- Breaking-change phase; test realm; no compatibility with the marker route.
- Video excluded (its pipeline is the slowest; not a proof target here).

Success criteria:

1. A mission in the test project that needs an image **and** a piece of
   music yields a plan whose delegation tasks name agforge and the request
   explicitly — written by the superdirector from `tools/agents.md`, not by
   code.
2. Each delegation task's workrun: supercoder opens `assetplan-…` in
   `agforge-agstudio1`, the forge flow completes through `assetrun-`, the
   asset lands in the project, the task is closed `✔` by the supervisor
   (Front, p5 shape).
3. Attributability grep over `agautolab/src` and `agautolab/agent`:
   `agforge`, `assetplan`, `ASSET_`, `[Asset]`, `asset_gate`,
   `mentions_us` all gone. forge is known only through the harvest.
4. agforge code unchanged (intro text may change).

Constraints (deliberately minimal):

1. Secrets in `.local/`; pyagag → push → re-lock in every consumer.
2. agautolab: push to GitHub; `--limit agstudio` deploy keeps the live
   checkout honest; agautolab1 optional.
3. Do not economize on runs — a delegation is two agents conversing, and
   that is the point.

## Step 1 — Lift the harvest into pyagag

`agfront/src/agfront/agents_md.py` (`harvest_intros`, `render_agents_md`,
`write_agents_md`) is the second consumer's dependency now. Move it to
`agag.intro` beside `post_intro`; **switch agfront to it in the same
change** (no "later"). Keep the shape: latest post per `intro-*` topic in
`#agents`, verbatim body, generated-at line, honest "no agents known" when
empty. Tests move with it. Push, re-lock agfront and agautolab.

## Step 2 — Delete the special route in autolab

Inventory (2026-08-21): `mission.py` — `ASSET_LABEL`, `ASSET_MARKER`,
`AGFORGE_SOURCE`, `ASSET_TOPIC_PREFIX`, `strip_asset_marker`, `is_asset`
plumbing on Work/TaskChange, `asset_topic`, `asset_order_key`,
`asset_order`, `asset_answer_context`; `zulip_listener.py` —
`ASSETPLAN_TOPIC_PREFIX` in the sweep tuple, `ASSET_PROMPT_NOTE`,
`asset_order_text`, `asset_gate` and its call in `serve_run`,
`mentions_us` and the whole "answering agforge's questions" handler, the
`assetplan_answer_superdirector` guide folder and its role. Plus ~50 test
references. Delete, don't stub. The `[AUTO]`/`FORGEAUTO` Plane markers are
forge's own work-selection vocabulary and stay.

## Step 3 — Give the runs what Front has

- Before a superdirector (workplan) run and before a supercoder (workrun)
  run, write `tools/agents.md` into that run's workspace via the shared
  harvest. Both roles need it: one to plan the delegation, one to perform it.
- Run env: `AGENTCHAT_ZULIP_ENV` → autolab's own `.local/zulip.env`, so the
  supercoder speaks as `autolab-agstudio1`. `agentchat` is on PATH once the
  new pyagag is locked in.
- Timeouts: forge's path can take `360 + 900 + 1200` s worst case against
  the supercoder's `WORK_TIMEOUT_SECONDS = 1200`. Options, implementer's
  call: raise the supercoder ceiling for this phase (the p5 precedent:
  `FRONT_TIMEOUT_SECONDS 360 → 3600`), and/or rely on the standalone-task
  shape plus re-trigger to resume (`agentchat read --since` makes the
  supercoder's resumption possible, the topic history is its memory).
  Record which it was and how long the real waits were.
- Guides (developer edits, as before): superdirector — "agents listed in
  `tools/agents.md` can be delegated to; write who and what into the task,
  and make a delegation its own task"; supercoder — "a task that delegates
  is a supervision: post, wait, answer, and say when the request is
  complete; bring the result URL back into this topic".
- Read forge's intro as its new reader would — a supercoder mid-task. The
  p5 fix for autolab's intro (one topic one task; step approval is not
  completion; how a topic closes) probably has a forge counterpart; if the
  intro does not say how an `assetplan-` is triggered into `assetrun-`, how
  long to expect, and what "done" looks like (URL + Work Done + `✔`), add
  it and re-post — a forge intro change is data, not the code freeze of
  criterion 4.

## Step 4 — Prove and report

1. Developer → Front: a mission for the test project that needs one new
   image asset and one music loop (e.g. a new enemy sprite and a stage BGM).
   Permit. Check the plan: two delegation tasks naming agforge, requests in
   words (criterion 1).
2. Front supervises the workruns (p5 shape). Watch the delegation tasks: the
   `assetplan-` topic under autolab's identity, forge's questions answered
   by the supercoder, `assetrun-` fired, URL returned, asset committed into
   the project, `✔`.
3. Greps for criterion 3 and a `git log` of agforge showing no code commit.
4. `report.md`: links (front-*, pj-<slug>, work-<label>, agforge-agstudio1,
   #agents), per-delegation wall clock and whether any run ended mid-wait,
   the harvest-lift commits, what the guides needed, and deferred items —
   video, execution-time delegation (not in the plan), method-2 hub,
   forge-side topic-scope lessons if they surfaced.
