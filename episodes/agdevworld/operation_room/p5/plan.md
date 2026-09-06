# operation_room p5 implementation plan — dashboard assembly

Assemble p4's parts into the finished operation room dashboard: left routine
list, recent sessions (max 3) on top, the conversation-flow graph as the main
pane, chat on the right. p4 established there are no technical blockers; this
phase is joinery plus the lifecycle decisions p4 deliberately left open.
UI text is English. Destructive phase; the minimal constraints are at the end.

## Starting position

- `/?parts=shell` — the four-pane DOM frame with routine/session selectors,
  loader counts, failure handling (`src/operationParts.ts`, `.css`).
- `/?parts=graph` — the SVG/DOM session graph, stress-proven on the
  28-conversation outlier (`src/sessionGraph.ts`).
- `src/routineState.ts` reads, `chatPanel.ts` + `POST /chat` (round trip
  proven in p3), and the p4 signal-mapping table with its explicit drop list.
- The seven Phaser views still live at `/`.

## Step 1: join the panes

- Mount the session graph and the chat panel into the shell's slots. Rework
  `chatPanel.ts` styling as needed to live in a pane instead of an overlay.
- Bind selection across panes: choosing a routine on the left drives the
  session list, the graph, and the chat pane (the routine's fire topic);
  choosing a session drives the graph and, where sensible, highlights the
  corresponding chat span. One selection model owns this — don't let each
  pane keep its own idea of what is selected.
- Fill the routine/session placeholders with the mapped presentation from
  p4's table: standing request, time since fire (never "execution
  duration"), state chip, schedule evidence (`next`, `overdue`), and the
  **overdue-ack annotation** (`acked` older than the stalled threshold —
  an annotation, not a new state).

## Step 2: the refresh lifecycle (the main open decision)

p4's prototype was snapshot + explicit refresh. Decide the dashboard's
lifecycle here, deliberately:

- **Polling the relay is free** — its reads are answered from memory that
  the event queue keeps current; no Zulip call results. The standing
  prohibition is on new *Zulip* polling, which this is not. So a modest
  frontend interval (a few seconds) against `/routines`, `/routines/<name>`
  and `/ops` health is the simple, safe default; pick it unless something
  better is equally simple.
- The in-flight read (`/inflight/<name>`) is the one signal with a real
  polling cost (host directory scans). Keep p3/p4's rule: poll it only for
  the selected routine, latest-only, and never attach it to historical
  sessions.
- Whatever the choice, the relay-health line stays on screen: when the event
  queue is dead the panes show unknown, not stale-as-fresh (`stale_state`
  semantics carry over).

## Step 3: honesty details carried from p4's boundaries

- **Cap exhaustion is shown, not hidden**: when a tree hit the relay's
  reconstruction bound (40 nodes / four hops), the graph says it was
  truncated. An untruncated-looking partial tree is the same defect as a
  wrong state.
- Current topic verdicts are not historical session outcomes; manual
  activity without a dispatcher fire is not an identified scheduled run.
  Both distinctions survive into the final copy (labels, popups).
- Graph refinement against the real outlier: branch organization, edge
  routing, full-name presentation, narrow-screen behavior. No fake stages,
  no percentages — the drop list is settled and stays settled.

## Step 4: placement and the fate of the old views

- Decide the dashboard's entry point: promote it to the default page, keep
  it at its own URL, or make it the eighth view. Implementer's choice, but
  record it and its reason.
- Decide what happens to the Phaser `routines` view (largely superseded by
  the dashboard) and the `ops` board (its stalled-first list is distinct
  value — presumption is it stays, possibly linked from the dashboard's
  health line). Retiring a view is fine in this phase as long as the report
  says so.
- Update README_DEV.md, and **rebuild the :8090 image** ("keep it current
  when a user may want to look" — the stale-container lesson from p2 is
  fresh).

## Step 5: verification

- Screenshots throughout (`.local/opsshot.mjs`); the visual pass has caught
  defects in five consecutive phases and the assembled dashboard is the most
  visual artifact yet. Include: the outlier session, a narrow viewport, the
  relay-down degradation, and the truncation marker.
- **At most one real chat round trip** through the assembled chat pane —
  the send path is proven, but its new wiring inside the shell is not. Same
  rules as p3: deliberate, once, no automated posting; unit tests mock Zulip.
- Selection-binding checks via CDP: routine click → all panes agree;
  session click → graph and chat agree; refresh does not lose the selection.
- `npm run build`, relay tests, and a `launchctl kickstart` deploy of any
  relay change.

## Constraints (minimal)

1. No new Zulip writes beyond the existing chat door; no new Zulip polling.
   Polling the *relay* is allowed and expected.
2. unknown ≠ idle, provenance on screen, truncation shown — on every pane.
3. The p4 drop list stays dropped (no progress %, no pipeline stages, no
   "next step").
4. Real posts in verification: manual, at most one round trip.
5. UI text in English. No credentials committed.

Everything else — refresh interval, entry-point choice, which old views
survive, styling fidelity to concept.png — is the implementer's discretion,
recorded in the report.
