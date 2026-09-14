# better_zulip_call p1 — phase report

Date: 2026-09-14. Seven steps, one day, deployed on this host. The per-step
reports (`report1.md` … `report7.md`) carry the evidence; this is the shape
of what changed and what it now costs.

## The problem, in one line

Every reader of the realm reconstructed the same public conversations from
Zulip each time it was asked and kept nothing: the agent room and the
completion walk on the Developer's 200-a-minute quota, every listener's
startup sweep, the Observer's per-tick lookup, and a notifier polling one
narrow 700 times an hour. Nine closes in twelve minutes were 1,800 Developer
calls and 250 refusals, and the board went blank for half a minute at a time.

## What exists now

**`agag.mirror`** (pyagag) — an event-updated, persisted copy of the realm's
public conversations, one per process on that process's own credential. A
queue registered for all public channels before the history is read; a
paged read of the whole realm to fill (62 calls, a second, 3,253 messages
in 30 channels); events from then on, applied idempotently with the
checkpoint in the same transaction; a deep resync after a queue expiry that
diffs the realm against the store; hydration on demand, single-flight, for
what the copy does not hold (archived channels' topics, once); a change
feed with a revision; `health()` saying live or stale, since when, and what
was spent on what. Messages are keyed by id with the topic each is in now,
which is what lets every consumer follow a conversation through a rename, a
resolve or a reused display name.

**The relay** (`agdevworld/agentroom`) reads with one credential — the
mirror's, `Opsroom Observer` — and writes with the Developer's, and does
nothing else with it. `/agents`, `/work`, `/ops`, `/routines`, `/frontdesk`
and every completion preview are queries of the copy; there is no
per-request read, no 30-second cache, no `forget()` after a write, no
subscription writes. A completion remembers its plan with what it depends
on, reads one listing per scoped channel before it writes (event lag is
real), refuses a plan that moved, builds its post-write answer from its
results and the mirror's confirmation, and reports what it actually spent.
The views show a stale copy as stale, beside the data.

**`agag.listen`** (pyagag) replaces the sweep loop in every listener: an
intake thread follows the mirror's change feed into a durable queue, one
executor evaluates each conversation as it stands now and serves it, and
neither waits for the other. Recovery reads the index. Front's run recovery
is the listener's hook and reads the mirror; cagent's topic listener and the
notifier's command intake are on queues too.

**The Observer** shares the listener's mirror: a tick makes no Zulip call;
a ✔ or a missing anchor is confirmed by one read before it ends a watch;
the location is verified once right before a notification.

**`agag.zulip.Budget`** honours a 429 once per credential — every client on
it, in the process and, through a sidecar beside the credentials file, on
the host — joins identical GETs in flight, and releases the queue spaced
after a pause. The client's ledger keys every call by purpose.

## What it costs, before and after

| scenario | step 1 baseline (2026-09-14 morning) | now (step 7, deployed) |
|---|---:|---:|
| cold board reload (`/agents` + `/work`) | 36 Developer calls | **0** |
| warm board reload | 0 (30 s cache) | **0**, no cache, and any change is in the next read |
| completion preview | 12–59 Developer calls | **0** (a few hydrations once per archived channel) |
| completion apply | two more walks + writes + a 36-call reload | **one listing per scoped channel** (2 in the demonstration) + the writes; no reload |
| nine closes in twelve minutes | ~1,800 calls, 250 refusals | the writes and a handful of listings, on two quotas that no longer compete |
| relay restart | 120 calls (`Opsroom Observer` sweep) | **62** once per store; a restart within the queue's lifetime **reads nothing** |
| listener restart | 69–84 calls (full sweep) and sometimes a paid run | **0** within the queue's lifetime; 62 once per store; recovery from the index |
| an idle Observer tick with a watch active | 1 call per watch, +3 every tenth tick | **0** |
| the notifier's command intake | 696 calls an hour | **0** between commands (an event queue), one narrow read at start |
| one Front serving | 13–25 calls | unchanged (the serving's own reads: threads, harvest, re-check) |
| one autolab serving | ~60 calls | unchanged |

The last two rows are deliberate: this phase separated *listening* and
*reading boards* from *serving*, and a serving's own reads are the next
thing the mirror can answer.

## The demonstration (step 7)

A request to Front in `#front › front-bzc-p1-demo` → Front asked
permission (its standing rule) → granted → Front delegated to the Observer
with `agentchat` → the Observer accepted `w6994` → the Observer was
restarted mid-watch and resumed it from the persisted store and the
resumed queue → a second watch was opened and cancelled by a ✔, confirmed by
one read → the condition was met → the Observer notified Front's
conversation → Front was served **in the same second** and replied → the
relay's preview reached the watch topic (already ✔) at zero calls → the
close read two listings, resolved the conversation, and a second client saw
it gone at once. Every listener on this host filled its mirror in 62 calls
at deployment and resumed its queue on restart.

One defect was met live and fixed the same hour: the notifier's new queue
path read *any* mention as a command, where the old narrow had only ever
returned mentions of the notifier itself; its refusal into Front's
conversation cost one paid Front run. `addressed()` now checks whose name it
is.

## Limitations that stand

- A tool outside `agag.zulip` — the browser, `curl` — shares the Developer's
  quota and knows nothing of the budget's pause.
- The pre-write check reads listings: new posts, new topics, moves and
  resolves in the scoped channels are seen; an edit or a deletion of an
  older message in an unchanged topic is not. Full isolation from concurrent
  Zulip users is not claimed.
- The persisted queue resumes only within Zulip's queue lifetime; a longer
  outage is a deep resync (62 calls here).
- A serving's own reads are unchanged; a run's `agentchat` reads from a
  shell are unchanged too. Both now go through a budget that is shared with
  the listener's mirror on the same credential.
- The Observer's delivery keeps its targeted reads by design.

## Where things are

- pyagag `main` at the "Budget" commit; every consumer locked to it
  (agentroom, agfront, agautolab, agforge, agobserver, comfynotify,
  arxivsage, cagent).
- Stores: `<consumer>/.local/mirror/mirror.sqlite` and `listener.sqlite`;
  `agentroom/.local/mirror/`; `pj-clusterintent/.local/cagent-window/mirror/`.
  All disposable.
- Measuring: `pj-agdev/devenv/zulip/callcount.py` over the Zulip server log.
- The relay's plist no longer carries `AGENTROOM_ZULIP_ENV`; it was
  regenerated and bootstrapped on this host. Everything else deploys with
  the same commands as before.
