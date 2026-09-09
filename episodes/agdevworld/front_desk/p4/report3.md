# Step 3 — integrate, verify live, deploy

agdevworld only; relay and web image redeployed on agstudio. Three live
completions, each from the view the plan named, each read back
independently; two relay rules were learned from them and pinned.

## Tests and build

`agentroom` `uv run pytest -q` → **251 passed** (5 added in this step:
the post-write plan, the row a close made unreadable, `#front` filed by
roster prefix, the archived-channel topic, resolved work on request).
`npm run build` clean. Focused relay tests for the generalized entry point
cover shared-request exclusion (`test_another_request_of_every_kind…`,
`test_a_plan_anchored_to_another_front_conversation…`), unfinished Plane
children (`test_an_unfinished_sub_work_blocks_its_mission`), a changed
preview (`test_a_plan_that_changed_since_the_preview_refuses…`, and the
409 over `/complete`) and retry after partial failure
(`test_a_retry_finishes_what_is_left…`,
`test_the_answer_to_a_close_is_the_plan_as_it_now_stands`).

## Deployment

`nctl drift`: every agdevworld/agent service converged (45 converged, 2
drifting elsewhere, unchanged by this work). The relay `com.agdev.agentroom`
was kickstarted with the p4 code (no plist change: the p3
`AGENTROOM_PLANE_ENV` is the last credential it needed); `/complete*` and
`/work?resolved=1` answered once the sweep was live. The web image on
`:8090` was rebuilt three times as step 3 fixed what the live runs found.

## Live, from the GUI

| request | from | preview | applied | read back |
|---|---|---|---|---|
| `front-routine-study-realworld-2026-09-08T16:47Z` (routine run) | operation room › study-realworld › Show resolved › run › **finish ✔ this run** | *1 to change · 5 already*: S4-3/S4-4 Done, plan and task ✔, **`#work-s4-3` will be archived**; *mentioned, not touched: routine-study-realworld* | **CLOSED — 1 changed, 5 already were** | channel gone from `client.channels()`; S4-3, S4-4 `completed`; `routine-study-realworld` open, `retired False`, schedule `next None` as before; a second preview after a page reload says *0 to change · 6 already*, the channel *already archived* |
| `pj-studyrealworld/workplan-publish-realworld` (Autolab request) | agent room › open work › ✔ resolved: shown › pj-studyrealworld › **finish ✔** | *1 to change · 5 already*: S4-5/S4-6 Done, both task topics ✔ (the re-run *also served on behalf of run 17:09Z — not touched*), **`#work-s4-5` will be archived**; *opened for run 16:57Z — stays open* | **CLOSED — 1 changed, 5 already were** | channel gone; run 16:57Z untouched |
| `front-p2-greet-agecho` (ordinary Front conversation) | agent room › front-agstudio1 card › its `#front` rows › **finish ✔** | *2 to change*: `#agecho-agstudio1 › hello`, then the conversation | first click **partially closed — 0 changed, 1 failed, 1 left**: `PATCH messages/1351 → HTTP 400: Invalid message(s)`, the conversation kept open; after the relay fix, second click **CLOSED — 1 changed, 1 kept** | `✔ front-p2-greet-agecho`; the hello topic left as it was |

Also previewed live, read-only: the desk conversations of p3 (every action
`done`), the standing request `routine-ghtrends` (refused, *retires the
routine*), the task topic `workrun-rerun-task1-s4-5` (not closable, both
parents named, the plan first), and six more `front-*` conversations from
August (two reach task topics in already-archived `work-` channels, now
shown *kept*).

## What the live runs taught (and what changed for them)

- **The plan after a close must be the plan as it now stands.** p3 answered
  a close with the pre-write actions and their fingerprint, so *try the rest
  again* could only ever be refused (step 2's browser fixture found it). Now
  the relay re-derives the plan after writing and lays the results over it.
- **…but a target the close made unreadable must keep its row.** The
  first live close from the agent room archived `#work-s4-5` and then could
  not read its task topics, so its own answer lost the channel row and
  reported *sub-works never reached: S4-6*. The pre-write rows now stay,
  marked done, with the fresh plan's fingerprint.
- **An ordinary Front conversation was unreachable.** The agent room filed
  an agent's conversations under a channel of its own name; Front's does not
  exist. `#front` is now walked and its topics filed under the instance whose
  roster **prefix** they carry — never guessed from the name.
- **A topic in an archived channel cannot be resolved**, however readable
  it is; the realm answers 400 on the move. `closing._mark_archived` reads
  the channel list once when an unresolved related topic exists, and such a
  topic is *kept — the channel is archived*, so the request closes over it
  instead of staying open for something nobody can do.
- **`front-routine-…T16:47Z` exists under both names** on the realm — a post
  landed under the bare name after the ✔ (the stray twin). The completion
  reads both and merges; nothing here tries to fold twins.

## Not exercised live

- **A Forge request with an open run.** Every `assetplan-` on the realm is
  ✔ and the August ones have no `assetrun-` at all (before forge opened its
  own run topic); `assetplan-red-apple` previews as *already ✔*, its parent
  named, nothing to change. `tests/test_scope.py` covers the plan+run pair
  and a Forge Work.
- **Un-resolving after completion**, as in p3: a post into a completed
  request buys a paid Front run. Checked in the code path only.
- **The screenshots** in `.local/shots/fd4/` were captured but not looked
  at by me in this session (the image read was declined); every claim above
  is the DOM text read at that step, or a realm read.

## Left standing

- The relay's operation history is memory: four kickstarts today emptied
  it four times. The realm and Plane are the record.
- `front-schedule` and `front-20260821-p9-assets*` reach `workrun-` topics
  of archived channels and would close leaving them *kept*; whether those
  August conversations should be closed at all is the human's call, not the
  button's.
