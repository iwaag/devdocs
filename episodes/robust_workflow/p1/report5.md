# Step 5 report — normal operation and failure recovery without rescue

Plan: [plan.md](plan.md) step 5. 2026-09-24 (JST), by the Omni Agent as the requester's
stand-in and trial operator. Test bed: the disposable project `pj-robustp1` (opened by Front in
T1 for this purpose).

## Trial plan, set before verification

The plan's initial budget was three normal cycles and two trials per failure case. Adjusted
from what steps 1–4 showed; the reasons are in the last column.

| Case | Live | Fixtures | Reason for the count |
|---|---:|---|---|
| Sequential authorized tasks | 3 cycles (N1–N3; N2 is forge → autolab) | pyagag `test_progression`, `test_listen` start notes; agautolab `test_progression` | as planned |
| Missing start | 2 (T1, done in step 4; T2 in N3) | `test_monitor`, `test_trace` p3 replay | as planned |
| Reply or delivery failure | 1 (Front's listener down while an answer arrives) | pyagag serving/delivery suites (explicit_reply p1), `test_trace` awaiting_delivery | the delivery machinery is unchanged by this episode and already has fixtures for ambiguous sends and restarts; one live check that the new callback path (parent hop) survives a downtime |
| Missing command or permission | 1 (a write the tool refuses) | `test_chat` opfail | cheap and deterministic once seen |
| Accidental resolve or rename | 1 (a ✔ on a running task by a third party) | `test_chat` resolve guard / unresolve, `test_trace` resolved-unfinished | the agent-side guard makes the p3 path a refusal; the live case is the one the guard cannot prevent (a human ✔) |
| Long-running work or human approval wait | every live trial (the requester's review time is a real human wait) | the p3 twin judged `legit` twice live (step 4) | a 45-minute `silent` threshold costs an hour per trial; judged kinds are covered by fixture and by the live `legit` |
| Unreadable target or stale observation | 0 new | `test_trace` unobservable; step 4's live stale-mirror finding and fix | found and fixed live already |
| Repeated detection or listener restart | 2 (autolab restarted with an acceptance pending, N1; Observer restarted with an incident open, N3) | `test_monitor` restart / lost store | as planned |

**Targets**

| Measure | Target |
|---|---|
| Normal cycles completed, results at the requester | 3 / 3 |
| Mechanical relays by Front on the normal path | 0 task-start posts |
| Missing start: detection (fault → request) | ≤ 6 min |
| Recovery (request → work served) | ≤ 2 min |
| Duplicate execution | 0 |
| False success reports | 0 |
| Omni Agent rescue interventions | 0 (requests, acceptances and quality checks are not rescue) |
| False-positive incidents on live trials | 0 |
| Unrecoverable case | an actionable stop report naming the next responsible party |

Cost is recorded per trial from the run records; Zulip calls added by the monitor are 0 per
look by construction and its posts are counted.

## Results

### Trials run

All through the ordinary entrance (`#front`), the Omni Agent posting as the requester's
stand-in; faults injected by the operator, never repaired by hand. Times UTC, 2026-09-23.

| Trial | What | Outcome |
|---|---|---|
| **T1** (step 4) | 2-task mission; one-shot fault skips task 2's automatic start | Observer asked at +3 min 32 s, Front started task 2 7 s later, rescue recorded — [report4](report4.md) |
| **N1** | 3-task mission; **autolab's listener stopped** while task 1's acceptance was relayed, restarted 21 s later | recovery queued 1, closed task 1, started task 2 itself; tasks 2 and 3 started by autolab; 0 start relays |
| **N2** | forge → human review → autolab (icon into the README); **Front's listener stopped** while forge delivered, for 11 min | the delivery was served once on Front's restart (recovery queued 1, marked served). **Observer did not detect it** — see defect 3 |
| **N3** | 2-task mission; **a human ✔ on task 1 while it ran**; one-shot fault on task 2's start; **Observer restarted with an incident open**; a side request to post into an archived channel | ✔ found and judged a stall by the local model, recovered (below); an undelivered closing report found and recovered; the missing start recovered in the same serving; restart recorded the rescue without a second request; the impossible post reported honestly |
| **S1** | **Front's listener stopped**, a question posted into a new conversation | Observer reported to the realm's owner by name at +6 min, asked nobody (there was nobody to ask); Front answered 8 s after it was back |

### Against the targets

| Measure | Target | Result |
|---|---|---|
| Normal cycles completed, results at the requester | 3 / 3 | **3 / 3** (N1, N2, N3; T1 as well) |
| Task-start relays by Front on the normal path | 0 | **0** in all four missions (9 tasks). The two start posts Front made (T1, N3) were recoveries of injected faults, asked for by Observer |
| Missing start: fault → recovery request | ≤ 6 min | T1 **3 min 32 s**; N3 **6 min 53 s** ✗ — the task 1 closing report had not been delivered (defect 6), so the stall surfaced as `undelivered` first, whose grace is 5 min |
| Recovery request → work served | ≤ 2 min | **7 s** (T1), **11 s** (N3, both the delivery and the start in one serving) |
| Duplicate execution | 0 | **0** (autolab restart, Front restart twice, Observer restart, repeated close-outs) |
| False success reports | 0 | **0**. Incorrect reports that were not success claims: T1 "not pushed" (was pushed; fixed, N3 correct), N1 "tasks 2 and 3 NOT_STARTED" (task 2 was running; defect 2), N2 "the next task hasn't started" of a one-task mission, N3 a stray "EOF_NOT_NEEDED" correction |
| Omni Agent rescue interventions | 0 | **0**. One human decision worth naming: in N3 Front declined to undo a ✔ a *human* had set and asked; the requester's answer "the ✔ was my mistake" is a decision about the requester's own action, not a rescue. Front then ran `agentchat unresolve` itself |
| False-positive recovery requests | 0 | **0**. Three candidates were judged `legit` by the local model and dismissed without asking anybody (p3's twin twice before the mirror fix, and N3's task 1 after its normal close) |
| Unrecoverable case | actionable stop report | **S1**: `@**Developer** Stopped: I could not get this moving…` with the fact, the expected next action and the request; responsible party now named (defect 7) |
| Missing command or permission | failure visible, no false success | **N3**: Front read the channel list, found `pj-refactorp1` gone, posted nothing elsewhere, and said so in every report |
| Long-running work / human approval wait | no nudge | the requester's own waits (up to 15 min in N2, an unanswered "Can I proceed?" in S1) produced no candidate; the 45-min `silent` path is covered by fixture only |

### Cost

| Trial | Runs | Cost | Front posts to other agents |
|---|---:|---:|---|
| T1 | 17 | $1.71 | 6 (goal, setup, workplan, 2 acceptances, 1 recovery start) |
| N1 | 21 | $1.80 | 5 (workplan, 3 acceptances, mission close) |
| N2 | 15 | $1.47 | forge request, run start, workplan, acceptance |
| N3 | 17 | $1.47 + 2 triage runs ($0, local) | workplan, 2 acceptances, 1 recovery start, mission close |
| S1 | 2 | $0.12 | 0 |
| **Total** | **72** | **$6.57** | |

Zulip calls added by the monitor: 0 per look (mirror reads); posts per incident 2–4; one
`subscriptions` read per 10 min. Local-model judgments: 6 in the episode, 43–91 s each.

### Defects found by the trials, and fixed

| # | Found in | Defect | Fix |
|---|---|---|---|
| 1 | N1 | Front's reply mark closed with `</ag-reply>`, the repair with three backticks after four: the requester got only "this run produced no reply" | the mark accepts both when unambiguous (pyagag `5da08bf`) |
| 2 | N1 | `agentchat trace` without an id traced a callback's anchor — the task topic that named Front — so Front saw one task and reported the rest from memory | defaults to the conversation being served (`5da08bf`) |
| 3 | N2 | forge's `delivered` note made the trace say *done* while the requester had never been served: no candidate for 11 min | an answer is owed until the requester takes it up — served mark, or the requester speaking at home since (`e705641`) |
| 4 | N3 | autolab's progress lines were posted under the old name after a ✔, opening a twin | progress follows the live name (agautolab `358556f`) |
| 5 | N3 | that twin made `agentchat unresolve` refuse, which would have blocked the recovery Observer and Front both named | a stray post with no conversation notes of its own is folded back in (`12b2145`) |
| 6 | N3 | **the listener skipped a callback whose topic was ✔'d right after it** — autolab resolves every task right after its closing report, so the report reached the requester only if the listener looked before the rename arrived | mentions in somebody else's ✔'d conversation are served, at intake and in `owed` (`87ac87e`, deployed to all six listeners) |
| 7 | S1 | the stop report could only say "the agent that owns this conversation" | names the entrance's owner (agobserver `879fb49`) |

Defect 6 is a plausible cause of earlier "lost completion report" incidents (agent_standardize
p9 fixed the reading side of the same race); it was caught here because Observer's
`undelivered` check (defect 3's fix) flagged it and asked Front, which recovered it.

### What these trials do not prove

A handful of runs on one small project. No trial ran a task longer than a few minutes, so the
`silent` judgment and the recovery of a long job are proved by fixtures only. The judged
`stall` verdict was exercised once live. The sample is not evidence of reliability at scale.
