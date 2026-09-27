# progress_panel p1 — step 3: the panel and its meters

Date: 2026-09-27 (UTC 10:00–11:05).

## What the Front Room shows now

A **`▦ progress`** toggle in the Front Desk's button row (the Arguing Room
does not get it). It carries the number of active requests and badges for
those that need you (`✋ n`) or have stopped (`■ n`). The panel opens in the
right column, beside the conversation; the history panel and it take turns
there. Its open state and which cards are expanded survive a reload
(`localStorage`, per browser).

The panel (DOM, `src/progressPanel.ts`):

- **status line** — when the board was observed, whether the realm's copy
  is live, Observer's monitor state, and which owners have health checks
  ("autolab-agstudio1 (others: conversation only)");
- **THIS CONVERSATION** — the request open in the room, pinned;
  **ACTIVE** — every other unfinished, held or tracked request;
  **RECENT RESULTS** — finished within 24 h, at most 6;
- **a card** per request: a state chip with icon and words (`✎ planning`,
  `⋯ queued`, `▶ working`, `⏳ waiting`, `✋ needs you`, `💬 answered`,
  `✓ completed`, `⊘ cancelled`, `■ stopped`, `? unknown`), its title, tags
  (`open here`, `✔`, `last known`, `held by a person`), the reason, whose
  move it is, last work and last post ages;
- **a plan meter** per plan: one segment per task in serial order
  (agreed = green, in progress = blue, awaiting agreement = amber, stopped
  = red, no evidence = grey hatching, not started = empty), the words
  "2/4 tasks agreed · 1 in progress · 1 awaiting agreement", and a note when
  the plan changed its denominator ("plan revised 1×: 3 → 4 tasks"). While
  planning there is no bar and no number, only `PLANNING`;
- **stages** as chips until their records exist: tasks agreed, plan
  accepted, run ended, report delivered, knowledge refreshed;
- **child work** (expandable): each plan, task, routine run and delegated
  conversation with its own chip, reason, `next · record · serving ·
  holder`, Observer's incident or a person's hold, `opfail` notes, and a
  link to its evidence post in Zulip. Under a task with a serving, a **run
  band**: `active` (sweeping), `waiting on a call` (slow pulse), `open
  (unconfirmed)` (dotted, still), `stopped`, `ended`, `no evidence` (grey
  hatching) — with the current action and its age and where the fact came
  from ("health check 08:59:10Z (11 s old)" or "conversation only");
- **links**: a Front Desk card opens its conversation in the room; every
  card and unit links to Zulip.

Motion is fresh evidence only: a band or segment animates only when its
health check is current, never on a conversation's claim, and nothing
animates while the board is stale or the relay is unreadable
(`prefers-reduced-motion` stops it altogether). When the relay stops
answering, the last board stays with "showing the last board, read HH:MM:SS
(n s ago); nothing below is current".

Showing or hiding the panel only decides whether the page asks the relay
for the board (every 8 s open, every 60 s closed for the toggle's badges);
the board is a read of the relay's memory. Nothing else changes.

## Found and fixed while building it

- **An `ended` health check beside an unended serving** (m11741's task,
  live): the probe says the run is over and its journal delivered
  #11766 (a progress post, pre-failsafe), but no post ended the serving.
  It had read `unknown` for the conversation's reason while the run band
  said "ended (health check)", and the meter counted it "1 in progress".
  Now the unit says both facts ("health check: … — but no post here ended
  the serving acked at #11753"), and the meter counts `stopped` and
  `unknown` tasks apart from those in progress (pyagag `0396b24`, test
  added).
- **The toggle grew over its neighbour** when a badge appeared (it was
  placed before its width changed); it is re-placed on every change.
- **A narrow window** had no room in the button row (the toggle was off
  screen), and putting it in the prompt bar left the composer 70 px wide.
  On a narrow screen it takes the row's left end in a compact form, the
  settings button that no longer fits there is hidden, and the composer
  keeps 146 px (asserted ≥ 120).

## Checks run

- `npm run build`: clean.
- `.local/pg/s1-demo.mjs` (`?view=frontdesk&demo=1`, 16 assertions, all
  pass): hidden at first; two cards, the open one pinned; planning shows no
  numbers; tasks known 0/3; the other request waits on its tool call; the
  plan changes 3 → 4 (1/4); a stale check reads unknown and does not move;
  a stopped run; request A completes 4/4 while B needs you; an unreadable
  relay keeps the last board frozen; show/hide survives a reload; the panel
  fits 390×800 and its toggle is on screen with the composer ≥ 120 px.
- `.local/pg/s2-live.mjs` against the deployed relay (kickstarted for the
  new route): ten cards off the real realm in ≈1.5 s, no Zulip read; the
  held round-2 request pinned as `needs you` with its task `unknown`.

Screenshots (ignored): `agdevworld/.local/shots/pg/` — `02-planning`,
`03-working-expanded`, `04-revised-stale`, `05-stopped`,
`06-completed-needsyou`, `07-relay-down`, `08-reloaded`, `09-narrow`,
`10-narrow-hidden`, `20-live`, `21-live-scrolled`.

## Commits

- agdevworld `8f3ebdd` (`progressState.ts`, `progressPanel.ts`,
  `FrontDeskScene.ts`), pj-agdev `382b166`;
- pyagag `0396b24`; the relay's lock is at it and the relay was
  kickstarted (twice, for the route and for this fix).

The web image on `:8090` is not rebuilt yet (step 5's deployment); the
checks used vite on `:5179`.
