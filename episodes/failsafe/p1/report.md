# failsafe p1 — report: recover a silently stalled request

## Outcome

P1's completion conditions are met:

| condition | result |
|---|---|
| The four trials meet their outcomes | ✓ T3 was 4 s over its target on the first run; after the fix (T3b) it met it |
| The silent stall reaches completion without Omni Agent rescue | ✓ |
| A restart preserves the recovery loop | ✓ |

The trials used the system's own path: Observer detects, Front decides and
resumes, autolab continues its work in its mission copy, and the requester
records the acceptance.

| trial | detection (target) | result |
|---|---|---|
| T1 worker ends after a progress post | 330 s (420 s) | resumed by Front 11 s after the request; completed 29 s later; mission `done` |
| T2 healthy work longer than the review interval | no incident in a 12-minute serving with 6-minute silences | no duplicate; normal completion |
| T3 progress misclassified as an answer | 2074 s (2070 s); after the fixes 1985 s (T3b) | the wrong state no longer exempts the work; judged `stall`; resumed and completed |
| T4 Observer restarts | during tracking and during recovery | nothing lost, nothing asked twice; the rescue counted "after 1 request" |

Other measures:

- **False interventions: 0** (one needless incident in T3, fixed).
- **Duplicate executions: 0.**
- **Human interventions**: the requester's own posts only.
- **Cost**: autolab supercoder runs in the trials cost $1.10.

Details are in `report4.md`.

## The contract (`report1.md`)

- **An obligation ends only by a record**: `done`/`cancelled` words.
  `replaced` transfers the obligation to the successor. None of these ends
  it: a serving's end, a process exit, an answer classification, a ✔, a
  `legit` verdict, or a sent recovery request.
- **Four separate facts per conversation**:
  - the conversation state (as before);
  - `execution`: `open`, `ended` or `unknown`;
  - `holder`: owner, delegate, requester, human, none or unknown;
  - recovery progress, kept by Observer.
- **Every tracked request stays due for review.** A judgment can postpone a
  look, never remove it. A judgment that fails or never returns is
  `unclear`.
- **Recovery is fresh evidence of work.** An acknowledgement or another
  promise is not. Observer never starts work, and when execution may still
  be live it asks for investigation, not a second run.

## Where it lives

**pyagag** (`cfcc80b`, `9b79405`, `f5c4359`; docs `f328cb7`):

- `agag.post`: `end=<ack>`; `compose` cuts a long post before its line;
  `is_truncated`. Spec: `docs/post-intent-v1.md`.
- `agag.topics`:
  - the listener writes `end=` on the reply that closes a serving;
  - `requester_of` never hands a reply to an `answer=none` aside.
- `agag.trace`:
  - `Node.execution`, `holder`, `ack`, `ended_by`, `work`, `taken_up`,
    `waiting_on`;
  - candidates `unheld` (mechanical) and `quiet` (judged, deepest unit
    only);
  - a Zulip-cut post is never an answer.
- `agag.selfnote.owed_start`: a Zulip-cut post never answers a start.

**agobserver** (pj-agdev):

- `monitor.py`:
  - `DETECTION_TARGET`; the rollout horizon `obligations_from`;
  - postponement with a ceiling and escalation; the judgment deadline;
  - work-evidence recovery for the new kinds;
  - obligations in `tracked.json`;
  - request routing (`asked_where`) and the request's facts and unknowns.
- `hold.py`: a person takes a request over.
- The triage guide: a "still running" post is only a claim.

**agfront** guides (`desk`, `front`, `routine_run`): "When Observer says
work has stopped". Resume by posting into the stopped conversation itself,
never beside an open serving, or ask the developer.

**agautolab**:

- the `workrun_supercoder` guide: wait for everything you started before
  replying, and resume from what the copy holds;
- the trial faults `stop-mid-task` and `stop-mid-task-unmarked`.

**Consumers on `f5c4359`**: agfront, agautolab, agforge, archsage, cagent,
the agentroom relay and agobserver. Their tests compare reply words through
`endmark.plain`.

**Documentation**: `README_DEV.md` (observer section, the trace section)
and `report1`–`report4`. Host details are in `pj-agdev/.local/devenv.md`.

## Tests

| suite | passed |
|---|---|
| pyagag | 936 (`test_failsafe.py`: 16) |
| agobserver | 130 (`test_failsafe_reproductions.py`: 13, the m11741 reproduction included) |
| agautolab | 305 |
| agfront | 180 |
| agforge | 265 |
| archsage | 37 |
| cagent | 204 |
| relay | 353 |

## Superseded behaviour removed

- A Zulip-truncated progress post read as an answer (trace and
  `owed_start`).
- A `legit` verdict re-asked at a fixed hour for ever, with no escalation.
- A judgment that never returned held its incident for ever.
- Every recovery request went to the origin conversation.
- A reply was handed to whoever spoke last, even an aside.

No code path became dead: the older incident kinds and their routing
remain.

## Remaining limitations

- **m11741 itself is held** (`agobserver.hold`, o11711). How to resume it is
  the Developer's decision. Release it to hand it to the monitor, which
  would then treat its task as a `silent` open serving (its posts carry no
  `end=`).
- **Requests before #11770 keep the older rules.** The tracked backlog with
  units of work nobody ever closed are listed for a decision: m6113, m7601,
  m7732, m8519 (mission and task 2), m9349 and m9697.
- **Plain conversations have no terminal record**, so their requests stay
  tracked indefinitely (harmless: nothing is asked).
- **Execution evidence is conversational.**
  - An `open` serving on a host that died is found only by `silent`
    (45 min, judged).
  - There are no leases or heartbeats. Cross-host reassignment and artifact
    replication are out of scope.
  - m11741's results still exist only on agstudio's disk.
- **Self-report is not improved beyond guidance.** autolab does not yet post
  `failed` itself when a serving ends with work in flight. Observer's
  `unheld` finds it within 7 minutes instead.
- **A resume can close a task in the same serving** (T1): the supercoder
  inferred the requester's agreement from the resume post.
- **The `quiet` judgment depends on the local model.** It judged correctly
  twice, and once failed with exit 2 (on the duplicate mission incident,
  now gone). Watching Observer itself is the relay's watchdog, unchanged.
