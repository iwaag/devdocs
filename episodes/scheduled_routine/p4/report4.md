# p4 step 4 — the remaining logical days

**Scope changed by the Developer mid-phase.** After logical day 1 the
Developer asked whether all seven days were intended and said about two more
would be enough. Steps 3–4 therefore cover **three logical days
(2026-08-26, 08-27, 08-28)**, not seven. Days 4–7 (`e27`–`e30`, `e34`–`e37`)
remain written and unfired. What three days cannot show is stated at the end.

Day 1 is in `report3.md`; this report covers days 2 and 3 and the whole run
together.

## The table

| day | paper (id) | signal | runnable | manual written | Front runs | autolab tasks | Developer replies | failures |
|---|---|---|---|---:|---:|---:|---:|---|
| 1 · 08-26 | 2608.21156 Graph Engineering survey | HF Daily Papers 08-24, #1/22 | no | — | 5 | 2 | 0 | upvote count wrong (50 vs 38) |
| 2 · 08-27 | 2608.23283 Apodex 1.1 | HF Daily Papers 08-25, #1 | **yes** | **2** (this paper + 2608.23552) | 12 | 5 | **1** | three wrong figures; Front asked permission; stray duplicate topic |
| 3 · 08-28 | 2608.20430 RISE | HF Daily Papers 08-25, #2 | no | — | 5 | 2 | 0 | stray duplicate topic |

Trial run (Step 1, real clock): 2608.23552 Prime Agent, HF Daily Papers 08-25
#4, `runnable: yes`, 5 Front runs, 2 autolab servings, 0 replies.

Totals across the trial + three logical days: **4 papers, 2 manuals,
27 Front runs, 11 autolab servings, 1 Developer reply.**

Day 2 is the expensive day because it is the only one that ran the full
A → decide → B chain, and B was two tasks.

## Duplicate avoidance — held, 4 for 4

Four runs, four distinct arXiv ids. Every run read `papers/INDEX.md` first and
said so. Day 3 went further and named the exclusions in its own summary:

> ranked #2 on the HF Daily Papers trending list with 20 likes, directly below
> Apodex 1.1 (**already indexed**)

and Front's decide recorded "excluding the three already-indexed papers". The
guard is not merely obeyed, it is visibly reasoned about. Note that days 2 and
3 drew from the *same* HF list (2026-08-25) and still did not collide — the
INDEX, not the day boundary, is what separated them.

## Signal drift — none, but the slice moved

All four runs used **HF Daily Papers**. No run tried Papers with Code or
Semantic Scholar. The slice within the signal wandered freely:

| run | list date | rank taken |
|---|---|---|
| trial | 2026-08-25 | #4 |
| day 1 | 2026-08-24 | #1 |
| day 2 | 2026-08-25 | #1 |
| day 3 | 2026-08-25 | #2 |

So "which signal" is stable and "which day's list, and how far down it" is not.
The standing text constrains neither, and nothing went wrong because of it.

## The trend evidence is unreliable — the central finding

Every summary names its signal and a URL, as the standing text demands. Two of
the four published numbers that the named source does not contain.

| run | fetch method | claimed | actual | verdict |
|---|---|---|---|---|
| trial | WebFetch, **then re-fetched the JSON API** | #4 of 12, 12 upvotes | #4 of 12, 12 | exact |
| day 1 | WebFetch (HTML page) | #1 of 22, **50 upvotes** | #1 of 22, **38** | **wrong** |
| day 2 | WebFetch (HTML page) | #1, **219 upvotes**; RISE **20**; Prime Agent **18.2k** | 119; 19; **12** | **wrong ×3** |
| day 3 | WebFetch (HTML page) | #2, 20 likes | #2, 20 | exact |

Checked by the Omni Agent against `huggingface.co/api/daily_papers` and against
the embedded payload of the very page each run named. Upvote counts only rise,
so none of these is staleness — day 1's page carried `"upvotes":38` at the time
and carries 38 now.

Three things make this the finding of the phase:

1. **It is not deterministic.** Day 3 used exactly the method that failed on
   days 1 and 2 and was perfect. Half the runs are right, half are wrong, and
   **nothing in the text distinguishes them** — every one is specific, sourced,
   and confident. A reader cannot tell without re-fetching.
2. **The 18.2k figure is traceable.** `PrimeIntellect-ai/prime-agent` has
   **18,199 GitHub stars** — verified in Step 1 — i.e. "18.2k". The model is
   substituting a star count for an upvote count. The *same* "18.2k upvotes"
   appeared in the trial run, where the agent recognised it as a hallucination
   and rejected it; two runs later it reappeared and was committed.
3. **It propagates into decisions.** Front's `e32` decide repeats "#1 on HF
   Daily Papers with 219 upvotes" as the evidence for adding a `manual` fire.
   Front cannot fetch, so it can only relay. The decision was still right — it
   turned on `runnable: yes`, not on the count — but a wrong figure is already
   being used as scheduling justification.

Nothing in the system caught either case. The standing text was obeyed. Front's
approval could not check it. The close-out does not look at content. Only the
Omni Agent re-fetching afterwards found it, and only because the summaries were
honest enough to name their URLs.

**Candidate, with two occurrences behind it:** require the *raw* source — a
JSON/API endpoint, not a rendered page — and quote the figure from it. The one
run that did this was exact. That is one data point, not proof, so the honest
form of the guidance is "fetch the machine-readable endpoint and quote it",
tested next phase rather than assumed.

## The Developer's two readings

The Developer (Omni Agent on the Developer's behalf) read two summaries in full.

**2608.23283 Apodex 1.1 — worth the repository.** Its `Local run` section and
the manual beside it are the kind of thing that saves a real afternoon: VRAM per
weight format (bf16 ~70 GB, FP8 ~36 GB, NVFP4/Int4 ~18–20 GB), the certified-GPU
matrix from `docs/install/gpu-compatibility.md` (Int4 at 32K on one RTX 5090;
RTX 4090 24 GB explicitly *not* certified), the three `config/sglang` env
templates by filename, and the documentation's own warning that naming a model
in `SGLANG_MODEL_ID` does not mean it fits. That is specific enough to act on
and specific enough to be wrong in a checkable way, which is what a digest
should be. Its "why it trends" paragraph, unfortunately, is the one with three
wrong numbers in it.

**2608.21156 Graph Engineering survey — worth it, and honest about being a
survey.** It does not pretend the paper has results it lacks ("As a survey, it
does not run its own benchmark; instead it aggregates evidence and resources"),
and its `runnable: no` is argued rather than asserted: it fetched the companion
repo and established that it is a bibliography, not code. A digest that
correctly refuses to make a reading list sound runnable is doing its job.

**Verdict: yes, these are worth the repository** — the paper choice, the
reading, and the runnability judgement were sound in all four runs. The defect
is confined to the trend figures, which is exactly the part a reader is least
able to check and most likely to quote.

## Interventions

**One.** Logical day 2, routine B's first run: Front stopped and asked

> Can I proceed and post the start messages into both workrun topics?

although `routine-manual`'s standing text already says *"Approving means acting
— start it and check it through to Done, ask nothing"* — the same sentence that
`routine-papers` has honoured across four runs. The Developer answered "yes,
proceed" and restated the rule in the topic.

Cause: a new routine's first run does not inherit the other routine's
demonstrated behaviour. p2 added this line after one occurrence and it worked
*for that routine*. The line is in B's text and B still asked, so the fix is not
another line — this is evidence that first-run caution is per-routine and that
the cheapest remedy is the Developer's one-word answer, not more standing text.

## Failures and near-misses

- **The stray duplicate `workrun-` topic, twice** (`work-p7-5`, `work-p7-10`).
  Front posted "Start task1 as planned" into a hand-made topic with the same
  name as one autolab had opened, after the real one had been resolved and
  renamed with `✔ `. p2's rename-blind read, recurring in p4.
  **It cost nothing**: autolab's p9 binding check answered "not bound to any
  task" and ran nothing, and Front noticed, explained it, and did not loop —
  where p2's Front looped and triggered a re-plan. The trap is unfixed; the
  damage from it is now zero, and that is the p9 anchor design working.
- **The production dispatcher died twice** (04:47Z, 04:52Z) with
  `cannot pull with rebase: You have unstaged changes` — an uncommitted
  `index.html` left in the schedule clone by the Omni Agent. The whole
  scheduler stops on any dirty file in that clone. `rtschedule` has a
  deliberate dirty-clone guard; `dispatch.py` just raises. Recovered by itself
  once the file was committed.
- **The logical date never leaves the dispatcher.** All four INDEX rows read
  `2026-08-25`; three of them were logically the 26th, 27th and 28th. Front's
  topics are `workplan-papers-20260825b/c/d`. `trigger.sh` stamps
  `date -u`, so nothing downstream can know the logical day. Over an
  accelerated week the `date` column — the first column of the digest — would
  be uniformly wrong.
- **Zero listener restarts, zero stalls, zero task failures.** No run had to be
  retried and no work was lost.

## What three days could not measure

- **Duplicate avoidance across a whole week.** Four papers is a weak test of a
  guard that has to hold at fifty. Days 2 and 3 drawing from the same list and
  not colliding is the strongest evidence here, and it is still only four rows.
- **Whether the signal eventually drifts.** Three days of HF Daily Papers shows
  no drift; it does not show stability.
- **Whether routine B ever says "nothing to do".** B ran once, with two papers
  waiting. The "if there is nothing to do, say so" branch of its standing text
  was never exercised — day 3 was `runnable: no`, so no B fire was added.
- **Whether the first-run permission ask recurs.** B ran once. One occurrence
  cannot say whether the Developer's answer fixed it.
- **Cost drift over a real week**, and everything about real timing — see the
  phase report.
