# Step 2 — the button, and the word "open"

Three changes in the frontend, one of which is not about confirm at all.

## 1. `✓ confirm N done` on the chip row

Shown only on the board view and only when there is at least one `done` row.
`POST /ops/confirm` with no target, then reload.

The count is taken over `shownState(row)` — `stale_state ?? state` — the same
reading the relay uses. While the queue is dead every row wears `unknown` and
carries its last verdict underneath; the receipts among them are still
receipts, and the chip still offers to clear them.

The view does **not** pre-filter what it sends. It asks for "all done" and
lets the relay decide; a refusal comes back as a 409 with a sentence, and that
sentence is put in the headline rather than swallowed by a reload that would
look like nothing had happened. A screen that decided for itself which debts
could be cleared would be a second opinion standing in front of the evidence,
which is the mistake the whole view is built against.

## 2. A `✓ confirm` button on each `done` card

Taken, not skipped. `row.actions` already existed for the tasks view, and one
card is one conversation, so the row-level confirm has an obvious home. The
card stays selectable — the evidence popup is the reason it is a card at all.

This is the discretionary half of the plan, and it carries the obligation the
plan attaches to it: `agent_room` step 5 left "action-button cards are not
screenshot-verified" as an open hole, and step 3 of this exercise is where it
gets closed.

**One layout change was needed to take it.** `PanelGridScene` drew action
buttons at a fixed `top + 64`, which is the bottom strip of the 84px task
card. The operation room card is 148px and its body is five lines of evidence
starting at `top + 52`; a button at 64 would have been drawn straight through
them. Buttons now sit at `top + height - 20`, which is the same pixel on the
task card and the bottom edge on the tall one. The title/status tightening
that `hasActions` used to trigger is now conditional on the card being short,
so the tasks view is byte-for-byte what it was and the operation room keeps
the spacing its three visual defects were fixed to in p2 step 4.

## 3. "N rows open" no longer counts `done`

The p2 review's small lie, and the reason the plan asked for it in the same
breath as the button. `done` is a receipt for a debt that was paid; counting
it under the word *open* meant the number grew all day while nothing was owed.

The subtitle now separates them:

```
3 stalled of 5 open rows · 12 done to confirm
nothing is owed a reply right now · 12 done to confirm
nothing is owed a reply right now
```

The done rows are still on the board — they are evidence and this board keeps
evidence — they are just no longer described as debt. The headline gained the
other half of the same honesty: `· N confirmed rows hidden`, so a board that
is quiet because somebody cleared it does not look like a board that was
always quiet.

## Proof so far

`npm run build` (which is `tsc && vite build`) is clean. That is the whole of
what a build can say here: every remaining claim in this step is about pixels
and clicks, and step 3 is where a browser is pointed at it.

(`npx vitest run` fails on `.local/verify-autolab.spec.mjs`, a stray Playwright
spec vitest picks up because it is not a vitest suite. Pre-existing, unrelated,
untouched.)
