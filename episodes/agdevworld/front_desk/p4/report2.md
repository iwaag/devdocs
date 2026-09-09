# Step 2 — one completion flow in the existing views

agdevworld only. The relay gained one read option; the frontend gained one
shared view model, one DOM overlay, and three entrances. `npm run build`
clean, `agentroom` **247 passed**.

## What exists now

**One view model, two renderers.** `src/completionState.ts` holds the
`ag.completion.v1` types, the relay calls (`/complete/plan`, `POST
/complete`, `/complete/history`) and `planLines(plan, phase)`: every line a
completion panel shows, in order, with its tone and indent — the scope
sentence, parents as navigation, each action with its reason or result,
`unreached_children`, `visitors` (*also served on behalf of …*), the
routine's *mentioned, not touched* rows, exclusions, gaps, the relay's last
operation on this request, the note. `titleLine`, `summaryLine` and
`planButtons` are shared the same way. The Front Desk's Phaser panel
(`frontDeskClosePanel.ts`) and the new DOM overlay
(`completionPanel.ts`) both draw exactly that list, so the two screens
cannot say different things.

**The DOM overlay** (`openCompletionPanel({channel, topic})`) is what the
operation room, the agent room and the ops board open. Fixed on the right on
desktop, the whole frame under 720px; Escape closes it. A parent line is a
button that re-targets the panel on that request. A plan that comes back
after the request changed under it is dropped, never drawn (generation
counter, same as the Phaser panel). After a close it fires
`agdevworld:completed`, and every view on the page reloads.

**A refusal is a plan.** Both panels used to test `'error' in found`, and
a 409 refusal carries `error` beside its fresh plan — so p3's *"try the rest
again"* would have shown *the plan could not be read*. `isCompletionPlan()`
tells the two apart. And the relay's answer to a close is now **the plan as
it now stands**, re-derived after the writes with the results laid over it:
its fingerprint is what a retry approves. Pinned by
`test_the_answer_to_a_close_is_the_plan_as_it_now_stands`.

**Entrances:**

- *Front Desk* — `finish ✔` unchanged in place; `relaySource.closePlan/close`
  now call the shared operation with `#front › front-desk-<id>`. Draft and
  IME handling untouched (the panel never takes the textarea's focus).
  `&finish=1` opens the panel on arrival, for fixtures.
- *Operation room* — a **finish ✔ this run** button under the run
  conversation's note, for the selected session; its reason beside it
  (*previews first; closes this run, its topics, channels and Plane Works —
  not the routine*; disabled with *no run selected* / *a completion needs a
  live relay*). Selecting a different run closes an open panel.
- *Agent room* — a **finish ✔** button on every topic row of an agent's
  popup and of a board's popup (project boards included, so a project's
  `workplan-` is reachable, not only agent-owned topics). A new chip **✔
  resolved: hidden/shown** lists resolved topics too (`GET /work?resolved=1`;
  each row carries `resolved`, cards read *N OPEN · M ✔*), which is how a
  finished request is reached at all.
- *Ops board* — a **FINISH THIS REQUEST** section in the row popup, with the
  sentence that keeps the two verbs apart: *"confirmed" on the board only
  hides a done row here and writes nothing; finish ✔ previews and then
  resolves this request's topics, archives its work channel and marks its
  Plane Work Done.*

## Verification

Browser fixtures over CDP (`.local/opsshot.mjs`, `.local/deskshot.mjs`)
against a **fixture relay** (`.local/fd4/fixture_relay.py`, :8095): it
proxies every route to the live relay on :8094 so the boards show the real
realm, and answers `/complete*` from the p3/p4 test fixtures with the
requested root standing in for the desk conversation, one target failing the
first time. Vite on :5174 with `VITE_AGENTROOM_URL` pointed at it. Shots in
`agdevworld/.local/shots/fd4/`; the panel text was read back from the DOM
at each step:

| shot | entry | what the DOM said |
|---|---|---|
| `ops-b-preview` | operation room › Show resolved › run 15:14Z › finish | *6 to change · 3 already · 0 blocked · 0 kept*, the routine sentence, *mentioned, not touched: routine-ghtrends — standing request* |
| `ops-c-partial` | close 6 targets | *partially closed — 4 changed, 1 failed, 1 left; the request stays open*; the root *kept open: related work is still blocked or failed* |
| `ops-d-closed` | try the rest again | **CLOSED — 2 changed, 7 already were**; the channel *already archived*, the run *is ✔* |
| `ops-e-narrow` | 640×900 | the overlay takes the frame |
| `ops-f-switched` | select another run while open | panel closed, the newer run selected |
| `room-c-preview` / `room-d-partial` | agent room › autolab card › `status-studyrealworld-p1` › finish | *FINISH THIS CONVERSATION* (kind `topic`), then partial |
| `room-g-workplan-preview` | agent room › open work › pj-studyrealworld › `workplan-create` › finish | *FINISH THIS AUTOLAB REQUEST* — a project workplan reached from the room |
| `opsboard-c-preview` | ops board › row › FINISH THIS REQUEST | *FINISH THIS ROUTINE RUN* for `front-routine-ghtrends-…T07:00Z` |
| `desk-demo-preview` / `desk-demo-narrow` | `?view=frontdesk&demo=1&finish=1` | the Phaser panel over the shared lines, 1600 and 520 wide |

I could not open the screenshots in this session (the image read was
declined), so the table reports the DOM text read at each step, not what
the pixels show. The layouts follow p3's measured ones and the operation
room's CSS; step 3 looks at the deployed screens.

Not exercised here: the **✔ resolved: shown** chip against the live relay
(the running relay is still p3's and ignores `?resolved=1`; the room test
`test_resolved_work_is_listed_only_when_asked_and_says_so` pins the route),
and the Front Desk demo's own click-through (p3's `p3-a…f` set covers it;
the panel's rendering is the shared list).

## Left for step 3

- Deploy: rebuild the web image on `:8090`, kickstart the relay for
  `/complete*` and `/work?resolved=1`.
- Live: complete the study-realworld run 16:47Z (one ready action,
  `#work-s4-3`) from the operation room; the publish run 16:57Z or its plan
  (`#work-s4-5`) from the agent room; read back Zulip, the channel list and
  Plane independently; repeat the preview; reload.
- The fixture relay and vite :5174 are still running for step 3's fixture
  cases; stop them at the end.
