# routine_tests p2 ex2 step 4 — the two questions, evaluated separately

Date: 2026-09-13 JST. Plan: [plan.md](plan.md) step 4.
Previous: [report3.md](report3.md).

## 4a. Did the same problems recur?

| Problem | p2 | ex1 | **ex2** | Evidence |
|---|---|---|---|---|
| **C** — first serving answers as if empty | 2 of 2 | 0 of 16 | **0 of 2** servings of the new `front-*` conversation | run-0615: 6 turns, opened the run (6618/6619); run-0616: the continuation, 1 turn, summarised the delivered report correctly (6671) |
| **B** — opening serving delegates for the run | 1 of 1 | 0 of 3 | **0 of 1** | no post in `#pj-studyuspolitics` during run-0615; delegation root note 6621 and task root note 6634 both name `routine-study-uspolitics/routinerun-2026-09-13T1853Z`; all four callbacks logged `serves routine-study-uspolitics/routinerun-…` |
| **A** — retirement strands the anchor | once | staged, recovered | **not exercised** | autolab retired nothing; no `[replaces]` note |
| topic-name collision | — | 1 of 3 | **0 of 1** | `workplan-next-uspolitics-slice` differs from the four open `workplan-` topics |
| stall with no wake signal | once | not exercised | **recurred, 1** | 18:59:22 task ends with a `nohup` download and *"will notify me on completion"* (6640); nothing served until intervention 6644 at 19:12:06 |
| task resolved without the requester's agreement | once | not exercised | **did not recur** | the task stopped uncommitted and asked (6650); Front approved (6652); the task committed, reported and only then resolved (6660–6663). autolab resolved its task topic before Front's final judgement (6667), but after Front's explicit approval of exactly that result |

Also exercised for the first time since ex1:

- **An autolab mission end to end on pyagag `ed65b4e`** — plan, start by
  the run, pause, completion, commit, resolution, delivery. It worked,
  given the one wake-up.
- **A real callback into `routine_run` with the plan only in `threads/`.**
  The run opened the thread (9 turns, entry 6637 quotes content only in
  it). ex1's one-turn "saw the chatlog, not the thread" did not happen.
  One observation.

**Interventions: 1** (p2: 7, ex1: 1). The run **was helped and finished**;
it did not need no help. Everything else — opening, delegating, approving,
starting, judging a pause as not done, approving the commit, ending the run
and delivering — ran unassisted.

One clean pass over B, C and the collision is one observation each, not a
reliability result. A was not on this path. The stall recurred on its first
opportunity.

## 4b. Which way did the researcher go?

Classified from the committed material (`84bbd91`) and the mission
conversation, not from the summaries.

**What it read first.** The plan (6626) opens with *What I found in the
project first*: `README_PROJECT.md`, `main/README.md`, both method files and
the report, named by path, with their slices, sources and match counts; it
names `ac2f7d2` and the earlier publish gate. The task session's tool log
(6639) shows the same reads before any fetch, plus run 2's scratch join.

**Grounding.** The choice is taken from the earlier material's own admitted
limit. The plan quotes it: both method files and the report's *What this
slice cannot tell you* named transaction-level itemized donors — who, from
where, which employer — as deliberately not done. The plan lists the
alternatives it did not take (outside spending, the other ~65 senators, the
House) and says why this one: it extends an existing population and method
rather than starting a new one, and answers a question the itemized/unitemized
finding raised but could not settle from totals.

**Kind of step.** Mostly **the same slice one layer deeper again** (the
third time: totals → itemized/unitemized → itemized transactions), with a
**new record level** (Schedule A transactions) and a **new join**
(candidate → principal committee, from the linkage file). Two new questions
asked of it: donor geography and employer/occupation. No change to the
population, no change to method *infrastructure* (no reusable script
committed — the filter and aggregations live in ignored scratch), no
tidying. Roughly: depth 60 %, new record level and join 30 %, new questions
10 %, method change 0 %.

**Reuse versus change.** The 35-candidate roster was reused as-is, with the
reason stated ("reused as-is, not re-derived"). The access pattern (check
FEC's file-description page, then bulk download, no key) was reused and the
human-dependency check was **repeated rather than assumed**. The existing
report was extended in place with Findings 7 and 8, and its *cannot tell
you* section rewritten to remove the limit it just addressed. The privacy
boundary (no donor-level records published) was chosen and argued, including
its own residual risk for thin employer categories.

**Reliability — a bridge was built.** Transaction-level itemized totals
were cross-checked against run 2's `individual_itemized_contribution` for
the 26 candidates that have one: +2.4 % in aggregate, three outliers named.
That is the right kind of step — the new layer is tied to the old one
rather than floating free. **But its explanation of the gap is wrong** (see
the checks below), and a correct explanation was one exclusion away.

**The report itself.** Findings 7 and 8 keep observation, interpretation and
uncertainty apart and name their own weak points (no null model against
population and wealth; unstandardised free-text employers; the "NOT
EMPLOYED" reading is an inference). Finding 8 is a real use of evidence: a
raw comparison is shown to be an artefact of a placeholder value, and the
corrected comparison reverses it. Missing data is mostly honest — the nine
candidates without a filed figure stay out of the reconciliation — with
one lapse: a **blank** occupation field is reported as a category named
"UNKNOWN (7.8 %)".

**What it said about direction.** The plan says why this step and what it
left (outside spending, other senators, the House, donor-level detail). The
report's rewritten closing lists the same as later extensions. It asked the
developer nothing about direction. Front's continuation offered "another run
or dig into the three reconciliation outliers" — recorded, not answered.

## Independent checks

Recomputed from the raw public files the method cites — `ccl26.zip` and the
full `indiv26.zip` (`itcont.txt`, read directly, not the researcher's
filtered copy) — and from run 2's join for the on-file totals. Scripts and
output in ignored evidence.

| Reported | Recomputed | |
|---|---|---|
| 35 of 35 candidates → exactly one principal Senate committee | 35 of 35, none with more than one | ✔ |
| 823,045 transactions under the 35 committees | 823,045 | ✔ |
| transaction total $102.4M vs on-file $100.0M, **+2.4 %** (26 candidates) | $102,418,119 vs $100,024,052, +2.39 % | ✔ |
| excluding `22Y`: $98.9M, −1.1 % | $98,886,233, −1.14 % | ✔ |
| Cornyn +31 %, Mullin +51 %, Lummis +94 % | +31.3 %, +50.6 %, +94.0 % | ✔ |
| Cornyn gap $1,107,320, `22Y` $1,127,810 | gap $1,107,320; `22Y` $1,114,010 in the rows the method sums, $1,127,810 only if memo rows are included | ✔ / minor |
| 18 of 26 within ±3 % | 20 of 26 | ✘ minor |
| top states CA 12.1, NY 9.5, FL 8.1, TX 6.6, VA 6.3, GA 6.0 — ~49 % | 12.1, 9.5, 8.1, 6.6, 6.3, 6.1 — 48.6 % | ✔ |
| 20 of 35 have CA or NY in their top 2 | 20 | ✔ |
| Collins, Lummis, McConnell: home state not in top 3, each led by FL | same three, each led by FL | ✔ |
| in-state mean 34.0 % DEM vs 24.6 % REP (median 27.8 / 21.6) | 34.0 / 24.6 (27.8 / 21.6), n = 13 / 21 | ✔ — Risch (ID) carries party `UNK` in the roster and is silently in neither group |
| occupations: NOT EMPLOYED 21.7 %, RETIRED 16.9 %, UNKNOWN 7.8 %, ATTORNEY 4.9 % | 21.7, 16.9, **blank field** 7.8, 4.9 | ✔ numbers / ✘ label |
| NOT EMPLOYED mean share 23.7 % DEM vs 0.5 % REP; RETIRED 7.1 vs 18.6 | 23.7 / 0.5; 7.1 / 18.6 | ✔ |
| "352,387 of 372,000" NOT EMPLOYED Democratic transactions conduit-routed | 352,387 are type `15E` (earmarked) of 371,578 | ✔ |
| top-10 occupation, raw 75.9 vs 64.6; excluding placeholders 20.3 vs 26.6 | 75.9 / 64.6; 20.3 / 26.6 | ✔ |
| top-10 employer excluding placeholders 5.6 % DEM vs 7.7 % REP | 11.4 / 12.6 with the method's own placeholder list | ✘ not reproducible as documented — the list omits `SELF EMPLOYED`, `SELF-EMPLOYED`, `SELF`, the 4th–6th largest employer values |

### The substantive error: `22Y` is a refund, not a receipt

The method calls `22Y` "earmarked/conduit-routed receipts" and concludes the
reconciliation gap is "open, investigated-but-unresolved": `22Y` is "neither
wholly inside nor wholly outside" FEC's itemized figure. FEC's own
transaction-type table defines **`22Y` as "Contribution refund to an
individual, partnership or limited liability company"**; the earmarked type
is `15E`. Refunds were summed as contributions.

Excluding them resolves the named outliers per candidate — Cornyn **−0.2 %**,
Mullin **−5.2 %**, Lummis **−2.5 %** — and puts 21 of 26 within ±3 %. The
method tried the aggregate only (−1.1 %), read that as over-correction, and
did not look per candidate. (Subtracting refunds instead goes to −4.7 %, so
FEC's candidate-summary itemized figure is gross of refunds; excluding them
is the matching treatment.)

Consequences for the published findings are **small but real**: 7,769 refund
rows are added to the dollars behind Findings 7 and 8 as if they were
giving, and the reconciliation note up front, the method's caveat section
and the report's reconciliation paragraph all rest on the wrong reading. The
checks above show the shares barely move at the level the report rounds to;
the interpretation of the gap, not the direction of the findings, is what
is wrong.

### Other defects in the committed text

- **An internal contradiction.** *Reconciling Findings 7 and 8* says
  Republican itemized dollars are more concentrated "geographically (higher
  in-state share, Finding 7)". Finding 7's own figures — and the recomputed
  ones — say the reverse: 34.0 % DEM vs 24.6 % REP.
- **A phantom example.** Finding 7 lists "King/ME's counterpart race" among
  candidates with CA or NY in their top two. King is not in the 35; Maine's
  candidate in the slice is Collins, who is in the home-state-absent group.
- **Missing presented as a category** — the blank occupation as "UNKNOWN".

None of these was caught by the task's own review, by Front's approval
(6652, which praised the reconciliation handling specifically) or by the
run's end. Front judged the relayed summary, and the errors are only in the
files — a supervisor that reads what it is told cannot see an arithmetic or
definitional mistake in material it never opens. The routine has no publication gate on this material yet; the
`publish` routine's four checks do not look at arithmetic.

## What the run cost

12 agent runs, **$3.3491**, all `claude_code` / `anthropic/claude-sonnet-5`,
`exec_source: default`.

| role | runs | cost |
|---|---|---|
| Front `front` | 0615 (open), 0616 (continuation) | $0.1382 |
| Front `routine_run` | 0053–0058 (six servings; 0057 the extra) | $1.0590 |
| autolab `superdirector` | 0198 (plan) | $0.3371 |
| autolab `supercoder` | 0268 (until the stall), 0269 (analysis), 0270 (commit) | $1.8148 |

For comparison, p2's two study runs and publication were $11.47 over 32
runs.
