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
