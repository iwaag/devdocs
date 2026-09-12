# routine_tests p2 step 3 (second run) — what the second run read, reused and changed

Date: 2026-09-13 JST (timestamps UTC). Plan: [plan.md](plan.md) step 3, second
half, assessed against step 4's *Reuse and improvement* question.
Previous: [report3.md](report3.md).

## The request

`#front` › `front-uspolitics-20260912T1537Z`, message **6435**, then
re-stated as **6438**. It said the project already holds methods and a
report, and that continuing from them is the researcher's job — and then
deliberately said nothing about what to investigate:

> What to investigate this time, and whether anything about the existing
> method should change, are the researcher's own decisions, not mine.

No method, no improvement, no source and no target was supplied. That is
what makes the answer to "what did the second run read from the first?"
worth anything.

## Timeline

| UTC | What |
|---|---|
| 16:21:22 | Front served the request — **and answered "I don't see a message or request yet"**, exactly as on the first run |
| 16:22:20 | Request re-stated (6438) — **intervention 4** |
| 16:23:00 | Run opened: `routinerun-2026-09-12T1622Z`, message **6441**, **guide 6393 (v2)** |
| 16:24:04 | Delegated to a **fresh** topic, `workplan-continue-uspolitics-contribution-study` |
| 16:25:41 | Plan for mission **m6448**; Front resumed correctly from the mention |
| 16:26:19 | Front approved and started task1 itself (6458) |
| 16:26–16:32 | Source verification, join, two concurrent analysis subagents, report, commit |
| 16:32:48 | task1 complete; **Front resumed from the mention without help** |
| 16:33:24 | Run finished, `achieved: true`, report delivered, run topic ✔ |

About 12 minutes end to end, against 46 for the first run.

## What the second run read

It read `README_PROJECT.md`, `main/README.md`, and everything in
`main/methods/` and `main/reports/` before planning anything, and said so
concretely — naming commit `d19a5fb`, the 35-candidate roster, the source,
and the four findings.

**It then reused, explicitly and with a reason.** The plan states that the
roster, the source-composition method and the caveats — non-uniform
`CVG_END_DT`, `candidate_inactive` as a bookkeeping flag rather than a live
check, the nine-candidate inactive subset — are *"sound and still current
for this cycle — nothing here needs to be redone, only reused as the base
for the next slice."*

That is the behaviour the guide asks for and the plan warns against faking:
retaining a working method with evidence, rather than changing it so that
something can be called an improvement. Nothing in the first run's method
was rewritten.

## What it chose to add, and why that choice is the interesting part

It did not pick a new topic. **It picked the first run's own stated
weakness.**

Finding #2 of the first report claimed a "small-dollar/grassroots financing"
pattern among 2026 Democratic campaigns and then flagged, in its own
uncertainty paragraph, that the aggregate `TTL_INDIV_CONTRIB` field cannot
distinguish large donors from small ones — so the claim was an
interpretation the data could not settle.

The second run went and settled it: FEC publishes a per-candidate split of
individual contributions into **itemized** (aggregate per-donor giving over
$200) and **unitemized** (at or under $200). Same 35-candidate population,
same aggregate committee level, one layer finer.

It also said plainly what it was *not* doing: the raw itemized-transaction
file is multi-gigabyte and *"left for a later run to scope deliberately, not
folded in here as an afterthought"*. Donor names, employers and geography
stay out of scope.

## The plan was wrong about the file, and the run handled it as planned

The plan assumed the split lived in `webl26.zip`, and wrote its own escape
hatch: if the real layout differs, *"the task documents what the file
actually contains and adjusts the analysis to whatever real split it offers,
rather than forcing the original framing onto a file that doesn't support
it."*

On contact it did not. The task checked FEC's published field lists, found
`webl26.zip` does not carry the split, located the **Candidate Summary
file** (`candidate_summary_2026.csv`, nightly-updated public CSV, 302
redirect to public storage, no authentication) and used that instead. The
correction is recorded in both the method note and the report, rather than
being quietly papered over.

The human-dependency check ran again and again found nothing to raise — the
replacement source is also keyless.

## What it found

A new method file, `methods/fec-senate-2026-itemized-unitemized-individual-contributions.md`
(175 lines), and finding #5 appended to the existing report (133 lines
added, 6 removed — the scope paragraph was updated rather than replaced).

- Mean unitemized share of individual contributions: **31.2% (DEM, n=9) vs.
  12.4% (REP, n=16)**; medians 31.9% vs. 10.4%.
- Dollar-pooled: 50.1% vs. 15.5%; $54.9M vs. $7.9M of unitemized money,
  with Republicans holding nearly twice as many candidates in the subset.
- The two highest unitemized shares are Booker (60.8%) and Ossoff (57.7%) —
  Ossoff's on a $67.8M individual-money base, so a large base of small
  donors rather than a high share on a small one.

The conclusion it draws is careful in a way worth quoting against the plan's
"cosmetic improvement" concern: the earlier grassroots reading is
*"strengthened rather than undermined by this finer split"* — but it is
*"a group tendency, not a uniform rule"*, and it names the counterexamples
in both directions (Reed 1.4% and Coons 3.0% at the bottom of the whole
distribution; Moody 30.9% and Husted 29.4% above several Democrats). The
uncertainty paragraph notes that the pooled Democratic figure is inflated by
Ossoff alone contributing 71% of the Democratic unitemized total, and points
to the median as the more robust comparison. Both subagents converged on
"gradient, not a clean party split" independently.

**Missing data stayed missing.** The split exists for only 26 of 35
candidates. The report states this up front, states that the missing nine
are *exactly* the nine already flagged `candidate_inactive` — verified as a
real absence by `CAND_ID`, not a name-matching artefact — and says every
statistic covers only those 26, *"a structural gap in the source, not an
analytical choice"*.

## Verified independently

Recomputed here from the raw `candidate_summary_2026.csv` (4,439 rows) and
the raw `weball26.txt` from the first run, joined against the 35-candidate
roster — not from the agent's derived JSON:

| Claim | Recomputed here | |
|---|---|---|
| 26 of 35 matched, 9 absent | 26 matched, 9 missing | ✓ |
| the 9 absent are the `candidate_inactive` set | same 9 ids | ✓ |
| itemized + unitemized = the first run's `TTL_INDIV_CONTRIB` | **26 match, 0 mismatch** | ✓ |
| mean unitemized share DEM 31.2% / REP 12.4% | 31.2% (n=9) / 12.4% (n=16) | ✓ |
| highest shares Booker 60.8%, Ossoff 57.7% | 60.8%, 57.7% | ✓ |

The third row is the one that matters most: the second run's new source
**cross-checks the first run's numbers from a different file**, and they
agree to the cent for every candidate present in both. That is an
improvement in reliability with evidence behind it, not a claim of one.

Commit **`78a4148`** — *"Add itemized/unitemized individual-contribution
split for 2026 Senate slice"* — pushed to `origin/main`. Working tree clean.
`publish/` still at `569a8e3`, untouched.

## Autonomy: better, but not clean

**The run itself needed no help.** Front opened it, delegated it, approved
and started the task, was resumed by autolab's callback, judged the result
and ended the run — all without intervention. The `continue_deliveries`
handoff fired again afterwards.

Two things went right that had gone wrong before, and both are diagnostic:

- **A fresh delegation topic.** Front opened
  `workplan-continue-uspolitics-contribution-study` rather than reusing the
  first run's workplan topic. Reusing it would have misrouted every callback
  to the finished run.
- **The anchor held.** autolab's completion mention resumed the run
  correctly — `mention … serves routine-study-uspolitics/routinerun-2026-09-12T1622Z`
  — because no plan was retired this time. This is the counter-observation
  that confirms report3's defect: the anchor is lost specifically on the
  **retirement** path, not on the ordinary one.

**But the entrance failed again, and that makes it a defect rather than a
slip.** Front's first serving of a brand-new `front-*` conversation answered
*"I don't see a message or request yet in this conversation"* — the second
time out of two:

| | run 1 request | run 2 request |
|---|---|---|
| record | `front` run-0594 | `front` run-0597 |
| `num_turns` | **1** | **1** |
| duration | 2.25 s | 4.82 s |
| `chatlog.md` held the request | yes, verbatim | yes, verbatim |
| answer | "I don't see a message" | "I don't see a message" |

The successful servings that followed ran 8 and 10 turns. Both failures used
**no tools at all** and both generation workspaces were correctly built
(`chatlog.md` plus `tools/agents.md`). So this is not a delivery fault: on
the first serving of a new conversation, Front answers as if the chatlog
were empty. Re-stating the request in the same conversation fixes it every
time, at the cost of one wasted paid run each ($0.0511 and $0.0307).

Two for two is a reproduction, not a coincidence. Owner: `agfront`'s `front`
role and its guide. Left for a follow-up with its evidence, per the plan's
"repair only demonstrated defects at their owner".

## Backends and cost

All `claude_code` on `anthropic/claude-sonnet-5`, `exec_source: default`.

| Agent | Runs | Cost |
|---|---|---|
| Front (`front` ×3, `routine_run` ×3) | 6 | $0.9503 |
| autolab (`superdirector` ×1, `supercoder` ×1) | 2 | $1.6893 |
| **Second run total** | **8** | **$2.6396** |

The whole research task was one `supercoder` run — 144.9 s, 16 turns,
$1.4590 — including the source correction, the join, two concurrent
subagents, the method note, the report edit and the commit.

Running total for the episode so far: **22 agent runs, $8.4847**.

## Step 3 (second run) conclusions, kept apart

1. **The second run read the first and reused it deliberately.** It named
   the commit, kept the roster, method and caveats unchanged with a stated
   reason, and did not rewrite working material to look busy.
2. **The improvement was chosen from the first run's own admitted limit**,
   not from anything the evaluator supplied, and it is an improvement that
   was *exercised*, not merely proposed.
3. **Reliability improved with evidence.** A second, independent FEC file
   reproduces the first run's individual-contribution totals exactly for all
   26 candidates present in both.
4. **A wrong assumption in the plan was caught and corrected in flight**, by
   the fallback the plan itself wrote, and the correction is recorded in the
   committed material.
5. **The run needed no intervention once it started** — but the request
   still needed restating, for the second time out of two.
6. **Two runs are not a reliability result.** What they establish is that
   accumulation worked once, between one pair of runs, on one subject.
