# Step 5 — verifying the evidence, and the correction pass

Date: 2026-09-15 JST (timestamps UTC). Plan: [plan.md](plan.md) section 5.
Previous: [report4.md](report4.md).

The committed files were checked against their original sources, not
against Front's summary. Ten of ten central figures reproduced; every
derived ratio recomputed; two defects and one coverage gap were found; the
responsible researcher corrected them in `main/` in one small mission,
reusing its saved methods and extending them once. `main` is now at
**`fd28557`**, pushed. Raw check notes and the full `main` diff are in the
ignored `.local/` beside this report.

## Which checks, and why these

The plan asks for central quantitative findings and consequential
definitions. Chosen: the figures the synthesis leans on (EA take-private,
Tencent–Vantage, General Intuition, HoYoverse AI spend, Marvel Rivals
decline, Take-Two's AI layoffs), the largest reported financials in each
cluster (Sony, Nintendo, Tencent, Ubisoft, Embracer), one tracker figure
(InvestGame) and one layoff figure (Epic). Sources were reached by the
outlet names the reports give, since they carry no URLs; primary documents
were preferred where one exists.

## Source checks

| # | claim (`main` at `9f0da53`) | checked against | result |
|---|---|---|---|
| 1 | Sony FY2025 sales ¥12,479.6bn (+4 %), OP ¥1,447.5bn (+13 %), G&NS OP ¥463.3bn record | Sony results release (SEC 6-K), coverage | reproduced; +3.7 % / +13.4 % in the release, rounded in the report |
| 2 | Nintendo FY3/2026 net sales ¥2,313.1bn +98.6 %; Switch 2 19.86M; Mario Kart World 14.70M | Nintendo results explanatory material (IR PDF) | reproduced (¥2,313.0bn in the PDF) |
| 3 | Tencent FY2025 games RMB 241.6bn +22 %; domestic 164.2 +18 %; international 77.4 +33 % | Tencent 2025 annual results release | reproduced |
| 4 | EA: $210/share, ~$55B EV, ~$36B equity + $20B debt, closed 2026-08-04, PIF >93 % | EA completion release | reproduced (PIF 93.4 %) |
| 5 | Tencent–Vantage: €1.16B, **"25 % stake"**, closed 2025-11-21; **three dollar figures** across two files | Ubisoft closing press release PDF, read page by page | **€1.16bn for a 26.32 % economic interest**, pre-money EV €3.8bn. 25 % was the March 2025 announced figure. Defect. |
| 6 | Ubisoft FY25-26 net bookings €1.53B −17.4 %, operating loss €1.3B, headcount ~16,600 | Ubisoft FY26 release | reproduced (€1,525.1M vs €1,846.4M; 16,590) |
| 7 | Epic: 2026-03-24, >1,000 (~20 %), ~4,000 remain, $500M savings | Epic's own post, WRAL | reproduced |
| 8 | General Intuition $133.7M seed, $320M Series A at $2.3B, $454M total | GamesBeat, InvestGame; **TechCrunch 2026-08-24** | rounds reproduced; **a later round at a $6B valuation reported inside the window was missing** |
| 9 | Embracer FY25/26 net sales SEK 15,906M −25 %, adjusted EBIT −68 % | Embracer Q4/FY report page | reproduced (SEK 15.9bn; adj. EBIT SEK 905M) |
| 10 | InvestGame Q2 2025 $904.6M / 113 deals, −27.2 % / −17.5 % QoQ | InvestGame Q2 2025 report page | reproduced |
| 11 | Take-Two laid off its head of AI and team, April 2026 | Engadget, Game Developer | reproduced |
| 12 | Marvel Rivals ~85 % concurrent-player decline from peak by Sept 2026 | Steam trackers | reproduced (644k → ~90k, −86 %); the trackers are Steam-only and the report does not say so — minor |
| 13 | HoYoverse up to RMB 100bn / 3 years on AI, "reported, unconfirmed by the company" | esports.gg, Game World Observer | the CEO said it at a private briefing reported by Chinese media, no official release — the report's characterisation is fair |

**Arithmetic recomputed from the tree alone**, all agreeing: Switch 2
software attach 48.71 / 19.86 = 2.45; Steam titles reaching 1,000 reviews
608 / 20,282 = 3.0 %; Tencent overseas share 77.4 / 241.6 = 32 % and the
two segments sum to the total; Nintendo's +98.6 % implies a prior year of
¥1,164.7bn against an actual ¥1,164.9bn; Ubisoft's −17.4 % implies
€1.852bn against €1.846bn; InvestGame's deal count 113 / 137 = −17.5 %;
Epic's 1,000 of ~5,000 = 20 % leaving ~4,000.

**Definitions**: net bookings and net revenue are used as the companies
use them (EA, Take-Two, Ubisoft); fiscal-year naming follows each company
(Sony "FY2025" = year to March 2026; Nintendo "FY3/2026"); announced,
completed, funding, acquisition value and operating spend are labelled as
the guide asked. No wrong definition of the kind ex2 found (`22Y`) turned
up. There are no tables or charts in the reports, so prose-versus-table
agreement reduces to cross-file agreement, which is where the one
inconsistency was.

**A tool defect met on the way.** The fetch summariser returned a
confident, entirely wrong reading of the Ubisoft closing release ("$60M
for 10 %, expected to close H1 2024"). The PDF was read directly and the
summariser's answer discarded. Same lesson as study-realworld's FOMC
PDF: read the document, not the summary of it.

## The correction pass

Posted as the Developer at autolab's ordinary entrance — a **new**
workplan topic, `#pj-studyindustry` › `workplan-correct-videogame-pass1`,
message **7059** (10:32:09Z) — rather than through Front, because a
correction of named defects in named files is a project request, not a
run of the routine, and the routine's supervisor adds nothing to it.
The request listed the three findings, asked for one small mission
reusing the saved methods, a link to the run log from the index and
README, a dated entry in the run log, and a statement of which methods
were reused and whether `methods/` needed extending.

| | |
|---|---|
| mission | **m7061**, plan 7069, started on the request's pre-approval |
| task | `#work-m7061` › `workrun-task1-m7061`, started by the Developer at 7070 (11:12:42Z) |
| result | 7076 (11:15:09Z); `main` **`fd28557`** "Correct Vantage Studios stake, unify its dollar figure, add General Intuition's $6B round", pushed |
| runs | `superdirector/run-0203` 56.7 s, 14 turns, $0.2219; `supercoder/run-0284` 145.6 s, 38 turns, $0.7278 — **$0.95** |

A 40-minute gap between the plan (10:33) and the start (11:12) is the
Omni Agent's: the watch on autolab's log missed the plan line, and the
task waited for its start post. Not an agent defect.

**Verified in the tree, not the report** (`git diff 9f0da53 fd28557`,
7 files, +139 −24):

- `reports/leading-companies.md`: both mentions now read €1.16B for a
  **26.32 % economic interest**, cite the closing release, and keep the
  March 2025 ~25 % as a labelled superseded announcement. The `$1.25B`
  and `$1.3–1.36B` conversions are gone; `grep` finds no dollar figure for
  the deal anywhere in `reports/`.
- `reports/investment-synthesis.md`: `$1.3B` replaced by the euro figure;
  every General Intuition sentence updated.
- `reports/emerging-companies-and-investment.md`: the 2026-08-24 round
  added to the existing entry, **labelled "in talks, not closed"** — the
  researcher read the TechCrunch article and found it reports talks, where
  my request had called it a round. The correction corrected the
  requester; that is the labelling discipline working.
- `reports/index.md` and `README.md` link `knowledge-tree.md`.
- `knowledge-tree.md`: a "Run 2" entry in the Run 1 format — what was
  checked, the primary sources, the exact per-file changes, two items
  still open (whether the $6B round closes; a general FX convention).
- `methods/sources-and-approach.md`: **extended** with a currency
  convention (state the native figure alone unless a citable rate and date
  exist; if a conversion is kept, one figure, stated basis, applied
  identically everywhere), pointing at the Run 2 entry as the case that
  produced it.
- `publish/` untouched, still one commit, level with its origin.

**Methods: reused and extended, not merely proposed.** The task's report
names the three rules it reused (primary-source preference, figure-type
labelling, cross-file checking) and says plainly that the first pass's
synthesis step should have caught the dollar drift and did not.

**One residual, left for the next pass rather than bought another run:**
section 1 of the synthesis still says "General Intuition's rapid climb to
a $6B valuation" without the in-talks qualifier that sections 2 and 5 now
carry. Recorded here and to be named in the publication request.

The mission was closed through the current interface — the relay's
completion door (`POST /complete` on the workplan topic): m7061 `done`,
its task `accepted`, `#work-m7061` archived, the workplan topic ✔, each
confirmed by the mirror. The first mission (m7026) is left for step 6,
where Front is asked to do the same.

## Reproducibility

This study did no computation and used no service, so the plan's
instruction applies: verify representative retrieval and the documented
reasoning rather than invent a database exercise. Representative retrieval
was verified above — every figure checked was reachable from the outlet
and document the report names, and the correction pass reached the same
primary documents from the methods file's rules without being told where
to look. What the methods file documents is sufficient to redo the
retrieval: the cluster split, the source classes and named trackers, the
labelling rules, the cross-checking rule, and now the currency rule. What
it does not carry, and would need if a later pass computes anything, is
per-figure URLs or document identifiers — traceability today is by search
against a named outlet and date. The exact-replay versus recomputation
distinction does not arise: there is nothing to replay.

**The optional-tool experiment remains untested.** Neither pass wanted a
service; both said so. Nothing in this step manufactured a reason for one.

## Step 5 conclusions

1. The reports' central figures are right; one stake percentage was the
   announced rather than the closed figure, one deal carried three
   dollar values, and one in-window funding report was missing.
2. The researcher corrected all three in `main/` from primary sources,
   improved on the request's own wording, recorded the pass in its run
   log, and extended its methods where the defect showed a gap.
3. `main` is at `fd28557`, pushed; `publish/` is untouched; m7061 is
   done and closed. Cost of the pass: $0.95.
4. One phrasing residual is carried into the publication request.
