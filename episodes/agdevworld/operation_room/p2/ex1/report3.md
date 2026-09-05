# Step 3 — the whole cycle in `#ops-testbed`, and what looking at it found

p2 step 5's recipe, extended by the two moves this exercise adds: confirm, and
then prove the row can come back. Relay run at `AGENTROOM_STALLED_SECONDS=60`
so a transition could be watched rather than waited out, restored to 900 after.

The artificial situation is the same safe one: a topic named
`agechoplan-ex1-confirm` posted by the Provisioner into `#ops-testbed`, a
channel no agent is subscribed to. Two agecho instances both declare the
`agechoplan-` prefix, so the board correctly shows **two rows for one
conversation** — p2's shared-prefix finding, still true, and it turns out to
matter for confirm (see §3).

## 1. The cycle, on the wire

```
posted #4946
t+4s                  awaiting  0 min unanswered · #4946 by Provisioner
t+40s                 awaiting  1 min unanswered · #4946
t+75s (>60s)          stalled   1 min unanswered · #4946
--- confirm all, with only stalled rows on the board ---
                      200 {'confirmed': 0, 'topics': [], 'refused': []}
--- confirm that one topic, explicitly ---
                      409 {'refused': ['stalled'],
                           'error': 'only done rows can be confirmed; stalled is still owed'}
                      stalled   2 min unanswered · #4946      (unmoved)
--- ✔ rename ---
                      done      ✔ resolved · last post #4946 by Provisioner
--- confirm (clicked in the browser, see §2) ---
                      rows=0  hidden=2
--- a new post into the ✔ topic, #4948 ---
                      done      ✔ resolved · last post #4948   (both rows back)
--- confirm again, then un-✔ the topic with no new post ---
                      rows=0  hidden=2
                      awaiting 0 min unanswered · #4948        (both rows back)
health: live · sweeps: 1 throughout
```

Everything above came off the event queue: `sweeps` stayed at 1 from the
startup sweep to the end, so `awaiting → stalled → done → hidden → back` is all
`message` and `update_message` events. The confirm itself costs zero Zulip
calls, which is the whole point of it living in the relay's memory.

**Two refusals, and the difference between them is deliberate.** "Confirm all"
with nothing done is `200 confirmed: 0` — the caller asked for the done rows
and there were none; the stall beside them was not what was asked for and is
not an error. Naming that topic explicitly is `409`, because then the caller
*did* ask for a live debt and has to be told no. The board did not move either
way.

**The unresolve came back with no new post at all.** This is the trap the plan
named. The row stopped being `done`, and the hide only ever applies to `done`,
so it reappeared as `awaiting` — no special case, no re-sweep. The cheap
implementation (`del self._topics[key]`) would instead have lost the un-✔
`update_message` entirely: it arrives naming an `orig_subject` the engine would
no longer have known, `_apply_update` returns early, and a re-opened
conversation goes unseen. That is p9's failure exactly, and it would have been
introduced by the feature meant to tidy p9's evidence away.

## 2. The screenshots, and the defect one of them found

Five shots, `.local/shots/ex1/`.

| shot | what it shows |
|---|---|
| `01-stalled-no-chip` | `2 stalled of 3 open rows`. No confirm chip, no card buttons — there is nothing done to confirm, so the affordance is not there. |
| `02-done-with-buttons` | `1 row open · 2 done to confirm`, the `✓ confirm 2 done` chip, and a `✓ confirm` button on each done card. **The defect.** |
| `03-done-buttons-fixed` | The same, six pixels up. |
| `04-after-row-click` | After clicking one card's button: `1 row open`, chip gone, headline ends `· 2 confirmed rows hidden`. |
| `05/06-chip-before/after` | The same round trip driven from the chip instead. |

**The defect: the button sat on the card's own border.** `PanelGridScene` drew
action buttons at a fixed `top + 64`, which is the bottom strip of the 84-pixel
tasks card. The operation room card is 148 and its evidence runs to five lines
from `top + 52`, so step 2 moved the buttons to `top + height - 20` — the same
pixel on the short card, the bottom edge on the tall one. In the screenshot the
button's green background is flush against the card's stroke and the rounded
corner clips its right end. It is small and it is exactly the class of thing
`agent_room` step 5 and `operation_room` p2 step 4 each found only by looking:
correct data, correct payload, clean build, and a card that reads as broken.
Now `top + (tight ? 64 : height - 26)`.

**`agent_room` step 5's open hole is closed.** It left "action-button cards are
not screenshot-verified" behind; they are now, and the verification was worth
more than the confirmation — it found the defect above, and it answered a
question the code could not: an action button lives inside the card's own
hit area, and Phaser's `topOnly` input means the button takes the click and the
card's evidence popup does not open behind it. Shot `04` is that answer: the
row vanished and no popup appeared.

## 3. One behaviour worth stating plainly

**A row button confirms the conversation, not the card it is on.** The confirm
key is `(channel, bare topic)`, which is what the plan proposed. A resolved
topic makes *every* party's row `done`, so clicking `✓ confirm` on
`agecho-agautolab1`'s card cleared `agecho-agstudio1`'s card too — both are the
same ✔ , seen twice. Both disappear together, so it is visible rather than
mysterious, and it is the right reading: what is being confirmed is that this
conversation reached done, not that one instance's opinion of it did.

A related wrinkle, found live: after confirming at `#4948`, un-✔ ing the topic
brought the rows back, and re-✔ ing it with no new post hid them again with no
second confirm. That follows from the mark being `(topic, last post id)` and is
right — nothing was said in between, so it is the same row that was already
seen — but it is worth knowing that a resolve/unresolve/resolve round trip with
no speech in it does not ask to be confirmed twice.

## 4. What was left running

- The relay is back on its default 900-second threshold, re-swept (241 calls,
  121 topics, 55 channels), and the board is the single quiet
  `agping-agstudio1` row p2 left behind. **`confirmed` is `{rows: 0, topics: 0}`
  after the restart**, which is constraint 2 visible from outside: the marks
  died with the process, exactly like the rows they were hiding.
- `#ops-testbed`'s `agechoplan-ex1-confirm` is resolved. Nothing was posted
  anywhere else, and the observer still never posts at all.
- `.local/ex1probe.py` is the throwaway driver, ignored like the rest of
  `.local/`.

## 5. One thing that could not be verified

The tasks view could not be screenshotted: its Plane backend answers
*"response was not JSON"* and the view renders *"Plane task list unavailable"*.
So the claim that its cards are pixel-unchanged rests on the arithmetic — the
`tight` branch is taken for any card under 124px, and for 84 it reproduces the
old constants exactly, `top + 64` included — and not on a picture. It is a
pre-existing outage of that view's data source, untouched by this work, and
worth naming rather than leaving as an unexamined assumption.
