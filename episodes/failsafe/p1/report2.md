# failsafe p1 — step 2: unresolved work stays due for review

## What changed

### pyagag `cfcc80b`

- **A serving's end is on record.** The listener writes `end=<ack id>` on
  the `ag-post` line of the reply that closes a serving
  (`agag.topics._with_meta`; also on the "nothing to answer" reply). A
  reply with no intent carries a line that says only `end=`. The field is
  new in `ag.post.v1` (`agag.post.PostMeta.end`). It says nothing about
  the words' meaning.
- **A marker survives transport.** `agag.post.compose` cuts a post longer
  than Zulip keeps (10 000 characters) *before* its line, and says it did
  (`[… cut by the poster to fit]`). autolab's live progress posts go
  through `compose`, so the #11758/#11759 truncation cannot recur. A post
  that Zulip already cut (`[message truncated]`) reads as output, never as
  an answer. That holds in the trace and in `owed_start`, where autolab's
  own start logic had also taken #11758 as the task's answer.
- **The trace separates execution and holder from the conversation state**
  (`agag.trace.Node`):
  - `execution`: `open`, `ended` or `unknown`. `ended` needs the `end=`
    mark, or an answer after the ack. Progress ends nothing.
  - `holder`: `owner`, `delegate`, `requester`, `human`, `none`, `unknown`
    or `done`. A child conversation holds its parent's work until it
    finishes or its answer is taken up. So does an agent the owner's
    current serving named in the same conversation and that has not spoken
    since. This is the supercoder guide's `@**Comfy Notifier** watch …`,
    acked with a reaction.
  - A child that waits only on its requester holds nothing, so a circular
    wait comes out as `none`.
  - Also recorded: `ack`, `ended_by`, `work`, `taken_up`, `waiting_on`.
    `agentchat trace` prints the serving and the holder under each
    conversation ("NOBODY holds the next move").
- **Two new stall candidates** (`THRESHOLDS`, `FAILSAFE_KINDS`):
  - `unheld`: holder `none` for 300 s. Mechanical.
  - `quiet`: a unit of work (identity note) whose holder is `unknown` or a
    requester, with nothing new anywhere below the origin for 1800 s and
    nobody explicitly asked a person. Judged.

### agobserver

- **Review scheduling does not depend on judgments** (`monitor.py`):
  - A `legit` verdict postpones the next judgment by `REJUDGE_SECONDS`
    (1 h), doubling on the same unmoving evidence up to 4 h.
  - `silent` and `quiet` waits judged legitimate for `MAX_POSTPONED_SECONDS`
    (6 h) with nothing moving are reported to the realm's owners once.
  - A judgment still without a verdict after `JUDGMENT_DEADLINE_SECONDS`
    (15 min) counts as `unclear`. Two unclear verdicts are reported, as
    before.
- **`unheld` and `quiet` recover only on work evidence**
  (`Monitor.recovered`). Both need a serving that began after detection
  (`ack_at_detection`) and that either showed work while open or ended
  handing the move to somebody. An ack, or a second "I'll report" promise,
  leaves the incident recovering.
- **Tracked requests carry the contract's durable facts** (`tracked.json`):
  - `obligations`: anchor → topic, identity, state, execution, holder;
  - `evidence_at`, `next_review` (postponement and retry due times from
    the incident records) and `contract`.
  - Incident records keep `postpone_seconds`, `postponed_since`,
    `postponements` and `ack_at_detection`. A restart resumes from them.
- **The rollout horizon** `obligations_from` sits beside `receipts_from` in
  `monitor-state.json`. It was set at this deployment to **#11770**.
  Requests older than it get no `unheld` or `quiet` and no escalation of
  long postponements.
- **`python -m agobserver.hold <why> o<id>`** (and `--release`) is a person's
  decision to take a request over. The request stays traced and tracked,
  and nothing is opened or asked about it. It is kept in `held.json`, which
  the monitor only reads.
- `DETECTION_TARGET` (look interval + grace + judgment ceiling) is what the
  trials measure against:

  | kind | target |
  |---|---|
  | `unheld` | 420 s |
  | `silent` | 2970 s |
  | `quiet` | 2070 s |

- The triage guide gained one piece of evidence-driven guidance from
  m11741. A "still running" post is the claim of the serving that wrote it.
  The trace's serving line says whether anything is open.

## Tests

- pyagag: **933 passed**. `tests/test_failsafe.py` (13) covers:
  - the wire: the `end=` mark, the size cap, a Zulip-cut post that is not a
    start's answer;
  - holders: ended-with-progress is `none` after its grace, a live serving
    is its owner's, a report hands the move back, an unmarked answer ends a
    serving;
  - delegation to another conversation, a same-conversation notifier watch,
    and a circular wait;
  - `quiet`, and its exemption while a person is asked.

  Tests about other properties compare replies without the mark
  (`tests/endmark.py`). The post and topics tests assert the mark itself.
- agobserver: **130 passed**. `tests/test_failsafe_reproductions.py` (13):
  - the step 1 reproduction, now passing (xfail removed):
    - fixed posts reach Front with no judgment consulted, within the
      `unheld` target;
    - as posted with a `stall` verdict, within the `silent` target;
    - as posted with a `legit` verdict, judged again after the
      postponement and reported to the owners at 6 h;
  - recovery: an ack plus a promise is not recovered; resumed work is
    rescued and the task stays tracked;
  - restarts: one during recovery asks nothing extra, and the bounded
    second request still comes after the retry interval; one during a
    postponement is judged neither early nor late;
  - robustness:
    - 80 min of healthy long work is neither asked about nor judged;
    - a judge that never answers ends in a report;
    - requests before the horizon keep the older rules;
    - a held request is traced but not acted on.

## A check against the realm

The new trace was run over every `front-` request in a copy of Observer's
mirror (15:40 UTC):

- Holders of unfinished conversations: 97 `requester`/`ended`, 7
  `delegate`, 3 `owner`/`open`, 3 `owner`/`unknown`, 1 `requester`/`unknown`.
- `unheld` fired nowhere. Historical posts carry no `end=`, so a legacy
  final progress post reads `open`, never `none`.
- `quiet` fired on 7 old units of work, all before the horizon: m6113,
  m7601, m7732, m8519 (mission and task 2), m9349 and m9697. None is
  nudged. They are listed for the phase report as obligations nobody ever
  closed.
- One consequence for the live stall: m11741's task is no longer
  `awaiting_requester`. It reads `executing` (open serving, last sign of
  work at 13:43), so the unchanged `silent` rule applies to it.

## Deployment

- `nctl status`: ok. `nctl drift --host agstudio`: converged, 1 info diff.
  agobserver is a manual toolchain service there.
- agobserver's lock was moved to pyagag `cfcc80b`. The listener
  (`com.agdev.agobserver-zulip`) was restarted at 15:47:49 UTC while idle
  (no judgment running). It logged "holds requests from #11770 to the
  failsafe contract". The relay's watchdog reads `ok`.
- **m11741 (o11711) is held** before the restart. Otherwise the new
  reading would have opened a `silent` incident and could have asked Front
  to resume the Developer's mission, whose resumption the case document
  leaves as the Developer's open question. `python -m agobserver.hold
  --release o11711` hands it back to the monitor.
- The other consumers (Front, autolab, forge, archsage, cagent, the relay)
  still run their older pyagag, so their replies carry no `end=` yet. They
  move in step 3/4 with the recovery guidance.
