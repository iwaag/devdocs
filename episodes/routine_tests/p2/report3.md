# routine_tests p2 step 3 (first run) — one attempt retired, one succeeded

Date: 2026-09-13 JST (timestamps UTC). Plan: [plan.md](plan.md) step 3.
Previous: [report1.md](report1.md), [report2.md](report2.md).

This report covers the **first** requested run only. The second run, which
must continue from what this one left behind, is step 3's other half.

## The request

One request at Front's ordinary entrance, `#front` ›
`front-uspolitics-20260912T1536Z`, message **6358** (15:33:03Z):

> Run the routine `study-uspolitics` once. Its guide is in
> `#routine-study-uspolitics` › `guide` — read the newest post there and
> open a run for it. Supervise it through to completion and report back
> here. One run only. No other conditions; use the current execution
> defaults.

No backend, budget threshold or schedule was named, per the plan.

## Timeline

| UTC | What |
|---|---|
| 15:33:19 | Front served the request — and answered *"I don't see a message or request from the developer yet"* |
| 15:34:03 | Request re-stated by the Developer (message 6361) — **intervention 1** |
| 15:34:44 | Run opened: `routinerun-2026-09-12T1534Z`, message **6364**, guide **6357** (v1), origin recorded |
| 15:35:31 | Delegated to autolab: `#pj-studyuspolitics` › `workplan-collect-and-analyze-contributions` (6368) |
| 15:36:17 | Plan for mission **m6371** posted |
| 15:36:59 | Front approved and started task1 itself (6381) — no human in the loop |
| 15:43:32 | task1 reported **progress, not completion**: FEC `DEMO_KEY` rate limit, subagent collision, a self-throttled background fetch launched |
| 15:44:06 | Front judged it correctly as still running and wrote **no** finish block |
| 15:43–16:05 | The background fetch made no progress; nothing served autolab again |
| 16:04:38 | Observed: **3 usable records out of 42**, still on candidate 1 of 30 after 6 failed retries |
| 16:05:05 | Plan scrapped and re-asked by the Developer (6394) — **intervention 2** |
| 16:07:10 | m6371 retired, `work-m6371` archived, replacement mission **m6400** planned |
| 16:08:08 | task1 of m6400 started (6410) |
| 16:10–16:15 | Collection, two concurrent analysis subagents, report, commit |
| 16:15:06 | task1 complete, `d19a5fb` pushed — but Front could not be reached |
| 16:17 | Front's run anchor restored (6423) — **intervention 3** |
| 16:19:19 | Run finished, `achieved: true`, report delivered to the Front Desk, run topic ✔ |

About 46 minutes from request to delivered report, of which the first
attempt consumed roughly 29.

## The first attempt, and why it was retired

autolab chose the OpenFEC REST API and called `/candidate/<id>/totals/`
per candidate using the FEC's public `DEMO_KEY`. Three parallel collection
subagents shared that one credential and one output directory. Two things
went wrong and one of them is worth more than the other:

- **The shared credential.** `DEMO_KEY` is api.data.gov's public key with a
  per-IP hourly allowance. The three subagents exhausted it between them,
  after which every call returned `OVER_RATE_LIMIT`. autolab stopped them
  and launched one self-throttled sequential script instead — 30 candidates,
  six 150-second retries each. Measured over 21 minutes, that script never
  got past candidate 2 and produced **3 usable records out of 42 attempted**.
  Extrapolated, it would have run about eight hours to write thirty files
  containing rate-limit errors.
- **The shared directory.** One subagent ran `rm -f .../raw_totals/*.json`
  to clean up error files while another was still writing there. autolab
  **flagged this itself**, unprompted, as *"Accidental data loss (my
  error)"*, and stopped all three subagents. Nothing had been committed, so
  nothing durable was lost.

Neither of those is the reason the attempt was retired, though. The reason
is what it did *next*: it neither asked for the real FEC API key that would
have removed the limit, nor looked for a route that needed no key. It left a
background job grinding and ended its run — and **nothing in the system
would ever have served it again**, because it was the last speaker in its
own task topic and Front was waiting on it. A run that is waiting on itself
has no wake signal.

## The guide gained a rule, and it is what made the difference

The developer added one instruction to the routine guide, posted as
**guide v2, message 6393** (a new full post, per the routine contract —
never an edit). The rest of v1 is word for word unchanged:

> **When something needs a human, say so — do not route around it.** Some
> work is not yours to do: obtaining an API key or an account, accepting
> terms of use, paying for access, anything needing a human's identity or
> authorization. When you meet one, either build it into the research plan
> as a step a human performs, or ask for it there and then. Do not silently
> substitute a degraded route for it, and do not stall waiting for it
> without asking.

The replacement plan was then asked for in the same workplan topic, with the
mission text unchanged, that rule quoted, and the subagent collision
reported as an observation rather than a prescription. **No source, method,
period or measure was chosen for the researcher**, and the plan said
explicitly that the first attempt's choice of the FEC API was not a decision
it had to keep.

### What autolab did with it

The replacement plan built the rule in as a step — a human-dependency check
before any bulk collection — and named the concrete case: *"if it again
reaches for FEC data, the developer should expect a request for a real
(still free) FEC API key"*. It also added per-subagent output directories,
a no-cross-deletion rule, and serialized or partitioned access to any shared
rate-limited endpoint.

Then the task ran the check, and **the check changed the answer**. It found
that FEC publishes the same data as a plain bulk download —
`https://www.fec.gov/files/bulk-downloads/2026/weball26.zip`, the "All
Candidates" file — with no key, account, login, payment or terms acceptance.
So it switched sources and **asked for nothing**.

That is the outcome the rule was for, and it is worth stating precisely: the
rule did not produce a request for a key. It produced a *check*, and the
check found the dependency was avoidable. Neither of the two failure modes
the rule names — silently grinding on a degraded route, or stalling without
asking — occurred.

A third thing the replanning caught, which nobody had asked for:
`main/.gitignore` excluded only `.DS_Store` and **not** `.local/`, so the
first attempt's bulk downloads and raw logs would have been committed into
the publish-ready repository. autolab flagged it and fixed it in the same
commit.

## What the run produced

Commit **`d19a5fb`** — *"First slice: FEC financial summaries for Senate
seats up in 2026"* — on `origin/main` of the internal `main` repository,
three files, 290 insertions:

| File | What |
|---|---|
| `methods/fec-senate-2026-candidate-financial-summaries.md` | 129 lines: the slice, what is deliberately excluded, both sources and their access methods, the field list and its published definition, and the caveats |
| `reports/2026-fec-senate-financial-summaries.md` | 160 lines: four findings, each split into observation / interpretation / uncertainty |
| `.gitignore` | `+.local/` |

**Scope, as the researcher declared it:** the 35 sitting US Senators whose
seats are contested in the 2026 cycle (33 regular Class II + 2 specials),
committee-level aggregate totals only. Explicitly *not* covered: itemized
donor-level contributions, the other ~65 sitting senators, the House,
leadership PACs, joint fundraising committees, and outside/super-PAC
spending. That is a narrowed first sample with its boundaries stated, which
the plan allows — and it is not coverage of Congress.

The findings, in one line each: receipts are extremely skewed (Ossoff alone
$77.3M of ~$280M); Democrats in the slice lean on individual donors and
Republicans somewhat more on PAC money; the nine senators FEC flags as not
running split into six dormant and three still financially active; debt and
self-funding are both rare.

## Verified independently, against the raw file

Not against the agent's derived JSON and not against its report — the raw
`weball26.txt` (4,294 rows) was re-parsed here with the roster of 35
candidate ids, and every checked figure was recomputed:

| Claim in the report | Recomputed here | |
|---|---|---|
| ~$280M combined receipts | $279,765,279.94 | ✓ |
| median ~$4.7M | $4,707,959.82 | ✓ |
| Ossoff (GA) $77.3M | $77,279,766.48 | ✓ |
| next: Booker $18.9M, Warner $17.9M | $18,869,341.39 / $17,889,335.15 | ✓ |
| one self-funder, Ricketts, ~$5.3M = 47% | $5,295,887.54 = 47.1% | ✓ |
| 5 of 35 carry debt, all under $75K | 5; max $73,959 (Cotton) | ✓ |
| all 35 roster ids found, no misses | 35/35 | ✓ |
| `CVG_END_DT` spans ~2 months, not one snapshot | 06/30/2026 → 08/26/2026 | ✓ |

Against the plan's step-4 questions: the source, retrieval time, period,
population and coverage limits are all identifiable from `methods/` alone,
and the retrieval is repeatable from it. The report distinguishes observed
facts from readings of them throughout, names its own outlier problem
(pooled-dollar comparisons dominated by one candidate), and says where a
category does not sum to 100% of receipts. Missing data did not become
zero: the report states that no figure in the dataset was absent and that a
`0` is a reported zero.

**Repository separation held.** `publish/` is still at `569a8e3` with only
its `LICENSE`, untouched, and the working tree is clean.

## The defect this trial found

**Retiring a plan strands the supervising run's anchor.**

When autolab retires a mission it renames the whole conversation aside —
here to `✔ retired-workplan-collect-and-analyze-contributions-m6371` — and
the replacement mission takes the original topic name. Front's
`[selfnote][rootchat]` origin note, **message 6367**, went with the old
messages into the renamed topic. The live m6400 topic carried no anchor at
all.

The consequence is complete and silent: autolab named `@Front` correctly,
twice, in two different topics, and Front correctly refused both —
`carries no root note of ours; ignoring`. The supervising run could not be
reached from the work it was supervising, and neither side was at fault.
Front's own rule (only a topic carrying its note reaches its run) is the
rule that protected it from p1's twin-forking defect; the retirement path
simply does not know about it.

This is the same family as p1's defect 3 — a rename that a routing lookup
cannot follow — on a path p1 never exercised, because p1 never retired a
plan mid-run. It is a demonstrated defect, its owner is the boundary
between `agautolab`'s plan retirement and `agfront`'s anchor lookup, and it
is left for a follow-up rather than patched inside this trial.

**Repair applied here:** the note was rewritten into the live topic using
Front's own credential (message **6423**), after which the very next mention
resolved — `mention … serves routine-study-uspolitics/routinerun-2026-09-12T1534Z`
— and the run finished normally. This is the same repair p1 had to make four
times, for a different cause.

> **Deus Ex Machina note:** *did the run-topic origin-anchor repair for
> agent Front, after a plan retirement moved it aside — handoff candidate.*

## Interventions — this run was assisted

Three, and they must not be read as a clean autonomous run:

1. **Front's first serving missed the request in front of it.** Its
   generation workspace was correct — `chatlog.md` held the request
   verbatim — and it answered in one turn, with no tool calls, *"I don't
   see a message or request from the developer yet"*. Re-stating the
   request in the same conversation was enough. A model-side miss, not a
   delivery defect; recorded, not repaired.
2. **The first attempt was retired by hand.** The researcher was stalled
   with no wake signal, so nothing in the system would have ended it. The
   developer scrapped the plan through autolab's documented
   retire-and-re-ask route. This is the intervention that matters most for
   what the trial can claim: *the system did not recover from the stall on
   its own.*
3. **The anchor repair**, above. Note that intervention 2 is what triggered
   the retirement that caused the stranding — so the defect was found by an
   intervention, though the defect itself is not a product of one.

Two further effects of intervention 2 are worth recording because they are
easy to mistake for defects: while the Developer was the last speaker in the
workplan topic, autolab answered the **Developer** rather than Front, and
Front stayed silent. That is correct behaviour on both sides. A third party
posting into an agent's delegate conversation takes over the reply, and the
routine's supervisor has no way to notice.

## Backends and cost

Every run was `claude_code` on `anthropic/claude-sonnet-5`, `exec_source:
default` — no execution option was requested, so none was set.

| Agent | Runs | Cost |
|---|---|---|
| autolab (`superdirector` ×5, `supercoder` ×3) | 8 | $4.8177 |
| Front (`front` ×2, `routine_run` ×4) | 6 | $1.0274 |
| **Total, including step 1's setup mission** | **14** | **$5.8451** |

The single most expensive run was the retired attempt's task1 —
`supercoder` run-0264, 176.0 s, **$1.8680** — which produced nothing
durable. The successful task1 (run-0265, 90.5 s, $1.7237) did the whole
collection, analysis, report and commit in half the time.

## Step 3 (first run) conclusions, kept apart

1. **Contribution data was actually collected and actually analysed.** The
   figures are verified against the raw source, the scope is declared, and
   both a method and a report are committed and pushed.
2. **The routine reached its goal**, `achieved: true`, and the report was
   delivered into the requesting conversation with the run resolved.
3. **It was not an autonomous run.** Three interventions, one of which
   (retiring a stalled attempt) the system had no way to perform itself.
4. **A guide change was measured, not assumed.** The rule added between the
   two attempts produced a dependency check that changed the chosen source
   from a rate-limited keyed API to a keyless bulk download. One trial is
   not proof that the rule generalizes.
5. **One defect is demonstrated and unfixed**: plan retirement strands the
   supervising run's anchor.
6. Whether accumulated knowledge is actually reusable is **not** answered
   here. That is what the second run has to show.
