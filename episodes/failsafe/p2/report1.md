# failsafe p2 — step 1: timing, facts, and resumption apart from acceptance

## What p1 left

`../p1/report.md` and `report4.md` were read against the code as it stands
(pyagag `f328cb7`, pj-agdev `cd1ca66`, agautolab `6fe2dfa`).

- **Detection runs on conversation evidence only.** The trace reads posts:
  - `execution` is `open` (an ack with no `end=`) or `ended`;
  - `holder` says who holds the next move.
  - Whether the harness process behind an `open` serving still exists is
    in no record. An `open` serving whose harness died is found only by
    `silent`: 2700 s of silence, then a model judgment.
- **Every long wait on the path.** Lowering one threshold leaves the others:

  | where | wait | value |
  |---|---|---|
  | `agag.trace.THRESHOLDS` | `silent` (judged) | 2700 s |
  | | `quiet` (judged) | 1800 s |
  | | `unheld` | 300 s |
  | | `unacknowledged`, `undelivered` | 300 s |
  | | `unstarted` | 180 s |
  | | `failed`, `resolved_live` | 60 s |
  | `agobserver.monitor` | look interval | 120 s |
  | | retry between requests (2 at most) | 600 s |
  | | `legit` postponement | 1 h, doubling to 4 h |
  | | owners told of a postponed wait | 6 h |
  | | judgment deadline (then `unclear`) | 900 s |
  | | `unclear` judgments before a report | 2 |
  | | unreadable conversation reported after | 1800 s |
  | `agobserver.triage` | one judgment's timeout | 150 s |
  | `agautolab` | progress posts at most every | 120 s |
  | | one task serving's timeout | 1200 s |

  A worker exit on p1's path therefore reached Front after about 45 to
  50 minutes. The worst case was longer: `silent` + interval + a queued
  judgment + a `legit` verdict, which postponed the next look by an hour.
- **Autolab's close-out** (`agautolab.zulip_listener._serve_run`) closes a
  task when its run writes `report.md` and the serving's processed input
  holds a post by somebody other than autolab. Autolab's own start note
  never counts (robust_workflow p3, trial G). In p1's T1 that post was
  Front's recovery request ("continue … and report task 1 done"). The
  resumed run inferred agreement from it and closed the task in the same
  serving (#11833 → #11839 `accepted … #11833`).

## Resumption is not acceptance (done)

agautolab `8f155f1`:

- **The rule.** A requester's post can close a task only when a post of
  autolab's in that task showed a result before it. That post either
  declares `intent=report` or requests confirmation (`response_request`,
  any ask except `question`). These show no result:
  - the task description;
  - the start line and progress posts;
  - acks;
  - a question.

  So "continue", "build it", or a resume after a stop asks for the work.
  It cannot agree to the result.
- **When the run writes `report.md` anyway:**
  - nothing is bound, integrated or started;
  - the reply shows the result;
  - the listener adds a confirmation request to the requester, the same
    `response_request … ask=confirmation` it already sends when nobody has
    agreed.
- **The requester's next post closes the task.** It follows a shown
  result, so it is the evidence in the `accepted` note.
- **The rule is mechanical.** It reads post intents (`ag.post.v1`) and
  their order. It does not interpret words, and Front's resume post needs
  no new vocabulary.
- **The supercoder guide says the same:** a resume request has not seen
  the result, so the worker shows the result and lets the developer agree.

Tests (agautolab, 308 passed):

- `test_a_resume_request_resumes_the_work_and_leaves_the_task_open`: T1's
  history. The resumed run writes `report.md`. Nothing is accepted,
  integrated or resolved, and the reply is a confirmation request to Front
  that names #18 as having come before any result.
- `test_an_acceptance_after_the_resumed_result_closes_the_task`: the same
  history, plus the resumed result and Front's "Accepted." The task closes
  with `accepted … #21`.
- `test_a_question_answered_is_not_a_result_agreed_to`.
- The shared fixture (`anchored()`) now shows a result before the
  Developer's post. Every closing test therefore exercises the rule.

## The facts kept apart

Each unit of work (a conversation with an identity, e.g. a `workrun-`
task) is described by five separate facts. Each has an observation time and
a source, and **unknown is a valid value for every one**:

| fact | values | read from |
|---|---|---|
| **process liveness** | `alive`, `exited`, `unknown` (+ pid, exit time/code when known) | the owner's health interface: the harness process behind the serving |
| **work progress** | time of the last confirmed progress + its source | harness events (a tool call, a message), the owner's work posts, `[change]` notes; never an ack, a claim, or a verdict |
| **waiting reason** | `tool` / `children` (a live tool call or background process, named), `requester` (an explicit response request, by id), `delegate` (a conversation or agent that holds the move), `none`, `unknown` | the health interface for what runs; the trace for what was asked |
| **observation freshness** | when each fact was observed, and how old its source was then | the probe result and the mirror's health |
| **recovery state** | `none` → `suspected` → `requested` → `escalated`; `recovered`, or a closed incident. Handed to a developer review separately (step 4) | Observer's own records, persisted |

A healthy **verdict** is derived from these facts only:

- `running`: alive, with progress within the probe threshold.
- `waiting`: alive, with an explained wait inside its own deadline.
- `stopped`: confirmed. The process is gone, or the serving failed or was
  abandoned without a closing post, or it ended handing the move to nobody,
  and the work is unfinished.
- `unknown`: anything else.

A listener heartbeat is not task health. An "it is still running" post is a
claim, not progress. A lack of output is not a failure.

## The timestamps

Every incident on the health path carries a timeline. It persists across
restarts:

| stamp | meaning |
|---|---|
| `onset` | best evidence of when the work stopped: the process's recorded exit, or else the last harness event, or else the serving's end. Marked estimated when not observed |
| `last_progress` | the last confirmed progress (above) |
| `first_suspicion` | the first look at which the unit was not confirmed healthy: no confirmed progress for the probe threshold, or a probe that could not confirm health |
| `health_checks` | each probe: when, its verdict, and its evidence key |
| `front_requested` | each request to Front (message id, time) |
| `escalated` | the report to the developer |
| `recovered` | the fresh evidence of work that ended the incident |

**Unchanged evidence does not reset uncertainty.** These leave
`first_suspicion` where it is:

- another "still running" post;
- a `legit` verdict;
- a probe result identical to the last one.

Only new progress clears it, or a probe that confirms a live, explained
wait. A new serving of the unit (a new ack) is new evidence. A new
episode starts from there.

## Operational targets (as implemented in steps 2–3; tuned in step 5)

| condition | action | constant |
|---|---|---|
| look interval | every request, every cycle | `AGOBSERVER_MONITOR_SECONDS` 120 → **60 s** |
| confirmed ended/failed execution with unfinished work | recovery request on the next cycle | `unheld` 300 → **60 s**; a probe's `stopped` → **same cycle** |
| no confirmed progress | health check | **120 s** (`PROBE_AFTER`), then every cycle while it lasts |
| continued uncertainty | ask Front to investigate | **180 s after first suspicion** (≤ 5 min with the interval and the post) |
| still unresolved | escalate to the developer with facts and unknowns | **600 s after first suspicion** |
| a confirmed healthy wait | stays under review, re-probed each cycle; no request | until its own deadline (the serving's timeout), then uncertain |
| one probe | bounded | **10 s**, then `unknown` |
| a model judgment on the paths that keep one | bounded | 900 → **240 s**, then `unclear` |

The end-to-end budget for a silent worker exit, with the exit at *t*:

- worst case without a probe: the last progress just before *t*, so
  *t* + 120 s (probe threshold) + 60 s (interval) + the probe and the post;
- with the probe, a confirmed `stopped` is asked for in the same cycle;
- about **3–4 min**, under the 5-minute target.

A threshold triggers **investigation**. It does not require agents to post
progress at fixed intervals. Harness events and the process are observed
directly.

## Next

Step 2 builds the health interface: a live execution record written beside
each harness run, and a bounded probe that reads it together with the
process table and the serving journal.
