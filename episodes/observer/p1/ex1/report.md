# observer p1 ex1 — report

## What was wrong, in one sentence each

p1 adopted the realm's hard-won rule — *a name is reusable, an id is not* —
for the watch **record**, and then went on reading names everywhere the
record was used.

1. **A destination accepted as `<channel>/<topic>` followed the name.** The
   requester's text was stored verbatim and re-resolved at send time, which
   asks Zulip *which conversation is called that today*. Rename the
   destination, let somebody take the freed name, and the notification goes
   to strangers.
2. **Cancellation compared cached topic names.** Renaming a watch topic
   before resolving it made the ✔ invisible: the check looked for
   `✔ <old name>` and what existed was `✔ <new name>`. The watch went on
   evaluating, and addressed a name it no longer held. `reconcile()` made it
   permanent by skipping any record whose state already matched.
3. **An unreadable destination was treated as a deleted one.** A timeout
   became a terminal `undeliverable`: the watch ended, and nobody was told —
   which is the one outcome the whole agent exists to avoid. The read-back
   that guards against an ambiguous send had the same flaw in the other
   direction: a failed read returned "no delivery found", and the only thing
   that produces is a second notification.

## What the contract is now

**A destination is an id before it is accepted.** Both spellings — a message
link and `<channel>/<topic>` — are resolved at intake to a message in the
conversation the requester meant, and that id goes into the Zulip accepted
note, so it survives a lost local store. The written name is kept for saying
what the requester wrote, never for looking anything up.

**A watch is found by its own anchor id, twice per tick.** One lookup answers
cancellation, renaming and removal together: `closed` cancels, `absent` ends
the watch locally and quietly, `open` gives the conversation's *current* name
and everything the watch says is addressed there, and `failed` concludes
nothing.

**Only Zulip's own "no" is terminal.** A destination that cannot be read, a
read-back that cannot be performed and a send that fails all keep the met
result and the pending notification, record the reason and retry at the
ordinary interval. The condition is never re-judged: it was met, and a
delivery problem is not evidence about the world. One watch stuck this way
does not hold up any other.

**The shared client learned the distinction** (pyagag `f613feb`).
`ZulipClient.call()` raises `ZulipRejected` for an answered 4xx and a plain
`ZulipError` for a 5xx, a timeout or a dropped connection; `message()`,
`conversation_of()` and `topic_history_across_resolve()` take `strict=True`,
under which absence means Zulip said so. The lenient default is unchanged, so
no other agent moved.

## Evidence

**Tests.** `agobserver` 33 → **63**, `pyagag` 584 → **592**, both green. The
reproductions are the three defects as three named tests:
`test_the_notification_follows_a_named_destination_through_a_rename` (and its
twin after the store is rebuilt from Zulip),
`test_a_renamed_then_resolved_watch_is_cancelled_without_a_look`, and
`test_a_destination_that_cannot_be_read_is_retried_not_abandoned`. Failure is
injected into the stand-in client, which mirrors the real client's error
policy as well as its method names.

**Live, on this host, 2026-09-13.**

- `w6745` — accepted against the **name** `ops-testbed/observer-ex1-dest`,
  which became `destination_id: 6742` in the Zulip record. The conversation
  was then renamed and its old name taken over by an unrelated post. The
  notification landed in `observer-ex1-dest-renamed` as message 6750; the
  conversation holding the reused name received nothing.
- `w6756` — renamed to `watch-ex1-cancel-renamed`, kept evaluating under the
  new name (two `not_met` looks), and cancelled at 04:56:23 when that topic
  was resolved. Nothing further for two and a half minutes; `evaluations`
  stopped at 2. A separate request posted under the freed old name became its
  own intake and never touched `w6756`.

Both would have failed on p1's code, in opposite directions: the first would
have notified the impostor, the second would have kept polling forever.

Deployed by `launchctl kickstart -k` at 04:51:22 with nothing in flight (all
four p1 watches were `met` and owed nothing). `nctl drift` converged=46,
0 errors, before and after. Zero paid tokens: six local runs, `profile
local`, `outcome done`.

## Remaining limitations

- **The anchor for a named destination is that conversation's newest message
  at intake.** Zulip can move a single message between topics, so an anchor
  moved on its own would take the destination with it. Every choice of anchor
  message has this property.
- **Exactly-once is bounded, not guaranteed**, and the module now says so:
  the read-back looks at the newest 40 messages of the destination, so an
  ambiguous send followed by more than that many arrivals before the retry
  would hide the delivery. For a conversation quiet enough to be waited on
  this is not a real window; for a busy one it is.
- **A destination unreachable for a long time retries silently forever.**
  There is no delivery-side streak report, no attempt cap and no backoff —
  one Zulip call per watch per interval. `delivery_attempts` and
  `delivery_error` are in the store for whoever goes looking.
- **Locating costs one message read per watch per tick**, where the old check
  was one channel listing shared by all of them. Cheaper per call and it does
  not grow with the channel, but it is per-watch.
- **A `removed` watch leaves its store record and its topic workspaces**, and
  nothing prunes either — p1's growth caveat is untouched.
- **Transient-error behaviour is proved in tests, not live**, deliberately:
  the alternative is breaking a Zulip other agents depend on.
- **No migration.** A watch accepted before this change carries a name and no
  id, and still resolves by name — which is the behaviour ex1 exists to stop
  relying on. There are none left on this host; every p1 watch is finished.
