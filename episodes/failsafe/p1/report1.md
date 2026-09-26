# failsafe p1 — step 1: the recovery contract and the reproduced gap

## The gap, reproduced

The historical case, m11741 (`case_and_opinion1.md`), is reproduced in
miniature in `agobserver/tests/test_failsafe_reproductions.py`. It includes
Front's Front Desk conversation, the routine run Front opened, autolab's
mission and its one self-started task. It ends on the final reply saying
"the local kit run is still running … I'll report". It does not depend on
m11741's current state. The same posts are built two ways:

- **as posted**: two progress posts cut by Zulip at 10 000 characters,
  ending in `[message truncated]`, which removed the `ag-post` line. Nothing
  says the serving ended.
- **fixed**: every marker intact, and the final reply saying which serving
  it ends (the contract below).

What today's code does with it, measured by driving the current
`agobserver.monitor` for two hours of simulated time (looks every 120 s,
with a judge that believes the last post, as the post invites):

| realm | task state (trace) | what the monitor did in 2 h |
|---|---|---|
| as posted (the historical shapes) | `awaiting_requester`: the cut post reads as the task's answer | nothing: no candidate, no incident, no request |
| markers intact, no end evidence | `executing`, last sign of work = the final reply | `silent` after 45 min, judged `legit` twice (at +45 min and +105 min), dismissed; Front never asked |

The live trace of the real request (`agentchat trace 11711`, 15:05 UTC)
matches the first row. It shows `work-m11741 › workrun-task1-m11741:
AWAITING_REQUESTER — answered #11759 for Front 1h36m ago`, and #11759 is a
truncated progress post.

Two findings go beyond the case document:

1. **Correct markers would not have been enough.** With the marker intact
   the task reads `executing`. The only candidate is `silent`, which a model
   has to judge. The evidence it would judge is a post saying the work is
   still running. Nothing in Zulip says that the serving which wrote that
   post has ended.
2. **The final reply was a progress post, and nothing reads that as a
   fact.** #11766 is the serving's own reply (`ag-reply intent=progress`),
   not a live progress line. A serving that ends by saying "work goes on"
   leaves no process holding the work unless it handed the work to somebody.
   That is decidable without a model, once the end of a serving is visible.

The routine run (`routinerun-…`, owned by Front) reads `not_started` and
the mission `awaiting_requester`. Neither is a stall of its own: Front was
correctly waiting on autolab's report. The obligation that was dropped is
the task's.

The reproduction's five tests are committed as a strict `xfail` and fail
today for the reasons above. Step 2 removes the mark.

## The contract

### What a request owes, and what ends it

- A **request** is its origin conversation's first post (`o<id>`, as
  today).
- An **obligation** is a conversation opened for the request (a trace node
  below the origin) that still owes something:
  - A **unit of work** (identity note: mission, task, asset, assetrun,
    change) is an obligation until its record says it ended.
  - A **plain conversation** (a question to archsage, a setup exchange) is
    an obligation only while something is owed in it: queued, executing, an
    answer undelivered, an explicit request pending, or nobody holding it
    (below).
- **Only a record ends an obligation.** Done words (`completed`, `accepted`,
  `done`, `delivered`) and cancel words (`cancelled`, `retired`) do. So does
  `replaced`, which **transfers** the obligation to the successor it names;
  it does not end it. The following end nothing: a process exit, the end of
  a serving, an answer classification, a ✔, a `legit` verdict, a recovery
  request sent, or an acknowledgement.

### Four separate facts per conversation

| fact | values | decided from |
|---|---|---|
| conversation `state` | today's vocabulary (`queued`, `executing`, `awaiting_requester`, …) | who spoke last, notes, served marks, `ag-post` lines. **Change:** a post Zulip cut (`[message truncated]`) without its line is unclassified, never an answer |
| `execution` of the owner's serving | `running` (acked, not ended), `ended`, `unknown` | the owner's ack, and **a reply saying which serving it ends**: `ag-post … end=<ack id>`, written by the listener on the reply that closes a serving |
| `holder` of the next move | `owner`, `delegate`, `requester`, `human`, `none`, `unknown`, `done` | execution, the node's children, and the final reply's intent (below) |
| recovery progress | incident state and attempts, the request's next review | Observer's store (below), never the conversation state |

The holder, for a conversation that is not finished:

- **owner**: its serving is running, or a post waits for its listener.
- **delegate**: a child conversation still holds something. A child holds
  something unless it is finished or its answer was taken up by this
  conversation's owner. A child waiting on its own requester (this owner)
  holds nothing, so a circular wait comes out as `none` rather than
  "delegated".
- **requester** or **human**: the owner's last ended serving handed the
  move over. It asked a pending question (`response_request`), reported, or
  the task is `held`, or an answer is owed delivery. It is `human` when the
  recipient is a person.
- **none**: the last serving **ended**, its final reply said only that work
  goes on (`progress`), and no child or request holds anything. Nothing is
  running and nothing will wake it. This is m11741.
- **unknown**: there is no evidence either way. This covers historical
  posts and agents that do not write `end=`.

### What Observer keeps, and when it looks again

Per tracked request (`tracked.json`, as today, extended):

- identity: `o<id>` and the origin anchor;
- the outstanding obligations by anchor, each with state, execution,
  holder and the newest evidence time;
- `evidence_at`, the newest post anywhere in the request's tree;
- `next_review`, and postponements (how many, since when);
- the incidents opened for it, each with its attempts and next
  verification time, in the incident record (unchanged);
- on release: the record that ended each obligation (message id).

Rules:

1. **Every tracked request has a `next_review`.** A look happens on schedule
   whatever the judge does. A judgment can move a review later. It cannot
   remove it. A judgment that fails or never returns counts as `unclear`
   once it is overdue.
2. **New detection kinds** beside today's:
   - `unheld`: holder `none` for a 300 s grace. Mechanical: no model.
   - `quiet`: a unit of work whose holder is `unknown` or an agent requester,
     with nothing new anywhere in the tree for 1800 s. Judged, because the
     evidence is missing rather than contrary.
3. **A `legit` verdict postpones.** The review comes back after
   `REJUDGE_SECONDS`, and each further postponement doubles it. A wait that
   has not moved for `MAX_POSTPONED_SECONDS` is reported to the realm's
   owners, the existing human-reporting path. It is never looked at for
   ever in silence.
4. **Recovery is fresh evidence of work, not an acknowledgement.** For
   `unheld` and `quiet`, a rescue needs a serving that started after the
   incident **and** evidence of work after it: a progress line, a
   checkpoint, an answer or a record. After a rescue the request stays
   tracked until the obligation's record lands. A later stall of the same
   work is a new episode of the same incident.
5. **No competing execution.** Observer never starts work. When execution is
   `running` or `unknown`, what it asks for is investigation. Front and
   autolab decide whether to wait, resume the stopped task in its mission
   copy (a post in its `workrun-` topic), report a blocker, or ask the
   Developer.
6. **Restart** reads the store. A lost store is rebuilt from the incident
   notes (as today), and the review schedule of a request with no record
   starts at its next look.

Detection targets, from the 120 s look interval (`DETECTION_TARGET` in the
monitor):

- `unheld`: 120 + 300 = 420 s;
- `quiet`: 120 + 1800 s, plus the judgment's time (up to 150 s).

### Expected outcome of the reproduction after the fix

| realm | expected |
|---|---|
| fixed posts | task `execution=ended`, `holder=none`; `unheld` within 420 s; Front asked; no judgment consulted |
| as posted, judge says `stall` | the truncated post is not an answer; task `execution=unknown`; `quiet` within the quiet target; Front asked |
| as posted, judge says `legit` | judged again after the postponement; reported to the owners once `MAX_POSTPONED_SECONDS` pass without movement |

## Scope notes

- Requests that entered tracking before the new contract is deployed carry
  no `end=` evidence. They also include long-abandoned trial fixtures: the
  Observer currently tracks o8286, o8512, o8721, o9227 and o9562, each with
  unfinished units of work from 2026-09-23. The rollout will set a horizon
  (like `receipts_from`). Requests older than it keep today's rules, and
  they are listed in the phase report for a human decision rather than
  nudged at Front.
- The live m11741 is left as it stands. It is the Developer's to resume.
