# routine_tests p2 ex2 — study-uspolitics again, on the repaired system

Date: 2026-09-13 JST. Plan: [plan.md](plan.md). Source:
[braindump.md](braindump.md). Steps: [1](report1.md) · [2](report2.md) ·
[3](report3.md) · [4](report4.md) · [5](report5.md).

One run of `study-uspolitics`, asked for in one sentence, to learn two
things: whether the problems p2 met recur after [ex1](../ex1/report.md)'s
repairs, and which way the researcher takes the study when told nothing.

## The run

| | |
|---|---|
| request | `#front` › `front-uspolitics-20260912T1853Z`, **message 6615**: *Run the routine `study-uspolitics` once and report back here. One run only.* |
| run | `#routine-study-uspolitics` › `routinerun-2026-09-13T1853Z`, guide **v2, message 6393** |
| mission | `#pj-studyuspolitics` › `workplan-next-uspolitics-slice`, **m6625**, plan 6626 |
| task | `#work-m6625` › `workrun-task1-m6625` |
| `main` | **`84bbd91`**, pushed: new `methods/fec-senate-2026-itemized-donor-geography-employer-occupation.md`; `reports/2026-fec-senate-financial-summaries.md` extended with Findings 7 and 8 |
| time | 18:53:17 → 19:21:55 UTC, 28 min, of which 12 min stalled |
| cost | 12 agent runs, **$3.3491**, all `claude_code` / `anthropic/claude-sonnet-5`, default execution option |
| outcome | `achieved: true`; report delivered to the request conversation; run resolved |

## 1. Which p2 problems recurred

| Problem | p2 | ex1 | ex2 |
|---|---|---|---|
| **C** — new conversation answered as empty | 2 of 2 | 0 of 16 | **did not recur** (0 of 2) |
| **B** — opening serving delegates for the run | 1 of 1 | 0 of 3 | **did not recur** (0 of 1); every delegation anchored to the run, every callback served it |
| **A** — retirement strands the anchor | once | staged | **not exercised** — nothing was retired |
| topic-name collision | — | 1 of 3 | **did not recur** — the run chose a new name beside four open ones |
| **stall with no wake signal** | once | not exercised | **recurred** — the task ended its session with a background download running and "will notify me on completion"; nothing does |
| resolved without the requester's agreement | once | not exercised | **did not recur** — the task stopped uncommitted, asked, and committed on the run's approval |

Also exercised for the first time on this code: an autolab mission end to
end, and a `routine_run` callback whose content lived only in `threads/`.
The run opened the thread rather than answering from the chatlog.

The three ex1 repairs that this path reaches held. **The stall is the one
open defect this run demonstrated**, and it is not one ex1 touched: a task
can start a process that outlives its session, and nothing turns that
process's end into a post. The agent did say what it was waiting for; the
thing it expected to wake it does not exist. Evidence in
[report3](report3.md).

## 2. What the researcher chose, and how it was grounded

It **read before choosing** — `README_PROJECT.md`, `main/README.md`, both
method files and the report, named by path in its plan — and chose the step
**both earlier method files and the report had named as deliberately not
done**: itemized, transaction-level donors for the same 35 senators. It
listed the alternatives it did not take and why.

The step is **the same slice one layer deeper again** (totals →
itemized/unitemized → transactions), with a new record level and a new
candidate-to-committee join, and two new questions asked of it: where
itemized money comes from, and from which employers and occupations. It
**reused** the roster and the access method with a stated reason,
**repeated** the human-dependency check instead of assuming it, set and
argued a privacy boundary, and **built a bridge** to run 2: transaction
totals reconciled against run 2's itemized figures.

The analysis is real rather than a listing. Finding 8 shows a raw party
comparison is an artefact of a placeholder occupation and reverses once it
is removed. Observation, interpretation and uncertainty stay separate.

**The independent checks reproduced almost every figure** from the raw FEC
files: linkage, transaction count, the +2.4 % reconciliation, the outliers,
state shares, the in-state means, the occupation shares
([report4](report4.md)). They also found:

- **a definitional error with consequences:** the method calls FEC type
  `22Y` an earmarked receipt; FEC defines it as a *contribution refund*.
  Excluding refunds brings the three "open, unresolved" outliers to −0.2 %,
  −5.2 % and −2.5 %. The mystery the report hands to a later run has a
  one-line answer, and refunds are counted as giving in Findings 7 and 8
  (small in effect at the report's rounding);
- **an internal contradiction:** the reconciliation paragraph says
  Republicans have the higher in-state share; Finding 7's own numbers say
  Democrats do;
- **a phantom example** ("King/ME") and a **blank field reported as a
  category** ("UNKNOWN");
- two minor figures that do not reproduce as documented (18 vs 20 within
  ±3 %; the placeholder-excluded employer shares).

None of these was caught by the task's review, by Front's approval or at
the run's end. Front praised exactly the reconciliation handling that is
wrong: it judged the summary, and the error is only in the files.

It asked the developer nothing about direction. What it left for later —
outside spending, the other senators, the House — is stated in the report.

## 3. Was it autonomous

**No — one intervention**, against p2's seven and ex1's one. The run was
**helped and finished**:

| # | What | Why |
|---|---|---|
| 1 | The Developer told the task, in its own topic, that its download had finished (6644), with the developer's approval before posting | nothing would ever have served it again |

> **Deus Ex Machina note:** *did the "your background job has finished"
> wake-up for agent autolab — handoff candidate.*

Everything else ran without help, including the two approvals and the end.
One further waste, not on the list: one redundant run serving ($0.11)
triggered a second after the listener marked the same topic served.

**What one run can say:** B, C and the collision did not recur on this
path, once each. **What it cannot:** that they will not — one observation is
not a rate — nor anything about A, which this path never reached. The stall
recurred on its first opportunity, which is stronger evidence than a clean
row: the condition that produces it (a large download inside a task) is
ordinary for a study, and the next run that meets it will stall the same way.

## Follow-ups at their owners (not done here)

- **autolab / pyagag — the stall.** A task that ends its serving with work
  still running has nobody to wake it. Either the task is told that a
  session's background processes end with it and nothing notifies anyone
  (guidance, measured next run), or a finished background job becomes a
  post in its topic — the ComfyUI notifier already does exactly that for
  one kind of job. The evidence says which gap; it does not yet say which
  fix.
- **studyuspolitics — the research content.** The `22Y` reading, the
  in-state contradiction, the phantom example and the blank-as-"UNKNOWN"
  label are the researcher's to correct, in a later run or a `publish` gate.
  They should be corrected before `84bbd91`'s material is published. The
  publish routine's four checks would not catch any of them.
- **Front's review of results.** Its approvals are of what it is told. An
  independent check on arithmetic and definitions is currently only
  available from outside the system.

## Left as it was

The routine stays available on guide v2. `publish/` is untouched at
`d467141` and its push is still the developer's. Five `workplan-` topics in
`#pj-studyuspolitics` are unresolved, now including this run's. Raw
evidence is in the ignored `.local/` beside this plan.
