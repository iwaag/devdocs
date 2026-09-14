# better_zulip_call p1 — step 4: completion on indexed relationships

Date: 2026-09-14. agdevworld commit "agentroom: the completion remembers its
plan, revalidates the scoped channels, and answers from its results";
pyagag "agag.mirror: refresh_listing" and "per-client facets".

## What changed

**A preview is walked once and remembered.** `Closer.plan()` runs the
discovery over the mirror (`MirrorRealm`, step 3) and keeps a `Plan`: the
root, the fingerprint, the mirror revision, the discovery, the actions, and
two kinds of dependency — `evidence` (live name, last post id, resolved) for
every conversation reached or excluded, and `listings` (the topic names the
mirror held) for every channel the walk consulted: the root's, every reached
conversation's, each mission's dedicated `work-` channel, and every channel
the archivable check looked at. The payload says so under `depends_on`.

**A close reuses the plan when nothing it rested on moved.** `_unchanged()`
compares the remembered evidence and listings against the mirror — local,
no Zulip call — and the walk is not repeated. When the mirror's revision has
not moved at all the comparison is skipped.

**Before anything is written, the scoped channels are read from Zulip.**
The fingerprint is the human's decision token and nothing else: an
unchanged local revision proves nothing about the realm, so `_revalidate()`
calls `Mirror.refresh_listing(channel)` once per scoped channel — one
`GET /users/me/<stream>/topics` each, names and `max_id` compared with the
store; a name whose newest id moved, a name that appeared and a name that
vanished are hydrated, and the store catches up on exactly what changed. If
anything changed, the plan is walked again (from the copy, which now holds
the change) and a fingerprint that no longer matches the approval is a
refusal with the fresh plan. A child added after the preview, with no event
delivered, is found this way — its topic is new in the listing — and holds
its mission `blocked` and its own topic `kept`.

**The post-write answer is built from the results.** `_after()` turns each
applied action `done` (saying whether the mirror has confirmed it), each
failed one `ready` again with the error as its reason, and leaves the rest
as they were; its fingerprint is what a retry approves. There is no third
walk. The remembered plan is spent by the close, so a retry reads the copy —
which now carries the acceptance notes, the ✔ names and the archive — and
finds the accepted record `already`, the resolved topics `already`, and
only the failed target left to do.

**What a completion costs on Zulip is reported honestly.** `gaps.reads` is
what the walk asked the realm interface (the mirror); `gaps.zulip_calls` is
what the mirror actually spent for those questions (hydrations, verifies),
read off its ledger by purpose so the ingest thread's own polling never
counts; `revalidated.zulip_calls` is the pre-write check; `confirmed` on
each applied result is the mirror's word.

## Verification

### Fixtures (`tests/test_close_plan.py`, five tests over a mirrored chain)

A Front Desk request delegated to autolab — a mission in `pj-x` with one
finished task in its dedicated `work-m<id>` channel — on a `FakeRealm`
whose mirror runs its ingest thread, so the writer's changes come back as
events, and every quiet mutation is exactly the event lag the plan names.

| fixture | what it proves |
|---|---|
| preview/apply reuse | one walk (`discoveries == 1`) for preview, second preview and close; the close makes **3** Zulip calls — one listing per scoped channel (`front`, `pj-x`, `work-m…`) — and nothing else; every applied result `confirmed`; the acceptance is written once; a preview afterwards reads everything `done` off the copy at zero calls |
| a child added after the preview | a second task posted with no event: the pre-write listing finds it, hydrates it, the plan is re-walked, the approval is refused, **nothing is written**; the mission is `blocked` and the new task's topic `kept` |
| a changed work state | a `[state] open` note posted quietly into the finished task: found by the listing, mission `blocked`, approval refused, nothing written |
| partial failure and retry | the archive fails: `channel` `failed`, the conversation `skipped` and kept open, `partial: true`, the channel `ready` again with the reason; the retry approves the post-write fingerprint, finds the record and the topics `already`, archives the channel, resolves the conversation, and the acceptance was written once |
| growth elsewhere | a new channel with five new topics elsewhere: the close still reads exactly the three scoped listings and walks nothing again |

### Live, on the side port (the mirror's store persisted from step 3)

| preview | actions | first call | second call | depends on |
|---|---:|---|---|---|
| `front-desk-20260908-164810` | 5, all done — **fingerprint identical to the old relay's** | 0.09 s, 6 reads, 0 Zulip calls | 1 ms, remembered | 3 channels, 4 topics |
| `front-routine-mediagen` | 50 (40 done, 10 kept in archived channels) | 0.01 s, 17 reads, 0 Zulip calls | 1 ms, remembered | 13 channels, 40 topics |

The larger request's 50 actions (step 3 saw 23 done / 27 kept before the
archived work channels were hydrated) are stable across restarts now: the
hydrated copies persist in the store, so what was `kept — archived` because
its history was unknown reads `done — already ✔` once it is known, and only
the ten genuinely unresolved topics in archived channels stay `kept`.

A live close — writing to the realm — is part of step 7's demonstration.

## Costs, before and after (per close, Developer credential)

| phase | step 1 baseline | now |
|---|---:|---:|
| preview | 12–59 reads, all Zulip | 0 (mirror), ≤ a few hydrations once |
| apply: re-derive | a second walk (12–59) | 0 (remembered) |
| apply: pre-write check | none | 1 listing per scoped channel (3–13) |
| apply: writes | the writes | the writes |
| apply: post-write answer | a third walk (12–59) | 0 (from results + confirmation) |
| the frontend's reload of `/agents` + `/work` | 36 | 0 |

Roughly 200 Developer calls per close became the writes plus a handful of
listings on the mirror's credential.

## Limitations, stated

- The pre-write check reads listings, so it sees new posts, new topics,
  moves and resolves in the scoped channels; an edit or a deletion of an
  older message in an unchanged topic is not seen before the write. Full
  isolation from concurrent Zulip users is not claimed.
- A conversation reached only through an archived channel is hydrated once
  and never re-read: nobody can post there, so nothing changes.
- The remembered plan lives in the relay's memory; a restart re-walks from
  the copy, which costs no Zulip call.
