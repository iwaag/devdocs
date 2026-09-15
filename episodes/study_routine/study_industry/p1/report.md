# study-industry p1 — phase report

Date: 2026-09-15 JST. Source: [braindump.md](braindump.md). Plan:
[plan.md](plan.md). Steps: [1](report1.md) · [2](report2.md) ·
[3](report3.md) · [4](report4.md) · [5](report5.md) · [6](report6.md).

The `study-industry` routine exists, ran once for the video game industry
from a one-sentence request with no human post after it, produced a
sourced first study pass whose central figures reproduce against their
original sources, corrected the three defects an independent check found,
and was reviewed for publication into a local commit that awaits the
developer's push. **21 agent runs, $10.63**, all `claude_code` on
`anthropic/claude-sonnet-5` at the default execution option. The one
capability the braindump added — optional local statistical or database
services — was read, considered in the plan, and declined with a reason;
it remains untested.

## 1. Layout, guide, and references

```
studyindustry/                      autolab study project, #pj-studyindustry (stream 163, folder 18)
  README_PROJECT.md                 ignored; repositories, .local/ rule, publication contract
  main/          autodev/studyindustry (internal Gitea)     787c1c0 → 9f0da53 → fd28557 → 06e9105, pushed
    README.md
    knowledge-tree.md               run-by-run log of what was checked, incl. null results
    methods/sources-and-approach.md
    reports/{index,leading-companies,emerging-companies-and-investment,investment-synthesis}.md
  publish/       https://github.com/iwaag/study-industry.git   a91105c (CC0, pre-existing) → bb77e52, local only
```

| thing | reference |
|---|---|
| routine channel | `#routine-study-industry`, stream 164, folder `routine` (13); Developer, Front, Opsroom Observer |
| guide | **v1, message 7015** (7,774 chars stored intact); the plan's intent as two block quotes, one added optional-service paragraph, operational context only; no source list, no template, no agent names |
| study run | `#front › front-studyindustry-20260915T1012Z` (7016) → `routinerun-20260915T101256Z` (7019) → `workplan-videogame-industry-study` → m7026 → `work-m7026 › workrun-task1-m7026` |
| correction | `workplan-correct-videogame-pass1` (7059, Developer direct) → m7061 → `workrun-task1-m7061` |
| publish run | `front-publish-studyindustry-20260915T1117Z` (7081) → `#routine-publish › routinerun-20260915T1117Z` (guide v2, 6254) → `workplan-publish-studyindustry` → m7093 |
| close-out | `front-closeout-m7026-20260915T1117Z` (7083) → `#autolab-agstudio1 › close-m7026` |

`README_PROJECT.md` is agent-written and tracked nowhere, as with every
study project.

## 2. Scope, findings, and commits

The researcher chose, before starting: global geography with US/Japan/
China/EU as the lens; three segments (platform holders and publishers,
engine/tooling, storefront and cloud streaming); twelve incumbents plus an
emerging-company list to be a research output; roughly 2024-09 to a
**2026-09-15 cutoff**; out of scope esports as a business, hardware supply
chains, culture, regulation beyond what explains a corporate move,
sentiment, stock trading. Five parallel subagents, one per cluster.

What the reports say, in the researcher's own synthesis: capital is
concentrating in fewer, larger strategic deals (the $55B EA take-private,
Tencent's €1.16B for 26.32 % of Ubisoft's Vantage Studios) while studio
venture funding stays depressed; AI is the one category growing at every
tier (HoYoverse's reported RMB 100bn plan, Tencent's Hunyuan platforms,
Unity Vector's >$1B run rate, EA–Stability AI, the world-model startups),
with Nintendo and Take-Two as stated counter-evidence; live-service
over-investment recurs behind the layoff waves; Chinese majors are the
most active outbound acquirers of Western IP; emerging-company capital
bifurcates between modest studio rounds and outsized AI-infrastructure
raises. The synthesis ends with six indicators to watch, not forecasts.
Gaps are explicit: GTA 6 (after cutoff), no reconciled VC total across
trackers, no disclosed Steam/Epic share, six things checked with nothing
found. Every figure carries an outlet and a date; labels separate funding
from announced from completed deals and buzz from traction; estimates are
labelled.

| commit | by | what |
|---|---|---|
| `9f0da53` | study run | six files, 958 lines |
| `fd28557` | correction mission | stake 25 % → 26.32 % from the closing release; three dollar figures → none; General Intuition's 2026-08-24 round added as *in talks*; run log linked; methods extended with a currency rule |
| `06e9105` | publish gate | residual phrasing fixed; six unsourced bullets relabelled "source not retained"; one deletion (per-subagent timings) |
| `bb77e52` (publish) | publish gate | the six files plus a stranger-facing README, local only |

## 3. Checks and reproduction

Thirteen claims were checked against their sources ([report5](report5.md)):
ten central figures reproduced (Sony, Nintendo, Tencent, EA, Ubisoft,
Epic, Embracer, InvestGame, Take-Two, Marvel Rivals); seven derived ratios
recomputed and agree; definitions (bookings vs revenue, fiscal-year
naming, deal labels) are used as the companies use them. Found: the
Vantage stake given as the announced 25 % rather than the closed 26.32 %;
the same deal at three dollar values across two files; a General
Intuition round reported inside the window and missing. The researcher
corrected all three from primary documents and, reading the article,
corrected the request too (the round is in talks, not closed). The fetch
summariser returned a fabricated reading of the Ubisoft PDF; the page was
read directly.

The study did no computation and used no service, so reproduction was
representative retrieval: every checked figure was reachable from the
outlet and document the report names, and the correction pass reached the
primary documents from the methods file's rules alone. Methods were
**reused and extended**, not merely proposed. What the methods do not
carry is per-figure URLs; traceability is by search against a named
outlet and date, which the publish gate later showed had failed for six
small funding-round lines.

## 4. Optional tools, notification, services

Considered and declined, twice, with a stated reason ("findings-and-
citations based rather than requiring computation over a dataset"). No
notification was sent because nothing was going to be used; cagent was
not asked; no service was created or reused; there is no operational
ownership to record. The clause was relayed by Front into the delegation
in full, and autolab's plan addressed it before starting. That is the
whole of the evidence: the option was understood and judged, and the
judgment was that this study did not need it. Whether the clause ever
produces a valuable analysis is untested.

## 5. Publication, separately stated

- **Prepared:** `publish/` at `bb77e52`, six files byte-identical to
  `main/`, zero dangling links, no private facts or internal vocabulary,
  README written for a stranger, the origin's CC0 history intact.
- **Not pushed:** the GitHub origin still holds only `a91105c`. The
  developer reviews and pushes by hand. The URL identified the
  repository; the contract was restated in the request, not overridden.

The `publish` guide needed no new version: it discovers projects from the
workspace and `README_PROJECT.md`, and it found this one.

## 6. Costs, timing, failures, and the limits of the experiment

| stage | wall | runs | cost |
|---|---|---|---|
| project setup | 1 min | 1 | $0.30 |
| study run (request → delivered report) | 15 min | 7 | $5.49 |
| correction mission | 4 min of agent time (+40 min waiting for a start post, see below) | 2 | $0.95 |
| publish run | 10 min | 7 | $2.97 |
| m7026 close-out via Front and autolab | 13 min | 4 | $0.91 |
| **total** | | **21** | **$10.63** |

By role: `supercoder` $7.04 (three tasks — the work), `routine_run`
$1.35, `front` $1.25, `superdirector` $0.98. The single study task,
$4.58, is where the research money went: five concurrent web-research
subagents in one session.

**Interventions: none during any run.** The study run, the publish run
and the close-out each went from request to delivered result on their own;
the only Developer posts were the requests, one "yes" to Front's standing
permission question, and the start post for the correction task, which is
the requester's job. The 40-minute gap before that start post was the
Omni Agent's watch missing the plan line — not an agent defect, and the
task simply waited as designed.

**Observed defects and oddities, none repaired here:**

1. autolab resolved the task topic before the requester agreed (both
   study tasks and the setup), against its own introduction. Nothing was
   lost; third project in a row.
2. The routine run did not close its mission: the plan topic stayed
   open with no acceptance until asked for. The completion door and
   autolab's own channel both work; neither is called by the run.
3. Front's first run serving opened with a sentence addressed to nothing
   in the conversation ("That's fine, just a stray parser note…"); the
   rest was correct.
4. Front named `agobserver` as an example when relaying the guide's
   deliberately agent-free "an agent on the board that waits".
5. Six funding-round lines had no source, contradicting the methods
   file's own claim, and were caught only at the publish gate.
6. `devlog/` appeared in the workspace again (fifth project).
7. The task run record's `duration_ms` measures the parent session, not
   the 12-minute wall time its subagents took.
8. autolab's `mission_done` leaves the `work-` channel unarchived; the
   completion door archives it. `#work-m7026` remains.
9. The WebFetch summariser fabricated a reading of a PDF (outside the
   system, but it is the same tool class agents use).

**What one trial establishes, and what it does not.** It establishes that
the routine is reachable, that a bounded, sourced, honestly-gapped first
pass can be produced and published-ready in under half an hour of agent
time, that the researcher reuses and extends its methods when corrected,
and that the current routing repairs held on this path (fresh workplan,
run-anchored callbacks, report delivered, no stall because no long job
was started). It does not establish a general improvement in autonomy:
one industry, one afternoon, no long-running job, no service wanted, no
external limit met. The optional-service clause has produced exactly one
data point — a considered "no" — and cannot yet be called Tool Giving
that worked or failed.

## Left behind

- The routine stays available on guide v1. `publish/` awaits the
  developer's push of `bb77e52`.
- `workplan-setup-studyindustry-workspace` (setup) and
  `autolab-agstudio1 › close-m7026` are open; `#work-m7026` exists with
  one ✔ topic. `README_PROJECT.md` still says no research has been done.
- Open research items are in `knowledge-tree.md`: the unsourced funding
  bullets, whether General Intuition's $6B round closes, Tencent–Hungry
  Studio, GTA 6 after the cutoff, a general FX convention.
- Raw evidence — every conversation, run record, diff, check note and
  board snapshot — is in the ignored `.local/` beside this plan.

## Deus Ex Machina

- Developer-side work by design: the project channel, both requests to
  Front, the direct correction request, the start post, the permission
  answer, and the completion door for m7061 and m7093.
- *did the workspace pattern marker for agent autolab — handoff
  candidate.*
- *did the source, arithmetic and cross-file audit of `main/` for agent
  Front — handoff candidate.* Front's approvals are of what it is told;
  as in the previous study, an independent check on figures and
  definitions is available only from outside the system, and it is what
  found the three defects.
- Nothing in `main/` or `publish/` was written or corrected by the Omni
  Agent.
