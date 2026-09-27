# failsafe p5 — step 2: queue evidence shared with Observer

Date: 2026-09-27 (UTC). Deployed 12:15Z (Observer and relay restarted;
autolab's health command and Front's `agentchat` run from their synced venvs).

## What a queued post is now

One reading, `agag.waits` (pyagag), used by the progress panel and by
Observer. A **wait** is one queued unit: the post, its original time
(`queued_at`, kept whatever moves ahead of it), what it waits behind
(`ahead`), the evidence source (`confirmed` or `conversation`), when it was
observed, and a state:

| state | meaning | how it is established |
|---|---|---|
| `behind` | the owner is busy with healthy work elsewhere: a legitimate wait | confirmed: the owner's own listener journal plus that serving's health check; or conversation: the owner has an open serving elsewhere with work in the last 30 min |
| `blocked` | the serving ahead is not confirmed healthy (stopped, or a check that establishes nothing) | confirmed |
| `unserved` | the listener is not serving it: idle for 90 s while it waits, passed over by 3 servings begun after it, or the listener gave up (`failed`) | confirmed |
| `unknown` | nothing establishes why it waits | no check, a check older than 120 s, a failed probe, nothing open anywhere, or a conversation-only excuse older than 30 min |

Only `behind` excuses a wait, and only on current evidence. An open
conversation alone is not process health: the conversation-only excuse
lapses 30 min after the post.

## Where the confirmed facts come from

`agag.health.probe_queue` (`python -m agag.health … --queued --since
<post time>`) reads the owner's `listener.sqlite` read-only: the
conversation's `pending` entry (`enqueued_at`, position, state), what the
executor is running now, and how many servings began after the entry was
queued. For the serving ahead, it runs the ordinary health probe on that
serving's own ack and execution record. Verdicts: `queued` (behind healthy
work), `stopped` (idle, passed over or failed), the picked-up serving's
own verdict, or `unknown` with the serving ahead named. Bounded like the
other probe: one read-only query, the process table, no Zulip call.

## Observer

Before `unacknowledged` becomes an incident, `Monitor.review_queues` reads
the wait. It uses a probe run with the look's other probes for autolab
(`prefetch`), or the open servings of every traced request for an owner
without the interface. Then:

- `behind` → nothing is opened or asked. The post is looked at again every
  look, and the history is kept in the health state (`q<anchor>`: post,
  original queued time, first seen, each change of what is ahead). The
  health record counts waits per state (`queues`).
- `blocked`, with the serving ahead held by another traced request → left
  to that request. Its own health path asks about the stopped work itself,
  in the request that owns it. This lasts at most `escalate_after`
  (600 s); after that, or when no traced request holds the blocker, the
  post is `unserved`.
- `unserved` → a new incident kind, **reported to the owners at once**.
  Nobody in the request can make a listener serve a post, and asking Front
  would only buy another post into the same queue.
- `unknown` → the plain `unacknowledged` rule, as before, with the reason
  the wait could not be established added to the fact. A failed or stale
  check therefore never excuses a queue.
- An incident opened on a wait that a later look confirms as `behind` is
  closed **`excused`** ("a legitimate wait, not a stall"). It is never
  `rescued`, so the rescue #13738 claimed in trial B cannot recur, and no
  developer review is opened for it.

Detection bounds: `unserved` fires on the first look after the
`unacknowledged` grace (300 s) once the listener has been idle for 90 s.
A blocker that ends and leaves the post unpicked is therefore seen 90 s
plus one look later. A blocked wait is reported after at most 10 min. A
healthy queue is never asked about, however long the work ahead runs,
because every look checks it again.

## Related fixes found on the way

- **`agentchat recheck` of a queued post said STOPPED** (#13745): pending
  posts were counted only *after* the `--after` id, and Observer names the
  unacknowledged post itself as `--after`. The post itself now counts, so
  the verdict is ASKED ("do not ask again"). This is what drove Front's
  false resume #13732.
- **A first request's owner was empty**: a new topic has no ack yet, so the
  trace knew no owner, and neither the panel nor Observer could say whose
  queue the post was in. This is the common case: trial A's #13327 was a
  plan's first post. The owner of an unacknowledged post in an unowned
  conversation is now the agent it names.
- **The panel**: `progress.queue_behind` now uses the same reading. The relay
  probes queued posts of probed owners (`--queued`, cached like the other
  probes). A card keeps the confirmed answer when there is one, and the
  conversation-only one otherwise, and says which it is (`unit.queue`).

## Verification

- pyagag `7725a52`: 1034 tests, including `test_failsafe_p5.py`
  (conversation and confirmed readings, freshness, the panel reason,
  recheck ASKED) and 7 `probe_queue` tests over a real listener queue
  file: healthy blocker, stopped blocker, idle executor, passed over,
  picked up, never taken in, and the command line.
- agobserver `test_p5_queue.py` (6): healthy queue for 28 min → no request,
  no incident, history kept; idle listener → `unserved` reported, Front not
  asked; failed check → plain rule; blocker stopped in another request →
  that request asked, the queued one not, reported after the bound;
  `unknown` then `queued` → `excused`; owner without the interface →
  conversation excuse, lapsing after 30 min of no work ahead. Full suite
  175.
- relay: 363 (a queued post shows the listener's own answer, probed once,
  `--ack 0 --queued`). agautolab 331, agfront 186 on the new pin.
- Live: Observer's cycles run clean after the restart (`queues` in its
  health record). `python -m agag.health --queued` against autolab's live
  queue answers from the real file. The concurrent trials of step 6 are
  the live test of a real queue.

## Not done here

- Owners other than autolab still have only conversation evidence.
  Front's listener is serial too, but it exposes no health interface, as
  before.
- `unserved` asks a person to look at a listener. Nothing restarts one
  automatically.

## Commits

pyagag `7725a52`; pins in agautolab, agfront, agdevworld (relay: queued
probes and the shared reading) and agobserver (`review_queues`,
`unserved`, `excused`, `probe_queue`) via pj-agdev.
