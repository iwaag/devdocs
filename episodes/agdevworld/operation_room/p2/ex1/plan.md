# operation_room p2 addendum — confirming done rows (manual clearing)

On the p2 board, a row whose ✔ rename the relay witnessed stays `done`
forever (the sweep never reads ✔ topics, so done only ever arrives as a
witnessed transition, and rows pile up until the relay restarts). Make these
clearable by **a human declaring "seen"**.

Policy: manual confirm, not timed eviction — matching the board's principle
that evidence stays until a human has looked at it. **Clearing everything at
once is the primary requirement.**

## Relay side: `POST /ops/confirm`

- No body (or `{"all": true}`) = confirm every current done row. Per-row
  confirmation (`{instance, channel, topic}`) is optional — clear-all alone
  satisfies the requirement.
- Keep the confirm record in the relay's memory. Its lifetime then matches
  the done rows themselves (both vanish on restart) and it is consistent
  across browsers. Do not persist it.
- This is a write to the relay's own memory, not to Zulip. p2 constraint 5
  (the observer never posts) is untouched. It adds one POST to a GET-only
  window, but this is a private loopback service — no auth needed.
- **Implementation hint (discretionary)**: record a confirmation not just as
  `(channel, bare_topic)` but paired with the latest message id at that
  moment. That buys, for free, the property that a confirm only hides the row
  *as it stood* — any later post to the topic (including an unresolve) makes
  the row resurface naturally. Avoid plain deletion (`del` from `_topics`):
  a later unresolve rename would hit `_apply_update` with an unknown old key
  and be ignored, silently missing the reopen.

## Which rows are clearable (the one rule to keep)

Only **`done` rows** may be confirmed (`unknown` rows with
`stale_state == "done"` may be included). `stalled` / `awaiting` / `acked`
must not be clearable — a button that removes a live debt from sight is a
machine for reproducing p9's "26 minutes nobody noticed" with one click.
Check the state on the relay side and refuse (do not rely on the frontend
hiding the button).

## Frontend side

- Show a "✓ confirm N done" button in the chip row only while at least one
  done row exists. Clicking it does `POST /ops/confirm` → reload.
- Per-row buttons are discretionary (the `row.actions` mechanism has a
  precedent in the tasks view). If added, use this verification to close the
  hole agent_room step 5 left open: cards with action buttons were never
  screenshot-verified.
- **Fix at the same time**: exclude done from the subtitle's "N rows open"
  count (resolving the drift between the word "open" and reality; a small fix
  already flagged in the p2 review).

## Verification

Reuse the p2 step 5 recipe in `#ops-testbed`:
artificial stall → reply/✔ makes it `done` → confirm-all removes it →
(if the message-id scheme was adopted) a further post to the same topic makes
the row resurface. Screenshot the whole cycle, including the look of a card
with a button on it.

## Constraints (minimal)

1. Only done rows are confirmable; the relay refuses others.
2. The confirm record is in-memory only (no file persistence).
3. No writes to Zulip.

Everything else (endpoint shape, per-row confirm or not, button look and
wording) is discretionary.
