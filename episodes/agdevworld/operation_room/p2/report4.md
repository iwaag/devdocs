# Step 4 — the operation room view

The sixth view in agdevworld, after `nodes → workspaces → autolab → tasks →
agentroom`. `PanelGridScene` reused, as the agent room reused it; the two
options it grew for that view (`panelHeight`, `nameFontSize`) were enough
again, and no third one was needed.

## What is on it

**owed replies** (default) — one card per `(instance, channel, topic)` with
something outstanding, stalled first. The card is named after the **instance**
and its body carries the conversation and the evidence:

```
🛑 agecho-agstudio1
   STALLED · ops-testbed
   agechoplan-opsroom-bo…
   4 min unanswered · #4939 by Provisioner · no served note
```

**agents** — one card per instance, its counts, and the routing it declares:
`channel agforge-agstudio1 · assetrun- assetplan-`, or for Front
`no channel of its own (declares front-agstudio1) · front-`.

Clicking either opens the whole of it: the full provenance sentence, the
message id and time, who posted, whether a served note covers it, the live
topic name, and the relay's own health beneath — because a row's meaning
depends on whether the relay could vouch for it at all.

## The two required things

**1. `unknown` is a state, not an absence.** It is **amber**, deliberately not
the grey this app uses for `idle` in every other view. Three separate cases
reach it and each says which:

- an instance whose introduction carries no roster block — *"introduction
  carries no roster block"*, and the popup explains that the observer refuses
  to guess because guessing is what produced 66 phantom rows;
- a dead event queue — every row's state is rewritten to `unknown`, the last
  verdict kept as `stale_state`, and the caption becomes *"relay not reading
  Zulip · last known stalled"*;
- an unreachable relay — the view renders **a card**, not an empty grid, and
  the headline reads *"⚠ UNKNOWN — the agentroom relay is not answering on
  http://localhost:8094. Every row below is the last thing known, not the
  state now."*

That last one is the case that matters most and is easiest to get wrong: an
exception that empties the grid produces a calm, green, empty board, which is
exactly the failure p9 taught. The screenshot of it is `11-ops-relay-down`.

**2. Provenance on every row.** The relay writes it — a full sentence and a
card-sized short form — and the view renders both without reading anything out
of either. No state is decided in the browser. A second copy of the rules there
would drift from the relay's, and that drift is p1's whole story.

## Three defects, all found by looking

The plan says to apply `agent_room` step 5's lesson directly, and it paid
again: the build passed, 34 relay tests passed, and the payload was correct
through every one of these.

**1. The provenance sentence overflowed the card.** The status line wraps at
about 28 characters, so *"stalled — 17 min since Developer's post #4931
(threshold 15 min), no served note, owner route"* rendered six lines and spilled
through the rounded border. Fixed in the **relay**, not the view:
`provenance.short` is served beside `provenance.text`, so both the board and
the popup get the length each can hold, and the shortening is done where the
state is decided rather than by the browser guessing at a substring.

**2. Four different rows rendered as four identical titles.** Naming the card
`ops-testbed/agechoplan-opsroom-…` clipped every one of them to the same
prefix — two topics for two instances, and the card said which of neither. The
card is now named after the **instance**, which is short and always distinct,
with the conversation on the first body line.

**3. The topic line ran into the neighbouring card.** A `channel/topic` name is
one unbroken token and Phaser's word wrap only breaks on spaces, so it did not
wrap at all: it drew straight out through the right border. Hard-clipped per
line — 52 overflowed badly, 27 and 24 each still reached the border, and 22 is
what the line actually holds at 10px. Measured three times, in three
screenshots, because there is no other way to measure it.

A fourth, smaller one: the popup header read `agping-agstudio1
agping-agstudio1` for a row with no conversation behind it, and offered a table
of eight em-dashes as its "evidence". The header's second slot now says
`standing`, and a row with no post behind it shows the sentence alone with a
line saying why there is nothing to show.

## Regression

Cycled all five older views with the relay down: `nodes` (6 cards, 3 converged
3 unknown) and `workspaces` render exactly as before; `autolab` and `tasks` are
in their documented no-backend states; `agentroom` says *"the agent room is
unreadable — the agentroom relay is not answering on http://localhost:8094"*,
which is its own documented behaviour and the correct answer while the relay is
stopped. The agent room's `⇄` label now reads `operation room`, closing the
cycle.

`PanelGridScene` was **not** modified for this view.

## Changed

- `agdevworld/src/opsState.ts` — new, the `/ops` read and its types.
- `agdevworld/src/views.ts` — `opsViewConfig`, three new `PanelSelection`s.
- `agdevworld/src/detailPopup.ts` — the row, instance and health popups.
- `agdevworld/src/viewSwitcher.ts`, `src/main.ts` — the sixth view in the cycle.
- `agdevworld/.local/opsshot.mjs` — the CDP driver (ignored).
- `agentroom/.../ops.py` — `provenance.short`, and the shared-prefix note.
- `agdevworld/README_DEV.md`.
