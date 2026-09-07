# Operation room p6 — routine usability

Make routines startable from the dashboard, make the flow and routine list
compact and recognizable, and hide resolved sessions by default.

This is a destructive phase in a private experimental environment. Backward
compatibility is unnecessary. Refactor APIs, components and data structures as
useful; the steps describe outcomes, not mandatory implementation recipes.
UI text is English. Main implementation: `pj-agdev/agdevworld`.

## Step 1: define session identity and resolution

- Keep a fire message ID as the session identity. Return explicit start/end
  message boundaries so chat highlighting does not depend on the visible
  session list's order. Distinguish manual and scheduled starts where known.
- Add session resolution and its evidence in the relay. `done` currently
  means a fire received an answer, not that the session finished. The shared
  fire topic's current resolve flag cannot resolve every historical session.
- Associate explicit resolve evidence with the relevant fire ID. Decide how
  to retain that association across relay restarts; a small local ledger is
  an option. Account for reopening and a later fire in the same topic.
- Historical evidence that cannot identify a session remains unknown. An
  unanswered lookup, no child nodes, or all *visible* children being resolved
  is insufficient evidence of completion. Expose reconstruction limits.
- Record the chosen completion semantics, storage and history limitations in
  the step report before wiring the filter. No general execution engine or
  migration of all old history is needed.

Pointers: `agentroom/src/agentroom/routines.py` owns fire recognition and the
session walk; `ops.py` owns topic/event observations. `sessions_of()` currently
returns only three sessions, and routine history is bounded to 200 messages.
Resolved child topics are skipped during startup sweeps, so a linked topic can
be known only from a note. Topic state is current evidence, not a historical
session outcome.

## Step 2: add New session

- Add a New session action for the selected routine, separate from ordinary
  chat. Show the standing request and allow an optional instruction if useful.
- Generate the start message in the relay and send through the existing
  Developer chat client. Support the first fire when no fire topic exists.
  Return the message ID, show pending event reflection, then select that fire.
- Keep ordinary chat as a continuation. Starting manually leaves the next
  scheduled event unchanged. Make retired/unstartable routines understandable
  and decide their interaction in the UI.
- Disable duplicate submission while sending. Report an uncertain send result
  without automatically reposting. Preserve user input on failure.

Pointers: `devenv/routine/trigger.sh` supplies the existing fire wording;
`FIRE_LINE` recognizes `Routine …, run of …`. Prefer a shared formatter or a
small explicit contract test over two drifting formats. The standing request
is the newest post by its original author, not necessarily the latest post.
`chat.py`, `server.py`, `routineState.ts` and `operationDashboard.ts` are the
existing send/controller seams. Extend or replace their API shape as needed.

## Step 3: hide resolved sessions by default

- Add Show resolved, initially off, and persist the preference in the browser.
  Hide only positively resolved sessions; unknown sessions remain visible.
- Filter before taking the latest three visible sessions. Expand the relay
  response or add a query option as appropriate; disclose the available
  history limit instead of implying an exhaustive search.
- If a selected session becomes hidden, select the newest visible session or
  show a clear empty state. Keep routine selection and chat drafts stable.
- Use explicit session boundaries for chat highlighting. Host observations
  remain attached to the actual latest fire, not the first filtered result.

## Step 4: make the flow compact

- Add Compact / Detailed, defaulting to Compact; persist the choice. Keep the
  existing detailed presentation available.
- Compact nodes show an icon, short label and state. Selection reveals full
  channel/topic names and evidence. Use text or symbols as well as color.
- Share graph topology between modes; vary card dimensions and spacing.
  Preserve branches, scrolling, selection and refresh behavior. Keep unknown
  health and truncation visible in both modes.

Pointers: `src/sessionGraph.ts` uses 300px cards with 360px horizontal and
196px vertical spacing. Parameterizing layout and rendering is a straightforward
starting point. `src/operationParts.css` owns the dashboard presentation.

## Step 5: make routine names recognizable

- Show an icon and human-readable title prominently, with the internal name
  available as secondary text. Keep routing keyed by the internal name.
- Support optional display metadata. Choose its simplest appropriate home;
  fall back to a standing-request heading, then the internal name. A new
  routine should remain usable without adding a hardcoded name mapping.
- Keep the list focused on identity, state and next scheduled time. Put
  detailed timing and provenance in the selected view or an expansion.
- Check long names, retired routines, keyboard navigation and narrow screens.

## Step 6: verify, deploy and report

- Add focused relay tests for manual fire recognition, first fire, resolution
  association/reopening, restart behavior and filtering before the limit.
  Mock Zulip writes. Run relay tests and `npm run build`.
- Exercise the assembled UI: new-session selection, ordinary continuation,
  filter changes, chat boundaries, latest-only host evidence, retained drafts,
  relay outage/recovery, both graph modes and a narrow viewport. Include a
  large branching graph and incomplete reconstruction in visual checks.
- Check current service state through Nautobot/nctl. Planning found the relay
  live and chat configured, but nctl reported `agfront: service_missing`;
  determine whether that is stale observation or an actual start blocker.
- Rebuild the served web image and restart the relay when changed. Commit/push
  changes and document outcomes, remaining limitations and any API decisions.
  Make any real start test deliberate and within the user's authorized scope;
  it runs the routine's actual work. Automated tests use the mocked send path.

## Minimal constraints

- Use the Developer write path; keep the observer credential for observation.
  Do not commit credentials or machine-specific environment details.
- Reuse the relay's event reconstruction; do not add repeated Zulip sweeps for
  UI refresh. Existing topic-list evidence can be retained more effectively.
- Unknown is not idle or resolved. Compact rendering and filtering must not
  conceal unavailable evidence or reconstruction limits.

Everything else, including module boundaries, storage, API shape and visual
design, is the implementer's choice. Prefer the smallest coherent solution.
