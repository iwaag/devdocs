# better_zulip_call p1 — step 2: the event-updated read model

Date: 2026-09-14. pyagag `agag.mirror` (commit on `main`, see the pyagag
log for "agag.mirror: an event-updated, recoverable local copy").

## What was built

`agag.mirror` — a package of two modules and one facade:

- **`store.py`** — SQLite, one file per consumer (`<store_dir>/mirror.sqlite`),
  WAL, one connection under one lock. Tables: `messages` by id with the
  channel and topic each is in **now**; `topics` from the channel listing
  (name, `max_id`); `coverage` per topic (`complete`, oldest and newest id
  held); `notes` — every `[selfnote][<tag>] <value>` line, generic;
  `channels` (archived included); `changes` — an append-only feed with an
  autoincrement revision; `meta` — the queue checkpoint and the account the
  store was built for.
- **`Mirror`** (`__init__.py`) — the ingest thread and the query door. A
  `Transport` protocol names the seven client methods it needs;
  `ZulipClient` satisfies it and the tests' `FakeRealm` does too.

The transport (`agag.zulip.ZulipClient`) gained what the mirror needs and
nothing else: `messages_page` (a raw page with `found_oldest`),
`channel_topics_detail` (names **and** `max_id`), a general `register`
(event types, `all_public_streams`, `fetch_event_types`), `poll(dont_block)`,
a per-purpose call `ledger`, and `rate_limit_reset` read as a moment. The
613 existing pyagag tests still pass.

## How it behaves

**Bootstrap.** `whoami` → check the store was built for this account (a
different account's copy is deleted, not reused) → if a queue id is
checkpointed, one non-blocking poll asks whether it is still alive; if it
is, the queued events are applied and the mirror is live with **no history
read at all**. Otherwise: `register` (all public streams, events `message`,
`update_message`, `delete_message`, `stream`, empty initial state) → deep
resync → checkpoint → poll loop.

**The deep resync** is the only history read the ingest thread makes, and
it is the recovery after a queue expiry as well as the first fill. One
`streams` (archived included), then per live channel one listing and the
channel's whole history in pages of 1000 (`narrow=channel`, walking
`found_oldest`). Every message read is written with `replace`, so an edit
that happened while nobody listened is corrected; every held message the
realm returned nowhere is marked deleted; every topic seen is marked
complete. Archived channels are not read and keep whatever copy they had.

**Events** apply in one transaction per poll together with the checkpoint,
so a crash cannot leave applied events uncheckpointed or the reverse.
Replay is safe by construction: a `message` event never overwrites an id
already held (a hydrated copy may be newer than the event's snapshot); an
`update_message` content edit with an `edit_timestamp` older than the one
held is dropped; a move to where a message already is does nothing; a
delete of a deleted message does nothing. A topic move (`orig_subject` →
`subject`, all `message_ids`; a resolve is one with a `✔ ` name) updates the
rows and carries the coverage across when the whole topic moved. `stream`
create / update / delete keep the channel table current; a channel that
vanishes from the listing is marked archived rather than kept live.

**Hydration on demand.** `hydrate(channel, topic)` reads one topic whole
under exactly the name given, single-flight: concurrent callers for the
same topic wait for the one read. `messages(..., hydrate=True)` hydrates
only a topic whose coverage is not complete. `verify(message_id)` is the one
targeted read for a moment when event lag is not an acceptable reason to be
wrong (before a notification, before a resolve): Zulip's answer is folded
into the store, and "gone" is folded in as a deletion.

**Freshness and completeness.** `health()` says `live` or `stale` with the
reason and `stale_since`, the queue and last event id, resync count and
cost, the revision, the store's counts (messages, topics, complete /
partial coverage, notes) and the call ledger by purpose. A stale mirror
keeps answering from its last state; the caller decides what to trust.

**The change feed.** `changes(since)` returns what moved after a revision
(`message`, `edit`, `move`, `delete`, `channel`, `resync`), or `None` when
`since` is older than the feed remembers — the signal to re-read the index
instead. `wait(since, timeout)` blocks until the revision moves. This is
what step 5's listener feeds its pending queue from.

**Names versus ids.** The index keys topics by live name and answers a bare
name with every topic that carries it: `topic("pj-x", "workplan-a")` returns
the ✔ one, the open one, or both when a post after a resolve opened a twin.
`live_name` prefers the open one. A message is found by id wherever it is
now; an anchor never follows a name.

## Verification

### The fixtures (`pyagag/tests/test_mirror.py`, 10 tests, 2.6 s)

Against a `FakeRealm` that speaks the transport protocol and whose every
mutation has a `quiet` form (a change with no event — what happens while a
queue is expired):

| fixture | what it proves |
|---|---|
| bootstrap overlap | `register` precedes the first history read; a message that lands mid-hydration is both in the page and on the queue and ends up **once**, with one change row |
| event replay | the same events delivered twice neither duplicate a message nor regress an edit; only the genuinely new event makes a change |
| edit and delete | an edit re-indexes the notes; an older replayed edit is dropped; a delete hides the message, drops its notes, and removes an emptied topic from the index |
| move, rename, reused name | a resolve moves the conversation and its coverage under the ✔ name; a post under the freed bare name is a second open topic beside it; the old anchor still answers with its own topic; a plain rename leaves the old name |
| queue expiry | with the queue dead, an edit, a deletion, a post and a new channel with a new topic happen silently; the recovery re-registers, deep-resyncs, and the store reflects all four; the new queue delivers again |
| restart without a scan | a second `Mirror` on the same store resumes the persisted queue: no `streams`, no listing, no history read; a message posted while the process was down arrives through the resumed queue; a different account's store is rebuilt |
| single flight and verify | three concurrent `hydrate` calls make one read; `messages(hydrate=True)` on a complete topic makes none; `verify` folds Zulip's answer in, deletion included |
| index, notes, intros | `last_real` skips selfnotes and Zulip notices; notes are indexed per line; `intros()` parses the roster and marks a ✔ introduction retired; `history()` answers Zulip-shaped dicts; an archive event removes a channel live; `wait()` wakes on a change |

### The live realm (Opsroom Observer credential, scratch store)

| measure | value |
|---|---|
| first fill (deep resync) | **62 calls, 1.0 s**: 1 `streams`, 30 listings, 31 message pages |
| what it holds | 3,253 messages, 241 topics (31 open), 30 channels (132 archived known), 707 notes, every topic complete |
| store size | 2.3 MiB |
| reopen after stop | **live in 0.21 s on 3 calls** (`users/me` and two polls), `resyncs 0`, reason `live (resumed queue)` |

The notes index, for scale: 361 `served`, 136 `rootchat`, 73 `state`, 28
`doc`, 16 `delivered`, 16 `accepted`, 15 `mission`, 15 `watch`, 9 `task`,
8 `result`, 4 `asset`, 4 `assetrun`, 2 `replaces`, 1 `rootchat-moved`.

For comparison, step 1 measured the relay's ops sweep at 120 calls for 50
unresolved topics read 50 messages deep, and a listener's full sweep at
69–84 calls. The deep resync reads **everything** — every topic, resolved
ones included, whole — in half the calls, because it pages by channel rather
than asking per topic.

## Decisions taken inside the step

- **Resync reads the whole realm rather than diffing listings.** The plan
  asked for an explicit resynchronisation of affected coverage after an
  expiry, because recent-message fetching cannot recover missed edits and
  deletions. At this realm's size the honest version (read everything,
  diff) costs 62 calls, which is less than any partial scheme's worst case
  and needs no reasoning about which topics *might* have changed. If the
  realm grows to where this matters, `topics.max_id` from the listing is
  already stored as the fingerprint a lighter pass would compare.
- **Archived channels are known but not read.** Their history is readable
  by an administrator and not, in general, by a bot; the store keeps what
  it held before the archive and `channels(include_archived=True)` says
  they exist, which is what the completion walk needs (`closing._channels`
  distinguishes "archived" from "could not be read").
- **The `mentioned` flag is carried on the change row**, not stored on the
  message: it is per-user, and the store is one credential's copy anyway.
  A consumer's own queue is where the flag is authoritative; content
  (`@**name**`) is how the shared model finds mentions of anybody.
- **No second work ledger.** Missions, tasks, watches and runs are read
  out of the notes index by the consumers that know their vocabulary; the
  mirror indexes the shape (`tag`, `value`) and nothing about meaning.

## Limitations, stated

- Events are per-credential and public-only. A private channel would need
  the credential to be in it; none exists on this realm today.
- The persisted queue resumes only within Zulip's queue lifetime (ten
  minutes of no polling by default). A longer outage is a deep resync.
- A message deleted in an archived channel stays in the store as it was.
- Message reactions, submessages and edit history are not kept; nothing in
  this realm reads them.
