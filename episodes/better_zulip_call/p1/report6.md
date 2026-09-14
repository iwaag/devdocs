# better_zulip_call p1 — step 6: the Observer off the index, and one budget per credential

Date: 2026-09-14. pyagag "agag.zulip.Budget: one pause per credential…";
pj-agdev "agobserver: the schedule, locations and cancellations read off
the listener's mirror".

## The Observer

**One mirror, two readers.** `agobserver.listener.main` opens the mirror
first and hands it to both the worker and `listener_main` (which gained a
`mirror=` seam for exactly this), so the process holds one copy of the realm
on its credential rather than two.

**A tick costs nothing.** With a mirror the worker's `reconcile()` reads
every open topic of its channel and each one's whole history off the index
(`anchor.read_watch(..., history=…)`), so it runs every tick instead of every
tenth and makes no call. `locate()` answers from the index: an anchor that is
in an open topic is `OPEN` at no cost. Zulip is asked in exactly two cases —
when the index's answer would be **terminal** (the anchor sits in a ✔ topic,
or is not there at all: the mirror may lag an event, and stopping a watch on
a lagging copy is the one mistake this must not make) and **right before a
notification** (`stand_down(verify=True)`), where the location must be
right. The periodic look at the watched resource (the local model's
evaluation) is untouched: it is a different concern from re-reading the
request.

**Lost scheduling state is rebuilt from the index**, as the old code rebuilt
it from the channel — but now without a call.

| fixture (`agobserver/tests/test_mirror_schedule.py`) | what it proves |
|---|---|
| an unchanged watch | three ticks, three looks, **one** Zulip call in all (`whoami`) |
| a ✔ before the next look | cancelled without a look, after one confirming read of the anchor |
| a ✔ during the look, before the notification | the notification is not sent; the watch is cancelled |
| a notification | preceded by exactly one verifying read of the anchor |
| a rename | followed off the index; the record carries the new name; no call |
| a lost store | rebuilt from the index; the watch keeps its schedule; no call |

Step 1 measured an idle tick at one call per watch plus three every tenth
tick; it is now zero, and the finish (route, read-back, post, notes,
resolve) is unchanged at about ten, once per watch.

## The transport boundary

**`agag.zulip.Budget`** — one per credential identity (the email), shared by
every `ZulipClient` on it in the process, and through a sidecar file beside
the credentials (`<env>.ratelimit`) by every process built from that file:

- **honours the server's guidance once.** A 429 answered to any client
  pauses the credential for the `Retry-After` the server named (jittered a
  little); every client's next call waits it out before it is sent. The old
  shape had each client discover the refusal separately and back off on its
  own schedule — the relay's mirror poller, its reader and the Developer's
  chat client were three such schedules on two credentials.
- **joins identical GETs in flight.** Two readers asking the same page at
  the same moment cost one call (`Budget.joined` counts them). Writes and
  long polls are never joined.
- **spaces the release after a pause** — calls that queued up during the
  pause leave 50 ms apart for the next two seconds, so the pause's end is
  not the wave that earns the next 429.
- **is measured.** The client's ledger keys every call by purpose
  (`poll`, `resync`, `register`, `hydrate`, `verify`, `read`, `write`) and
  endpoint; the budget counts `waits`, `refusals` and `joined`; the mirror's
  `health()` carries both. Retries are the waits.

| fixture (`pyagag/tests/test_budget.py`, `test_mirror.py`) | what it proves |
|---|---|
| a 429 on one client | a second client on the same credential waits it out; a client on another credential does not |
| identical GETs | three concurrent readers of one page make one call; POSTs are never joined |
| the sidecar | a pause another process wrote is honoured; our own 429 is written for them |
| release after a pause | four callers leave spaced, not at once |
| a 429 on the mirror's poll | the mirror keeps its copy and answers, says `stale` and why, waits, and is `live` again with no resync |

The relay's views were already built (step 3) to show the last good copy
under a `stale` health block rather than a board of channel errors; the
mirror fixture is what pins that a 429 is one of the ways that happens.

## Reader credentials, stated plainly

The mirror runs on each consumer's own credential, and the relay's on the
`Opsroom Observer` bot: that is **contention isolation** — a burst of
Developer writes cannot starve the relay's reads, and one listener's serving
cannot starve another's — not a reduction in calls. The reduction came from
steps 2–5: reads that are not made.

## Limitations, stated

- **Tools outside `agag.zulip` are outside the budget.** The Developer's
  browser session and any `curl` share the Developer's quota and know
  nothing of the pause; they can still earn a 429 that this code then
  honours. The sidecar is best effort, not a lock: two processes can each
  receive a 429 in the same second and each write the file.
- The Observer's delivery path — resolving the destination from its anchored
  id, the read-back after an ambiguous send, the post, the finish notes —
  keeps its targeted Zulip reads by design; a notification is the one moment
  a lagging copy must not decide.
- Joining applies to GETs with identical URLs on one credential inside one
  process; a page asked through two credentials is two calls, as it should
  be.
