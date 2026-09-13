# observer p1 — step 3: the one post, delivered once

## The problem is not sending

A watch is met once and the requester must be told once — across restarts,
retries, and a send whose answer never came back. Three things carry that,
and they are relied on in this order:

1. **The store's `delivered` record is the fact.** A watch carrying one is
   never delivered again, and the worker stops scheduling it, so ordinary
   repeated polling cannot reach delivery at all.
2. **The watch's own id is in the post.** After an ambiguous send the
   destination is read back for a message of ours naming `` `w6676` ``, and a
   delivery that did land is recognized instead of repeated. The id is a
   message id, so nothing else in the realm produces that string by accident.
3. **The Zulip state note is written after delivery**, so a crash between
   them leaves the watch *owed* rather than silently finished. Owed is the
   safe side, because the read-back turns the retry into a no-op.

**The remaining crash window, named rather than engineered away:** a crash
after Zulip accepted the post but before either the read-back could see it or
the store was written. The next attempt reads the destination back, finds the
post and records it — so the window costs a duplicate only if the post is
*also* unreadable at that moment, which is an outage rather than a race. A
general exactly-once messaging system was not built and is not wanted.

## Routing

The destination is resolved **from its anchor at send time**, never from the
name it carried when the watch was accepted. A message-link destination is
therefore immune to a rename in between: the notification follows the
conversation, and a reused topic name gets nothing. Proven in
`test_the_destination_follows_a_rename`, where the destination is renamed and
an unrelated conversation takes the freed name — the notification goes to the
renamed one and the impostor gets nothing.

**Observer never opens a conversation to deliver into.** A destination that is
gone, or that has been closed with a ✔, is a terminal `undeliverable` outcome
recorded in the *watch* topic; nobody is told, and the watch topic is left
**open**, because that is the one outcome a human has to see and decide about.
A delivered watch, by contrast, is resolved.

The notification names nobody. The post itself is the requester's turn, and
Zulip's own topic route is what serves them — which is the whole of "route it
so the requesting agent is served through its normal listener".

## Live proof

`w6676` — the file watch accepted in step 1 — was completed by renaming
`release.zip.part` to `release.zip`.

- Look 4 (04:02:21Z): `not_met`, "`test -f` returned RELEASE_ZIP_MISSING".
- Look 5 (04:03:21Z, 15.8 s): **`met`** — "`ls -la` shows release.zip and no
  release.zip.part; `test -f` returned RELEASE_ZIP_EXISTS".
- Delivered the same second as message **6697** into
  `ops-testbed/observer-p1-dest`, resolved from the `/near/6673` link the
  requester pasted.
- The watch topic got `[selfnote][state] met` and one visible line — *"Done —
  `w6676`. I told ops-testbed/observer-p1-dest (message 6697) and I am no
  longer watching this."* — and was then resolved (`✔ watch-release-zip`).
- **No further evaluation**: the store went to `state: met`,
  `pending_notification: false`, and the schedule is empty. Nothing has been
  logged for `w6676` since.
- **A normal restart did not replay it.** The listener was restarted after
  delivery; the worker started, found nothing to do, and the destination
  still holds exactly one notification.

## Controlled failures

The failure cases are real and rare, and reproducing them against the realm
would mean breaking something other agents use. They are made instead, with a
Zulip stand-in that speaks only the calls this agent makes — and every one of
those is a real method name on `agag.zulip.ZulipClient`, so a fake cannot let
a test pass over code that could not run. `tests/test_notify.py`, 7 passing:

| case | what is asserted |
|---|---|
| a met watch is delivered | one notification, `met`, `pending` cleared, watch topic resolved |
| repeated polling | a second `deliver` over the stored record posts **nothing** |
| the send fails | returns *retry*, stays `pending_notification`, records the reason, posts nothing — and the next attempt delivers |
| the send is **ambiguous** (accepted, then the connection drops) | the post really landed; the retry reads it back and does **not** repeat it |
| the destination was deleted | terminal `undeliverable`, the watch topic says so, and no conversation is created |
| the destination is ✔ | terminal `undeliverable`, the closed conversation is **not** reopened |
| the destination was renamed and its name reused | the notification follows the id; the impostor gets nothing |

## A correction to report 2

Report 2 said the interval was measured from the end of the work. The code
holds a fixed **cadence** — the wait is what is left of the interval, with a
one-second floor so an evaluation that overran the whole interval cannot
produce back-to-back ticks. The comment and the report now say that.

## Left for step 4

Tests for lifecycle, restart recovery and cancellation (delivery is covered);
the launchd service; the natural-language conversation watch and a request
made by an in-system agent, including whether the acknowledgement wakes it;
and the documentation.
