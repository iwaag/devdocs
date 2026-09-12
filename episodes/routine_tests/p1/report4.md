# Step 4 follow-up — the fixes, measured on a live run

The trial left three defects fixed but unmeasured and half the user's request
unmet. One follow-up run settles both. It is a **clearly identified
follow-up**, not a redo: none of the completed work was repeated.

## What was asked

`#front` › `front-publishall-20260912T1250Z`, message **6255**,
2026-09-12T12:47:18Z, as the Developer:

> Across all study projects, publish everything that has not yet been
> published.
>
> Earlier today a run of this covered only some of the study projects, so
> please work out the full set yourself before you start, and account for
> every project you find — including the ones that turn out to have nothing
> left to publish.

The first sentence is the original request verbatim. The second says *find the
set yourself* without naming a project or a count — so coverage is still the
system's to get right, and the measurement is honest.

Front opened `routine-publish` › `routinerun-20260912T1248Z` at 12:48:28Z,
reading guide **v2** (message 6254) and saying so. It finished at 13:14:53Z —
**26 minutes 25 seconds**.

## Measurement 1 — coverage: fixed

Front discovered the set from the workspace rather than from the table, and
found **five** publish-capable entities where v1's table named two:

| entity | candidates | outcome |
|---|---|---|
| `studyarxiv` | 17 papers | no-op, re-verified by `diff -r` — no task opened |
| `studyrealworld` | 16 reports + 6 source files | no-op, re-verified by `diff -r` — no task opened |
| `ghtrends` | 11 | **published**: `main` `e31519c` pushed, `publish` `44ab05a` local |
| `mediagen` | 15 | **published, first ever**: `main` `331fe0e` pushed, `publish` `7220044` local |
| `studynourl` (archived) | 0 | found at discovery, empty `main/` and `publish/`, no-op, not delegated |

It also deduplicated `papers` explicitly, citing its `README_PROJECT.md`
naming the same repository as `studyarxiv`'s `main/`.

Two things exceed what was asked. It surfaced
`.local/projects-archived/studynourl/`, which satisfies the discovery rule
literally, and accounted for it as a no-op rather than silently skipping it or
opening a pointless mission. And it re-verified the two no-ops by content
rather than trusting filename parity — which is what its delegation briefs
told autolab to do as well.

**Verified against the trees, independently:**

- `ghtrends` — all 10 repo summaries plus `index.md` now published; every
  published file byte-identical to `main`, `README.md` excepted. One gate fix
  in `main` (`e31519c`, an unattributed figure reframed as the project's own
  claim).
- `mediagen` — all 15 files published including both PNG contact sheets; every
  one byte-identical to `main`, `README.md` excepted. Four files fixed in
  `main` (`331fe0e`) — internal jargon, repository references, a
  reporting-chain description and dangling paths genericized, two quotes
  paraphrased.

Both `publish/` clones are committed locally and **unpushed**. Their remote
relationship is neither ahead nor behind but **gone**: those origins hold no
branch at all, because neither had been published before. The guide asks for
the clone's remote relationship to be reported without acting on it, and that
is the honest reading of this case.

So the user's original request is now satisfied across all four study
projects, and the publication boundary held everywhere.

## Measurement 2 — the premature finish: fixed

This is the one that mattered, because it was answered with guidance rather
than a code guard.

**Before:** four `ag-routinerun` blocks across three runs, every run's first
serving, each contradicting itself.

**After:** the follow-up run's record holds **17 entries and exactly one
finish block** — the real one, at the end, `achieved: true`. Its first serving
delegated to four projects and then simply stopped, with no block at all,
which is precisely what the rewritten guide asks for. No run topic was ever
resolved early.

## Measurement 3 — the twin fork: not reproduced

`routine-publish` holds one topic per run and no twins. Because no run ended
early, no callback ever arrived at a resolved run, so the repaired code path
was **not exercised** — the defect was removed at its cause rather than
survived at its symptom. `relay_late_answer` remains covered by tests and
unproven in the realm; that is stated rather than claimed.

## Interventions

**None.** No anchor repair, no topic closed by hand, no nudge. The chain ran
from one request to a complete report on its own — which the first attempt,
being assisted, could not show.

## Cost

Front 30 runs / **$7.9461**; autolab 15 runs / **$10.7386**. All
`claude_code` on `anthropic/claude-sonnet-5`. Higher than the original trial
because two real publications ran here (19 gated files against the trial's 25,
but including a first publication with images) — and because a full
discovery pass reads every project's `README_PROJECT.md` before delegating.

## What this changes in the p1 conclusions

- **Work completed** — all four study projects are now published to the local
  boundary. Four `publish/` commits await the developer's manual push:
  `8901b34` (study-realworld), `cac3527` (study-arxiv), `44ab05a` (ghtrends),
  `7220044` (mediagen).
- **Autonomous coordination** — now shown twice: the composite chain across
  three stages (assisted), and this four-project run end to end (unassisted).
- **Still open** — `relay_late_answer` is untested in the realm; the
  post-sweep recovery hook has not yet had an owed handoff to recover; and two
  workplan topics from the trial are left unresolved, which Front noticed and
  reported rather than tidying on its own, as its contract requires.
