# routine_tests p2 ex2 step 3 — observing the run

Date: 2026-09-13 JST (timestamps UTC). Plan: [plan.md](plan.md) step 3.
Previous: [report1.md](report1.md), [report2.md](report2.md).

The run started from the one request in [report2](report2.md) and finished
**`achieved: true`** 28 minutes later, with one intervention. What the
researcher did and how its figures hold up is [report4](report4.md); this
report is the routing, the timeline and the help it needed.

## Conversations

| | |
|---|---|
| request | `#front` › `front-uspolitics-20260912T1853Z`, message 6615 |
| run | `#routine-study-uspolitics` › `routinerun-2026-09-13T1853Z`, opened 6618, guide 6393 (v2) |
| delegation | `#pj-studyuspolitics` › `workplan-next-uspolitics-slice` — **a new topic name**, request 6622 |
| mission | **m6625** (`[selfnote][mission]` 6625), plan 6626 |
| task | `#work-m6625` › `workrun-task1-m6625`, description 6630 |
| `main` commit | **`84bbd91`**, pushed (origin `main` = `84bbd91`) |

## Timeline

| UTC | msg | What |
|---|---|---|
| 18:53:17 | 6615 | request posted |
| 18:53:40 | 6618 / 6619 | run opened; Front tells the Developer where; listener `starting run …` |
| 18:55:11 | 6621 / 6622 | run serving 1 delegates: root note **names the run**; request restates the guide's goal and rules, adds no research direction |
| 18:55:21 | 6624 | run entry: checked the project's earlier missions first, found them finished, chose to ask fresh |
| 18:56:48 | 6625–6632 | autolab's plan (m6625), task opened, `@Front` |
| 18:56:49 | — | `mention … serves routine-study-uspolitics/routinerun-2026-09-13T1853Z` |
| 18:57:33 | 6634 / 6635 | run serving 2 approves and starts task 1 itself; root note in the task names the run |
| 18:57:43 | 6637 | run entry quoting the plan |
| 18:59:22 | 6639 / 6640 | task session ends after launching a 2.18 GB download under `nohup`: *"The download will notify me on completion. I'll pause here"* — `@Front` |
| 18:59:47 | 6642 | run serving 3 judges it correctly as not done and waits |
| ~19:00 | — | download complete; nothing serves anybody |
| 19:12:06 | **6644** | **intervention 1** — Developer: the download has finished; please continue |
| 19:20:07 | 6650 | task reports the analysis done, **uncommitted**, and asks for approval — `@Developer @Front` |
| 19:20:32 | 6652 | run serving 4 approves and asks for the commit hash |
| 19:20:45 | 6657 | run serving 5: re-read, nothing new (see *extra serving* below) |
| 19:21:13 | 6660–6663 | task: commit `84bbd91` pushed, `[state] completed`, autolab resolves the task topic |
| 19:21:55 | 6667 | run serving 6: entry and **one** `ag-routinerun` block, `achieved: true`; run topic resolved |
| 19:21:55 | 6665 / 6666 | listener delivers the report to the request conversation and writes `[delivered]` |
| 19:22:03 | 6671 | continuation serving in `#front`: one turn, summarises, asks nothing |

## Routing, as it happened

- **B held throughout.** Every delegation the run made — the plan request
  (6621) and the task start (6634) — carries a root note naming the run.
  Every callback — plan (18:56:49), pause (18:59:22), analysis (19:20:07),
  completion (19:21:13) — was logged `serves
  routine-study-uspolitics/routinerun-2026-09-13T1853Z` and was served by
  `routine_run`. Nothing reached the Front Desk conversation until the
  listener's delivery.
- **No topic-name collision.** The run chose `workplan-next-uspolitics-slice`,
  a name none of the four open `workplan-` topics has.
- **No retirement**, so A was not exercised.
- **The threads question.** The plan callback's prompt carries the run's
  own conversation; autolab's plan was only in `threads/…/workplan-next-uspolitics-slice.md`
  (the prompt builder adds a placement line for threads, not their text).
  Serving 2 took 9 turns and its entry quotes the plan by message range and
  by content that exists only in that thread, so it opened it. It did not
  answer from the chatlog alone. No transcript is kept for this role, so
  *how* it read it (file or `agentchat read`) is not recorded.
- **Continuation fired once and did not loop.** `continuing
  front/front-uspolitics-20260912T1853Z: a run reported there and nothing
  has served it` → one 1-turn serving (Front `front` run-0616) → speech spent
  the note.

## The stall — the p2 first-attempt behaviour, recurred

The task's second step needed `indiv26.zip` (2,180,607,680 bytes). The
supercoder session (run-0268, 31 turns, 102.7 s) started it with `nohup curl
… &`, started a `while` loop waiting for the file size, then ended its
session with *"The download will notify me on completion"*, naming Front.
The loop died with the session; the `curl` was reparented and finished
around 19:00. **No process in the system turns a background job's end into a
post**, so autolab was never served again, and Front — correctly judging a
pause as not done (6642) — had nobody to wait for but did not know it.

This is the same shape as p2's first attempt (a self-throttled background
fetch the task left running when it ended). It is **not** the human-dependency
case: nothing needed a human, and the guide's rule about asking was not
engaged. The researcher did not stall silently either — it said what it was
waiting for; what it expected to wake it does not exist.

Evidence kept: the task topic at the stall point, supercoder run-0268, both
listener log segments, Front's entry 6642, the download's size and process
state at 18:59:47 and at 19:11.

## Intervention and assistance

| # | UTC | What | Why | Smallest? |
|---|---|---|---|---|
| 1 | 19:12:06 | Developer posted in the task topic (6644) that the download had finished and asked it to continue — no research content | the stall above; nothing else would ever serve the task | yes: one post, a fact the agent could not observe. The developer approved it before it was posted |

> **Deus Ex Machina note:** *did the "your background job has finished" wake-up
> for agent autolab — handoff candidate.*

Nothing else was needed: no anchor repair, no re-stated request, no
serving by hand, no answer to a human-dependency question. **Front's
approvals (6635, 6652) and the run's end were the agents' own.**

## Two further observations, not on the step-1 list

- **An extra run serving.** At 19:20:44 the listener marked the task topic
  served up to 6653 and at 19:20:45 served a mention in it again; serving 5
  (routine_run run-0057, 2 turns, $0.1091) found only autolab's
  acknowledgement and wrote "nothing new". Most likely autolab's
  `Message received` ack after Front's approval arrived as a new mention
  event while serving 4 was running. One wasted serving, no harm.
- **"Developer (Front) reviewed and approved."** autolab's result post calls
  the approver the Developer. The approval was Front's; the Developer said
  nothing in that topic but the wake-up. The task's `@Developer` mention in
  6650 also asked the Developer for approval that the run then gave. No
  consequence here, but the record misstates who agreed.

## Topic state left behind

- `✔ routinerun-2026-09-13T1853Z` (Front), `✔ workrun-task1-m6625` (autolab).
- `workplan-next-uspolitics-slice` is **unresolved**, like the four earlier
  `workplan-` topics; m6625 has no `done`/`accepted` note from the requester.
  Not tidied, per the plan.
