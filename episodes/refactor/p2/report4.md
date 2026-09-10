# p2 step 4 — the operation room reads forge where forge now keeps it

## What changed

### `agentroom/forge.py` — the reader

The companion of `autolab.py`, written for the same reason and to the same
discipline: this room is a *reader* of other agents' realms and holds no
dependency on any of their packages, so it reproduces the format
`agforge.anchor` writes rather than importing it. Both reading rules are the
writer's — **identity is written once** (earliest note wins; its own message
id *is* the record) and **state and the current plan change** (newest note
wins) — and only the record's own author's notes are read, with `accepted`
as the one exception, because acceptance is somebody else's word by
definition.

### forge's lifecycle is not autolab's, and is not made to be

The plan's sharpest requirement. autolab's rule is parent/child counting: a
mission is finished when every one of its live tasks is. Applying that to
forge would be wrong in both directions — a request with no run at all would
be "blocked, it has no task", and a request that failed once and succeeded
on the second attempt would be blocked forever by its failed first child.

`forge.reason_not_accepted` asks the question forge's work actually poses:
**has anything been delivered?**

| state | verdict |
|---|---|
| a run is `pending` | blocked — a generation is still running, named |
| `planned` | blocked — nothing has been delivered yet |
| `failed` | blocked — its last attempt failed; run it again or retire it |
| `delivered` | **ready** — its asset was delivered; accepting it |
| `accepted` | done — a successful no-op |
| `retired` | kept — its replacement is the request to finish |

A pending generation being *blocked* is what makes step 1's rule do the work
the plan asked of it here: a blocked record holds back its own conversations
(`close._hold_dependents`), so unfinished generation stays reachable.

### The three things stay distinct on forge's half too

| what happened | who says it | how it is written |
|---|---|---|
| the generation succeeded and the asset reached the requester | the run | `[state] delivered`, in both conversations |
| a person accepted it | this room's button | `[state] accepted` on the **request** |
| the conversation is over | the same button, afterwards | Zulip's `✔ ` |

`accepted` goes on the request and not on its runs: a run is an *attempt*,
and a person accepting an asset is saying something about the request, not
about each try that led to it. It is written under the topic's live name,
because a request's conversation is usually already ✔ by then and a post
under a resolved topic's bare name opens a twin beside it.

**Closing a request never archives a shared channel.** forge keeps no
dedicated channel — its conversations live in `#agforge-agstudio1` — and
`closing._channels` only ever considers `work-<label>` channels bound to a
mission of this request. There is a test for it rather than an argument.

### Plane is gone from the relay

forge was its last consumer, so the whole branch went with it:

- `closing.PlaneBoard`, `PlaneReader`, `_plane_works` — deleted;
- `close.PlaneOps`, `GROUP_IDENTITY`, `_plane_work_action`, `Closer.plane_factory`,
  the `plane.complete` call — deleted;
- `main.PLANE_VARIABLE` / `AGENTROOM_PLANE_ENV` and its `PlaneReader` wiring
  — deleted, and removed from `devenv/launchd/com.agdev.agentroom.plist.in`;
- `status.plane` and `gaps.plane` — out of the payload, and out of
  `completionState.ts` (the "No Plane credential" warning line and the
  `gaps.plane` spread) and `frontDeskState.ts`'s fixture.

**The `[selfnote][work]` note went too** — `parse_work_note`, `WorkNote`,
`work_notes`, `Related.works` and `ops.Topic.works`. It named a Plane issue,
autolab stopped writing it in p1 and forge in p2, and no reader was kept for
the old format: an abstraction preserved only for records nothing writes is
the cost these two phases exist to remove. `ops.Topic` retains forge's notes
instead, the same shape as autolab's and for the same reason (a held topic's
`history` has the selfnotes filtered out of it).

One small fix fell out of the naming change: `closing._stem`, which pairs a
`assetrun-` topic with its `assetplan-` sibling so the walk knows what to
read, now drops a trailing `-a<id>`/`-m<id>` anchor suffix. Without it the
run topic `assetrun-robot-a31` no longer paired with `assetplan-robot` and
dropped out of the walk entirely — caught by the existing chain test.

## Verification

`agdevworld/agentroom`, fixtures only:

- **`tests/test_forge.py`** (20 new tests) — the format and the two reading
  rules, `[]` vs `None` toolsets, results in order without repeats, a
  visitor's `[state]` line not moving somebody else's record, `accepted`
  read from anybody, and every branch of the lifecycle including *a request
  needs no run of its own to be finished* and *one failed run beside a
  delivered request does not block it*.
- **`tests/test_closing.py`** — the chain fixture now carries forge's real
  record (request `a31`, run topic `assetrun-robot-a31`, object key
  recorded). New: the request and its run read out of their conversations, a
  run attributed by the **request id its note names** (a run naming another
  request is orphaned, not adopted), a pending generation as the request's
  state rather than a missing child, and forge contributing no channel to
  archive.
- **`tests/test_close.py`** — a delivered request ready; a failed one
  blocked; a pending one blocked with **its conversations left open while
  autolab's independent branch still closes**; a retired one kept; the
  acceptance written on the request and not on its runs; the second click a
  no-op.
- **`tests/test_no_plane.py`** — the relay loads no `agag.plane`, in-process
  and in a fresh interpreter, and `Closer` has no `plane` field and no
  `plane` key in `status()`.

**Acceptance is exercised with the human-writer identity**, as the plan
asks: the fixture realm posts as the Developer (id 8), not as forge, and
`test_the_forge_acceptance_is_visible_to_its_own_next_preview_too` asserts
the sender id and that the next preview reads the request as accepted. That
is the seam `refactor` p1 step 4 found live, checked on forge's half before
it could be found live again.

```
$ uv run pytest -q        # agdevworld/agentroom
267 passed in 8.8s

$ npx tsc --noEmit && npm run build      # agdevworld
clean; built
```

`agentroom`'s pinned `pyagag` was moved from `0e38c25` to `218cb31` — the
older pin still re-exported `PlaneConfig` from `agag/__init__`, so importing
`agag.selfnote` loaded the Plane HTTP client and the import-graph assertion
could not have been true whatever this relay's own code did.

## Limitations left standing

- **Old forge work is unreadable, deliberately.** A `[work]` note is not
  parsed by anything any more, so a pre-p2 forge conversation shows its
  topics and no work record. Nothing was migrated, as the plan permits.
- **The `tasks` view still dispatches from Plane** (`planeState.ts`, the
  autolab gateway route). Untouched, as the plan reserves it for a later
  phase — so `agag.plane` and the Plane service both remain, with cagent and
  nctl as their other consumers.
- **Not yet deployed or seen in a browser.** This step is the code and its
  fixtures; the relay restart, the web container rebuild and the live
  combined autolab/forge request are step 5.
- **A retired request's conversations are found but not shown as a pair.**
  The preview excludes a retired request with its reason, the way it does a
  replaced mission; it does not draw a link from the replacement back to it.
