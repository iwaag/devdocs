# progress_panel p1 — step 1: evidence map and display model

Date: 2026-09-27 (UTC 08:28–08:35).

## Deployment state at the start

- `nctl drift`: `converged=46`, no diff. Every launchd job is up
  (`agfront`, `agautolab` + gateway, `agforge` + service, `archsage`,
  `agobserver`, comfy notifier, the `agentroom` relay); `:8090` answers 200.
- The relay's `/healthz`: mirror `live`, Observer monitor `ok` (cycle 32,
  11 requests looked at, 2 tracked — both held — and 0 open incidents).
- Pins: pyagag `673e70d` in every consumer, the relay included (failsafe p4).

## One request, end to end

The representative request is the sage p2 study run
`#front › front-desk-20260926-sagep2-aisvgs` (o11522): Front → archsage
establishes the study → Front runs the study routine → autolab plans a
mission with one task → result → acceptance → archsage refreshes the sage.
`agag.trace` over a copy of the relay's mirror (no Zulip call, 9 reads):

```
front/front-desk-20260926-sagep2-aisvgs [Front]: ANSWERED
  … held by a conversation opened from it
  ! operation failed: #11687 Front: accept #11579 refused: #11522 is older than m11579 itself
  └ archsage-agstudio1/study-aisvgs [archsage]: AWAITING_REQUESTER   (taken up, served up to 11694)
    └ pj-aisvgs/workplan-setup-aisvgs [autolab]: AWAITING_REQUESTER  (taken up)
  └ routine-study-aisvgs/routinerun-20260926-2100 [Front]: NOT_STARTED
      … serving open since ack #11649; held by its owner
    └ pj-aisvgs/✔ workplan-aisvgs-round1 — mission m11579: DONE
      └ work-m11579/✔ workrun-task1-m11579 — task 11579#1: DONE (accepted)
```

What each hop leaves, and what the trace could read:

| hop | record | read by trace? |
|---|---|---|
| request | the origin's first post (o11522); Front's ack/replies; `ag-post` intent lines | yes (`human` root, `agag.outstanding`) |
| delegation to archsage | root note `[rootchat] front/… #11524`; archsage's answer; Front's `[served]` | yes; state + receipt |
| study setup | autolab's `workplan-setup-…`, answer line "study layout established" | yes (as a plain conversation, no identity note) |
| routine run | `routinerun-…` opened by Front with a root note, served by Front | **wrong state** (below) |
| plan | `[mission]` note = m11579, `[doc]` plan, `[state] started/done` | yes |
| tasks | every `task[N].md` becomes a `workrun-` topic at plan time (`plan_changes`/`mirror_task_changes`), `[task] <m>#<n>`, `[state]` words, result post `intent=report`, `[change] accepted … +shown= +checkpoint=` | yes: counts, states, acceptance |
| execution | ack `Message received`, `end=<ack>` on the closing reply (since failsafe p1), autolab's live execution record `executions/s<serving>-<role>-<ns>.json` | ack/end yes; the live record only through `agag.health` |
| result delivery | the requester's `[served] <remote> <id>` receipt | yes (`awaiting_delivery` until it) |
| acceptance | `[selfnote][acceptance] #<post> by <user>`, `[state] accepted/done`, ✔ | yes |
| knowledge refresh | archsage's prose reply #11694 ("sage:aisvgs is refreshed… revision moved from …") | **no record at all** |
| recovery | Observer: `incidents/<hash>.json` (kind, state, origin `o<id>`, node anchor), `health.json` per unit anchor (ack, verdict, first suspicion, checks), `held.json`, `retired.json`, `monitor-health.json`; the incident topic in `#agobserver-agstudio1` | not by trace; by files on this host |

A second request, o11711 (round 2, m11741, held for the Developer), shows the
case the panel must not get wrong: the trace reads task 1 as `EXECUTING`,
"last sign of work 18h50m ago; serving open since ack #11753; held by its
owner". The run died on 2026-09-26; only a health check (process gone) or
Observer's hold says so. A conversation-only view would draw a moving meter
for a dead run.

## Gaps found

| # | gap | consequence for a panel | where it belongs |
|---|---|---|---|
| G1 | A conversation its owner opened and serves itself (Front's `routinerun-`) classifies as `not_started` even after acks and replies: `classify` reaches "nobody has posted here since it was opened" because no *other* sender spoke and there is no `[start]` note. | every routine run reads "not started" while it works | pyagag `agag.trace` |
| G2 | `routinerun-20260926-2100` (and `-2225`) never ended: the mission was accepted from the desk conversation, the run's own `ag-routinerun` block was never written, so the run is open for ever and holds its parent ("held by a conversation opened from it"). | a finished study would never read complete; "what is open" lies | Front's routine path (record), shown as-is meanwhile |
| G3 | The sage refresh — the last required step of a study routine — exists only as prose in archsage's reply. | "knowledge refreshed" cannot be shown or required from a record | archsage `sage sync` writes a note |
| G4 | Execution health is exposed by autolab only (`health.toml` lists `autolab-agstudio1`). Front, archsage, forge and cagent are conversation-only. | their "working" is an unconfirmed claim | documented limitation; shown as such |
| G5 | An `open` serving is a claim: o11711's task reads `executing` 19 h after its process died. | must never become a healthy moving meter | panel rule + probe |
| G6 | Receipts for callbacks were once written in the run topic instead of the desk (#11572), which made Observer ask twice (#11615, #11636). Pre-failsafe; the p3 receipt rule covers it. | none now; noted | — |
| G7 | Trace uses 200-post histories; a long origin conversation's early asks are "bounded". | the card says "history bounded" | shown |

## Display model

### Card identity and scope

- **A card is a request, and a request is its origin conversation's first
  post in `#front`** (`o<id>`, Observer's `origin_key`). Every `front-*`
  conversation qualifies — Front Desk (`front-desk-<id>`) and ordinary ones
  alike, so requests made in different conversations sit side by side. A
  rename or ✔ changes only the label.
- **Scope, newest activity first:**
  - *active*: not ✔ and activity within 12 h (Observer's discovery window),
    or any unit of work below it unfinished by record, or tracked/held by
    Observer — whatever its age;
  - *recent results*: finished (everything below done/cancelled by record,
    or ✔ with nothing running) within the last 24 h, at most 6;
  - at most 16 cards; the **current conversation** is always included and
    pinned first, whatever the bounds. Desk conversations link back into the
    Front Desk (`?conv=<id>`); others link to Zulip.
- **One request's children stay under its card**: the trace tree below the
  origin. A plan is a node with a `[mission]` (autolab) or `[asset]` (forge)
  identity, or a Front routine run above one; tasks are its `[task]`
  children in serial order; delegations (archsage, Observer, …) are plain
  child conversations.

### Three facts per node, never merged

1. **Work state** — from the records (trace state + `[state]` words).
2. **Execution health** — `agag.health.v1` verdict for probed owners, with
   its observation time; otherwise "conversation only: serving open/ended".
3. **Recovery** — Observer's incident for this node (kind, state), a hold
   or retirement by a person, Observer's own health.

### User-facing states

| state | evidence |
|---|---|
| planning | a plan conversation exists and no task of it is known yet; or the request is served with no plan/delegation yet |
| queued | a post waits for its owner's ack (`queued`); a task not started while an earlier task is unfinished (its turn has not come); `not_started` owned by its owner |
| working | execution `open` — **confirmed** only by a probe verdict `running` within the freshness bound; otherwise "working (unconfirmed)" |
| waiting for another agent/tool | holder `delegate`, `waiting_on` a named agent/notifier, or a probe verdict `waiting` on a named tool call |
| awaiting your response | a response request to the viewer pending in the origin (`agag.outstanding`), or a request held by a person |
| completed | all units `done` by record, the answer taken up (served), and — for a study — the sage refresh recorded |
| cancelled | `cancelled`/`replaced`/`retired` by record |
| stopped | probe verdict `stopped`, a failure notice (`failed`), or an Observer incident of kind `stopped`/`unheld`/`unanswered` |
| unknown | unobservable conversation, stale mirror, probe `unknown`, holder `none`/`unknown` |

Stale or unavailable evidence overrides motion: a card whose mirror is not
live, or a run whose last health observation is older than its bound, is
drawn frozen with "last known at …", never with a moving band.

### Stages kept visible until their record

result delivered (`[served]`) → task accepted (`[state] accepted`) → plan
completed (`[state] done`) → knowledge refreshed (study only; G3's note).
An ended run or a delivered answer alone completes nothing.

### Meters

- **Plan meter**: completed tasks / current total (tasks not cancelled),
  segments for working and awaiting-acceptance tasks. While planning: no
  percentage, the word "planning". A changed plan (more than one `[doc]`,
  a cancelled task, a new serial) is said: "plan revised: 3 → 4 tasks".
- **Run meter** under each task: no determinate unit exists in any record
  today (task counts are the only counts), so runs are an activity band —
  animated only when a fresh probe says `running`/`waiting` — plus the
  current action from the live record (`last_work`, open tool call with its
  detail) and its age.

### Cost bounds

- The read model is computed off the relay's mirror (`MirrorReader`), so a
  view costs no Zulip call; cached and recomputed at most every 5 s.
- Probes only for open servings of owners Observer probes (its
  `health.toml`, the same command), cached per (owner, ack) for 15 s, run
  in parallel within a 5 s budget. The panel being shown or hidden starts
  nothing and changes nothing in Observer.
