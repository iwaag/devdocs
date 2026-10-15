# Agent session p1 ex3 — final report

Plan: [plan.md](plan.md). Step reports:

- [report1](report1.md): baseline, contracts and budget;
- [report2](report2.md): protocol and driver;
- [report3](report3.md): execution;
- [report4](report4.md): evaluation and accounting;
- [report5](report5.md): reporting and preservation.

Per-set reports: [set A](set-a.md) (rally points), [set B](set-b.md)
(Shift waypoints), [set C](set-c.md) (save/load). This builds on
[ex2's report](../ex2/report.md).

## Result

| Set | fresh | run (retained) | run/fresh | Quality | Major work differences |
| --- | --- | --- | --- | --- | --- |
| A rally points | **$6.57** | **$6.00** | 0.91 | both accepted, all criteria | same design; the same help-layout defect and repair in both |
| B Shift waypoints | **$3.71** | **$3.02** | 0.81 | both accepted, all criteria | same design; no repair in either; fresh also added a "残り N" counter |
| C save/load | **$5.57** | **$6.08** | 1.09 | both accepted, all criteria | different designs (strict vs lenient validation); one different defect and repair in each |
| **All three** | **$15.85** | **$15.10** | **0.95** | 6/6 accepted | |

- **Active time** (attempt wall time): fresh 4,392 s, run 3,256 s (0.74×).
- **Steps**: fresh 544, run 392.
- The totals include every attempt, the follow-ups and both arms' repairs.
- Total spend was **$30.95** of the $45 cap. No run reached its $9 or
  45-minute ceiling.
- The costs are client-side list-price estimates on a Max subscription.
  They are not bills or quota.

**Conclusion.** On these somewhat larger tasks, with immediate answers, the
two policies cost about the same.

- `run` was cheaper in A and B, more expensive in C, and 5 % cheaper overall.
- It was consistently faster: fewer steps and about a quarter less time.

ex2's pattern holds. Comparable attempts cost similar amounts, and the
difference in a whole run comes mostly from what work happened. It does not
establish an average: there are three pairs, one model, and in C the designs
diverged.

## Phases (sum over the three sets, $)

| Phase | fresh | run | Note |
| --- | --- | --- | --- |
| Plan (new session in both) | 1.66 | 1.68 | equal by construction |
| Initial implementation (implement, verify, present, ask) | 4.97 | 4.05 | run cheaper in A and B, slightly dearer in C |
| Staged follow-up | 4.78 | 4.34 | run cheaper in A, equal in B, dearer in C |
| Defect repair (A and C) | 3.12 | 3.24 | same defect in A; different defects in C |
| Close-out (cleanup, final report) | 1.32 | 1.79 | run dearer: few steps over a 140–220 k prefix |
| **Total** | **15.85** | **15.10** | |

No inconvenient work was subtracted. The repairs are real work in both
arms, and they are in the totals.

## Why: reconstruction against carried history

| Mechanism | fresh | run |
| --- | --- | --- |
| Context at the start of a later attempt | 44 k every time (29 k-char bootstrap) | previous context + 0.8–1.3 k: 86–221 k |
| Full-history reads after the plan | 15 (111 k chars) | 0 |
| Help, plan and definition rereads | 23 help, 6 plan, 7 definition | 10 help, 1 plan, 0 definition |
| Source reads | 49 (478 k chars) | 29 (253 k chars) |
| All tool output | 892 k chars | 416 k chars |
| Cache read / cache write | 44.5 M / 1,072 k | 55.0 M / 517 k |
| Compaction or cache expiry | none | none |

- A fresh attempt spends steps rebuilding what it knows: the record, the
  plan, help pages and the source it will change. Each of those steps is
  cheap, because its prefix is small.
- A retained attempt skips that work. Each of its steps rereads a prefix
  two to five times larger.
- In these runs the two effects roughly cancelled:
  - The retained attempt was cheaper where its prefix stayed below about
    160 k: the implementation and follow-up attempts of A and B.
  - It was dearer where the prefix reached about 160–220 k: A's repair,
    C's implementation through repair, and every close-out.
- Token counts are not cost. The retained arm read 24 % more cached tokens
  and wrote half as many, and came out 5 % cheaper.

## What carried over from ex2

- **Boundaries were clean.** Every answer was delivered after the attempt
  had ended and its process was gone, so neither arm continued inside its
  own process.
- **Continuations worked on larger tasks.** Every resumed attempt was
  given 1.7–2.1 k chars carrying the answer's exact text. It acted on that
  text without reading the record, including graph revisions for
  follow-ups and repairs.
- **No harness surprises.** There were no stale notices on resume and no
  compaction.

## Recommendation

**Keep `fresh` as the default; `run` is a reasonable per-workflow choice.**

- **Cost.** The evidence shows no penalty for `run` on medium tasks when
  answers come quickly. It even shows a small saving, plus about 25 % less
  wall time. That does not justify switching the default:
  - the saving is small;
  - it reversed on the largest task;
  - p1 showed that a wait of more than an hour loses the cache and
    rewrites the whole prefix.
- **Choose `run`** for workflows where people answer within minutes and
  the history stays well under about 150 k.
- **Keep `fresh`** where waits are long or runs grow large.
- **Not measured**: a plan → implementation session split, which ex2
  designed but did not test.

No production setting was changed.

## Preserved outputs

All six implementations pass their own checks and the independent ones.
They are on the branches `trial/agent-session-p1-ex3/{a,b,c}-{fresh,run}`,
based on `7669fdb` (`979d6a7` + two trial workflow files).

The branches are pushed to the RTS project's Gitea repository
(`developer/pj-rts-vs-bot`), with the person's manual approval after the
first attempt was refused. They are also kept in the ignored
`pj-agdev/.local/agent-session-p1-ex3/trial-branches.git` and in each run's
bundle.

### Integration into `main` (after the trial, at the person's request)

The person asked for any of the implementations to be integrated. All three
features went into the RTS `main`. Only the `game/` changes were taken: the
trial branches' run records and trial workflow files stayed out. There is
one commit per feature, fast-forward from `979d6a7`:

| Commit | Feature | Source | Why this arm |
| --- | --- | --- | --- |
| `a05a7a0` | rally points | `a-run` | sets the rally for every selected production building |
| `d01f9fb` | Shift waypoints | `b-fresh` | adds the remaining-count label and in-game help |
| `ce67c01` | save/load | `c-run` | serializes entities as raw data, so the other features' new fields are saved without a field list |

Integration work beyond the arms:

- **Merge conflicts**: README rows, the two draw functions, the help text
  and the test blocks, resolved by keeping both sides. The help text gained
  the Shift+right-click entry.
- **Validation**: save/load now checks each entity's rally point and
  waypoint list.
- **New test block 13**: a rally point and a queued route survive
  save/load, and the route then finishes identically to the original. A
  broken or missing waypoint list and a broken rally point are rejected.

Checks on `ce67c01`:

- headless suite 73/73 (baseline 31 + A 15 + B 9 + C 11 + combined 7);
- the independent A, B and C probes 29/29;
- all five corrupt saves rejected;
- the browser checks of all three features in Chromium and WebKit, with no
  page errors.

Pushed to Gitea `developer/pj-rts-vs-bot` `main`. The production workspace
was fast-forwarded to it. No RTS run was in progress, and nothing else is
deployed for the game.

### Run records imported (`ba2daa7`, at the person's request)

The person chose renamed records over a separate viewer. The six run
records are committed in the RTS project as runs kept in the project tree,
with unique ids per set:

| Set | fresh | run |
| --- | --- | --- |
| A, rally points | `ex3-fresh/run-001` | `ex3-run/run-001` |
| B, Shift waypoints | `ex3-fresh/run-002` | `ex3-run/run-002` |
| C, save/load | `ex3-fresh/run-003` | `ex3-run/run-003` |

**What changed in the records.** Only `<workflow>/run-001` was rewritten in
sets B and C: paths, branch names and the run id, in 20 files. Presented
content and its digests are unchanged. Each run folder has an `IMPORTED.md`
naming:

- its original identity;
- the trial branch;
- what was rewritten;
- where its attempt logs and costs are.

**How they appear.**
- The project page and `/api/…/runs` list all six as accepted.
- The run views show graph, questions, presentations, decisions and
  reports.
- The trial workflows `ex3-fresh` / `ex3-run` are not added to the
  project; the run view shows their source as no longer present. That
  keeps them from being offered as runnable workflows.
- This workspace's exec store has no rows for their attempts.

**Notifications.**
- Before the commit landed, the six run keys were registered in the
  notifier store as `skipped`, the state it gives runs created before
  announcements began. A backup was taken first.
- Result: no message was queued and no IO channel was made.
- The notifier did provision the two workflow channels it makes for every
  workflow with runs: `wf-rts-vs-bot.ex3-fresh` (#302) and `.ex3-run`
  (#303). They are empty.

Checked by rehearsing on a private service first, then on the production
API and UI.

## Versions and conditions

- **Deployed**:
  - WFE agdevworld `a642e27`;
  - executor agautolab `1f1ca20` (tests `9382b1b`), run in-process by the
    driver against private areas;
  - harness claude_code 2.1.283 with `claude-sonnet-5`, the host's default
    profile;
  - wfrun guide `cf7ff7b2…`, `agent/review.md` `fd1fa285…`, catalog
    `1d35e919…`.
  - None of these changed during the trial.
- **Environment**: the Omni session's `CLAUDE*`/`ANTHROPIC*` variables were
  removed, as in ex2. The trial cwd was outside the projects tree.
- **Order**: A fresh-first, B run-first, C fresh-first, one execution slot.
- **Shared cache**: every run's first session read the same 20.4 k
  harness prefix from cache.

## Limits

- **Sample size.** Three pairs, one model and one game. A result in either
  direction is a diagnostic, not an average.
- **Design divergence.** In C, the arms built different validators and
  hit different defects. Repair cost was similar ($1.41 against $1.39), but
  the designs shaped the later prefixes.
- **The evaluator's choices shape the repairs.** The repairs depend on what
  the evaluation checked. The same checks were applied to both arms of
  each set. C's corrupt-save battery was added during C run's evaluation
  and run on C fresh's evaluated commit, where it passed. A truly
  identical sequence of evaluation was not possible once the first arm's
  defect was known.
- **Short waits only.** Answers came within two minutes. Long human waits
  were not tested (see p1).

## Deus Ex Machina note

None of the in-system agents' work was done by the Omni Agent. Every
person input was a labelled scripted answer from the frozen contract,
including the defect reports, which name observed behaviour only. The
evaluation was independent and used no model.
