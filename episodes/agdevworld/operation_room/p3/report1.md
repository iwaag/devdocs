# Step 1 — the relay reads routines

`GET /routines` on the agentroom relay. One row per routine, carrying the
standing request, the local schedule, the last fire and — the question this
step exists for — **whether that fire was answered**.

The realm half costs **no Zulip call at all**. A routine's topics are in
`#front`, the `/ops` engine already sweeps every public channel and its event
queue already delivers every post in them, so the whole of this is a second
reading of memory the relay was holding anyway. The plan's constraint 3 (no
new polling) is not merely obeyed here; there was nothing to add.

## What a row is made of

| source | read as | why not otherwise |
|---|---|---|
| `#front` › `routine-<name>` | the standing request | no `front-` prefix, so Front never serves it: a document, not a conversation |
| `#front` › `front-routine-<name>` | the fires and the whole conversation | `trigger.sh` posts one line here as the Developer |
| `.local/rtschedule/schedule.json` | next / last / overdue events | the `:8093` GUI serves the same clone with **no CORS header** — a browser cannot read it |

`AGENTROOM_SCHEDULE_JSON` carries the path, in the launchd plist, never in a
tracked file (`devpolicy/styles.md`). Unset is an answer and not an empty
schedule: the payload says `configured: false`, the way `/ops` says which
credential is missing rather than showing a quiet board.

## Three rules the live realm corrected

**An ack is not an answer.** Front posts `Message received. Please wait for
the reply.` to everything it is served. A boolean "was there a reply" would
have called every routine answered a second after it fired. So there are three
states — `answered`, `acked`, `unanswered` — and the age is always measured
from the **fire**, never from the ack: the human's question is how long ago
they asked for this, not how long ago the transport agreed to carry it.

**A fire is the trigger's own wording**, `` Routine `<name>`, run of … ``.
The obvious rule — the newest post by the Developer — would have made every
comment on a run into a new unanswered fire, and the Developer comments on
runs constantly.

**The standing request is the newest post by the topic's *author*, not the
newest post.** `trigger.sh` tells Front the request is "the latest post in
`#front` › `routine-<name>`", and on this realm that sentence is false: on
2026-09-04 Front filed a `ghtrends` run report **into the standing-request
topic** instead of the fire topic, so the latest post there is a report about
the routine. A board that repeated the trigger's rule would have shown that
report as the routine's standing request — and so, presumably, does Front on
its next `ghtrends` run. `routine-rtnotes` has fourteen such posts.

The stray posts are served as `request_strays` rather than dropped. They are
the evidence for the rule and, separately, a defect somebody should fix.

## Live reading, 2026-09-06

Eight routines, from a 239-call sweep of 119 topics in 55 channels:

| routine | state | last fire answered | fire-topic posts | strays |
|---|---|---|---|---|
| ghtrends | done | answered in 57 s | 33 | 1 |
| imgprompt | done | answered | 59 | 0 |
| localtest | done | answered | 59 | 0 |
| manual | done | answered | 18 | 0 |
| **mediagen** | **unknown** | **no fire** | 164 | 0 |
| papers | done | answered | 82 | 0 |
| publish | done | answered | 48 | 0 |
| rtnotes | done | answered | 146 | 14 |

**`mediagen` has never been fired by the dispatcher.** Its fire topic is the
busiest of the eight — 164 posts — and not one of them is a trigger line: every
run of it was started by hand. The row says `unknown`, which is the p2 rule
applied here (an absence is never rendered as quiet), and it is the correct
answer: this is a routine in name and a conversation in fact.

Only `papers` has schedule events left; the dispatcher prunes at seven days,
so the other seven routines have a realm history and no schedule at all. The
board shows both halves separately rather than reconciling them into one
number, because they disagree for a reason.

## Two changes inside the engine

- **`Topic` now keeps history and links.** Full history (200 messages) for the
  sixteen routine topics only — the chat view of step 4 *is* that history, and
  a topic costs one call whatever depth it is read at. And, for every topic,
  the two selfnote link notes: `[rootchat]` (what this topic was opened for)
  and `[served]` (which remote callbacks its owner has answered). Step 2 is
  built on those edges. Constraint 4 holds as written — they link, they are
  never rendered, and `history` never contains one.
- **A routine's two topics are read through a ✔.** Everywhere else a resolved
  topic is skipped on the sweep, because reading every ✔ topic on the realm
  would multiply the one cost p1 capped. Here resolution is how a routine is
  *retired*, and there are sixteen such topics rather than a realm's worth.

`70 passed` (`tests/test_routines.py` is 22 of them). The live realm proves
none of the states that matter: every routine on it is answered, and a broken
route reads exactly like a quiet board.
