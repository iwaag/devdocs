# routine_tests p2 — a study routine that developed its own methods

Date: 2026-09-13 JST. Steps: [report1](report1.md) … [report5](report5.md).
Source: [branindump.md](branindump.md). Plan: [plan.md](plan.md).

The braindump asked for a study routine given only a goal and a way to
accumulate knowledge — no fixed research method — to see whether it would
design its own and improve it. It ran twice, published once, and cost
**$11.4654 across 32 agent runs**, every one of them `claude_code` on
`anthropic/claude-sonnet-5` at the default execution option.

## 1. The routine exists and is discoverable

`#routine-study-uspolitics` — stream 154, in the existing `routine` channel
folder, with Developer, Front and Opsroom Observer subscribed, the same
three as every other routine. It shows on the ordinary routine board
alongside the other four, `retired: false`, with two runs and none open.

Two guide versions, both full posts rather than edits, per the routine
contract:

| version | message | what |
|---|---|---|
| v1 | **6357** | the braindump's instruction verbatim as a block quote, plus operational context only |
| v2 | **6393** | v1 word for word, plus one added bullet (below) |

The guide deliberately chooses no data provider, population, period,
measure, report shape or research order, and says so in its own text.
Plan step 4's evaluation questions were kept out of it — they are the
observer's, not the researcher's.

**The one rule added between the runs**, at the developer's request, after
the first attempt met a hard external limit and neither asked nor stopped:

> **When something needs a human, say so — do not route around it.** Some
> work is not yours to do: obtaining an API key or an account, accepting
> terms of use, paying for access, anything needing a human's identity or
> authorization. When you meet one, either build it into the research plan
> as a step a human performs, or ask for it there and then. Do not silently
> substitute a degraded route for it, and do not stall waiting for it
> without asking.

## 2. Contribution collection and useful analysis were actually performed

Verified here against the raw source files, not against either agent's
report. Full check tables in [report3](report3.md) and [report4](report4.md).

**Run 1** — 35 sitting US Senators whose seats are contested in the 2026
cycle (33 regular Class II + 2 specials), committee-level aggregate totals
from FEC's public bulk "All Candidates" file. Recomputed independently from
the raw 4,294-row file: combined receipts $279,765,279.94, median
$4,707,959.82, Ossoff (GA) $77,279,766.48, one self-funder (Ricketts,
47.1%), five candidates carrying debt all under $75K, all 35 roster ids
matched with no misses, `CVG_END_DT` spanning 06/30–08/26/2026. Every
figure agreed.

**Run 2** — the same population, one layer finer: individual contributions
split into itemized (over $200) and unitemized (at or under $200).
Recomputed independently: 26 of 35 matched with 9 absent, the 9 being
exactly the already-flagged `candidate_inactive` set; mean unitemized share
31.2% (DEM, n=9) against 12.4% (REP, n=16); highest shares Booker 60.8% and
Ossoff 57.7%. And the check that matters most — **itemized + unitemized
equals run 1's `TTL_INDIV_CONTRIB` from a different file, for all 26
candidates, with no mismatch.**

Against plan step 4's questions: source, retrieval time, period, population
and coverage limits are all identifiable from `main/methods/` alone and the
retrieval is repeatable from it; observation, interpretation and uncertainty
are separated per finding; the reports name their own outlier problem (one
candidate contributing 71% of a group total) and say where categories do
not sum to 100% of receipts. Missing data stayed missing: the nine absent
rows are recorded as a structural gap in the source, never as zeros.

**A narrowed first sample with stated boundaries — not coverage of
Congress.** Both reports say so: no donor-level records, no House, not the
other ~65 sitting senators, no leadership PACs, joint fundraising
committees or outside spending.

## 3. A later run reused and assessed accumulated methods

Run 2 was asked for with no method, no target and no improvement supplied —
only "continue from what is already there."

It read `README_PROJECT.md`, `main/README.md` and everything under
`methods/` and `reports/` first, named commit `d19a5fb`, and **kept the
roster, the source-composition method and the caveats unchanged with a
stated reason**: *"sound and still current for this cycle — nothing here
needs to be redone, only reused as the base for the next slice."* That is
the guide's preference — retaining a working method with evidence over
changing one so something can be called an improvement — actually observed.

Then it chose its new work from **the first report's own admitted limit**.
Finding #2 had claimed a small-dollar pattern and flagged in its own
uncertainty paragraph that the aggregate field cannot separate large donors
from small ones. Run 2 went and settled that, and reported the answer as a
group tendency rather than a rule, naming counterexamples in both
directions.

It also caught its own plan being wrong. The plan assumed the split lived in
`webl26.zip`; on contact it did not, and the fallback the plan had written
for exactly that case fired — the real source was located, used, and the
correction recorded in the committed material rather than papered over.

**Proposed versus exercised:** the improvement was exercised. The
reliability gain is evidenced by the cross-source agreement above, not
asserted.

## 4. Publication was prepared; remote publication has not happened

`publish` ran once for `studyuspolitics` only, under guide v2 (6254).

The project is **not** in that guide's example table, and was discovered
anyway from the workspace and `README_PROJECT.md` — the p1 defect where a
stale table narrowed coverage did not recur.

All three candidate files failed **check 4** (internal-workflow residue:
subagent-dispatch narration, retired-attempt framing) with the related
check-1 fix (gitignored local paths generalized). Fixed in `main/` first,
then copied unchanged. Material with evidentiary weight was rewritten into
the findings prose rather than deleted. No provenance or quote-hygiene
failures, and no paper-specific version line or local-test requirement was
invented for a political report.

| Repository | Commit | State |
|---|---|---|
| `main/` — `autodev/studyuspolitics` (internal Gitea) | `d817882` → `d19a5fb` → `78a4148` → **`ac2f7d2`** | all pushed |
| `publish/` — `github.com/iwaag/study-uspolitics` | **`d467141`** on top of the pre-existing CC0 `569a8e3` | **committed locally, not pushed** |

Verified here: the three published files are byte-identical to `main/`; the
supplied repository's existing history is intact and was never reset or
reconciled; the clone is one commit ahead of its origin and stays that way;
`publish/README.md` is written for a stranger with no mention of `main/`,
routines, missions or internal repositories; the published tree contains no
markdown links at all, so none can dangle. Four `DEMO_KEY` mentions remain
and are not residue — that is FEC's own public shared key, named in the
explanation of why the keyless bulk route was chosen.

**The remote push is outstanding and is the developer's.** The supplied
GitHub URL identified the destination; it did not override the contract's
developer-owned final push.

## 5. Autonomy and reliability — what two trials can and cannot establish

**Seven interventions.** This was an assisted episode and calling it
autonomous would be false.

| # | Step | What | Why |
|---|---|---|---|
| 1 | 3 | Re-stated the run-1 request | Front's first serving answered "I don't see a message" |
| 2 | 3 | Scrapped mission m6371 by hand | the stalled attempt had no wake signal; nothing in the system could end it |
| 3 | 3 | Restored Front's run anchor (6423) and named it (6424) | retirement had moved the anchor into the renamed topic |
| 4 | 3 | Re-stated the run-2 request | same empty first serving |
| 5 | 5 | Answered Front's "shall I start?" (6501) | ordinary, but only needed because of defect B |
| 6 | 5 | Wrote the run's anchor into the publish delegation (6500) | **did not take effect** |
| 7 | 5 | Served the publish run directly (6516) | the only way left to let it finish |

> **Deus Ex Machina notes:** *did the workspace pattern marker and the
> project channel creation for agent autolab; did the run-topic
> origin-anchor repair for agent Front after a plan retirement moved it
> aside; served the publish run directly and repaired its delegation anchor
> for agent Front — handoff candidates.*

**What ran unassisted.** Run 2's whole execution: Front opened it, delegated
to a *fresh* topic (avoiding the known callback-misroute trap), approved and
started the task itself, was resumed by autolab's callback, judged the
result and ended the run. The `continue_deliveries` handoff from p1 fired
after both study runs.

### Three demonstrated defects, none repaired here

**A. Retiring a plan strands the supervising run's anchor.** Retirement
renames the whole conversation aside, taking Front's `[selfnote][rootchat]`
with it; the replacement mission carries no anchor, so autolab names Front
correctly and Front correctly refuses — `carries no root note of ours`.
Complete and silent, with neither side at fault. Found via an intervention,
but not caused by one. ([report3](report3.md))

**B. A run that delegates in the serving that opens it anchors the
delegation to the Front Desk.** The `front` role writes the note naming its
own conversation, so the `routine_run` role that drives the run is never
told anything: the plan and the completion both went to the Desk, the task
was never started until a human answered, and the run wrote no entries and
no finish block. **This one arose with no intervention at all.** Adding a
second anchor does not fix it — the first note keeps winning the lookup.
([report5](report5.md))

**C. Front's first serving of a new conversation can answer as if it were
empty.** Twice out of the first two requests, and then *not* on the third —
so it is intermittent, not deterministic. Both failures ran one turn with no
tool calls while the correct `chatlog.md` sat in the workspace; the
successful servings ran 8–20 turns. Not a delivery fault. Cost: one wasted
paid run each.

A and B are the same family — a routing anchor that does not survive a
change of context — and per the developer's decision their root cause is a
later phase's work, not this trial's.

### What is proven, and what is not

- **Proven:** the routine exists and is reachable; two runs collected real
  public contribution data and analysed it, with figures independently
  reproduced from the raw sources; the second run read, reused and extended
  the first through its own accumulated methods; the publication gate found
  a project absent from its own table and staged it correctly.
- **Measured once, not generalized:** the guide rule added between the runs
  produced a dependency check that changed the chosen source from a
  rate-limited keyed API to a keyless bulk download. One trial.
- **Not proven:** autonomous operation — seven interventions, two of them
  for failures the system cannot recover from by itself. Long-term
  reliability — two runs, one subject, one afternoon, is evidence of
  repeatability and nothing more. Whether the routine holds up on a
  different question, a stale source, or a genuinely absent update remains
  untested.
- **Not manufactured:** no source outage was staged. The FEC rate limit was
  a real external constraint met by a real attempt, and the run that hit it
  is reported as the failure it was rather than smoothed over.

## Left behind

- **The routine stays available**, its guide not retired, for future
  requested runs.
- **`publish/` awaits the developer's manual push** — `d467141`, one commit
  ahead of `https://github.com/iwaag/study-uspolitics.git`.
- **Four `workplan-` topics in `#pj-studyuspolitics` are unresolved**
  (setup, the two study missions, publish). autolab does not tidy on its
  own — that is contract, not oversight — and they are recorded here rather
  than closed.
- **`README_PROJECT.md` is tracked nowhere**, since the workspace root sits
  outside any repository. True of every study project, so a property of the
  pattern rather than a fault of this one, but it means a later serving
  depends on a file with no backup.
- **Three defects are open**, A and B assigned to a later phase by the
  developer, C recorded with its evidence.
