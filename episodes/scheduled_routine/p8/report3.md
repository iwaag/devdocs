# Step 3 — the publish routine, defined and fired once

## Standing text (`#front › routine-publish`, message 2753, v1)

> Standing request for the `publish` routine, v1.
> 
> In `#pj-studyarxiv`, ask autolab for **one** mission that reviews the reports in `main/papers/` and produces a sanitized **copy** of the publishable ones in `publish/`. This review is its own mission, always — never a self-check appended to a `papers` or `localtest` run. Autolab's project pattern (`agent/project_pattern.md`, "study" pattern) describes `publish/`; read it first.
> 
> **Candidates.** Every paper in `main/papers/` that has a `test.md` is a candidate. Whether a summary-only paper (no `test.md`) is worth publishing is autolab's call; say which rule was applied.
> 
> **The three conditions**, checked per candidate file (`summary.md`, `manual.md`, `test.md`):
> 
> 1. **No local-environment or secret facts.** Grep-level: hostnames (`agstudio`, anything `.local` or `.home.arpa`), local paths, ports, socket paths, container/volume names, usernames, credentials or credential placeholders, Gitea repository URLs. Environment information is reduced to generic experimental conditions — "Apple-silicon Mac, local Ollama, model `<name>`", processor model, memory.
> 2. **The paper's version is clearly stated.** `summary.md` normally carries a "First posted …; current version vN" line; verify it is present and names a version, add it from arXiv if missing.
> 3. **Quote hygiene.** No long verbatim passages from the paper (copyright-scale quotation), nothing presented as a quotation that is not in the paper, and no direct transcription of quoted text in general — paraphrase.
> 
> **Edit, then copy — never move.** For each candidate, edit the files into compliance (rewrite local facts into generic ones, add a missing version line, paraphrase or trim quotations, fix links that would dangle outside `publish/`) and copy the result into `publish/` in a per-paper layout of autolab's choosing. `main/` stays exactly as it is: no file moved, deleted, or edited there. Add or update a `publish/README.md` index. Commit `publish/` locally and **do not push it** — the developer reviews the commit and pushes by hand; final publication is that manual push, nothing autolab does.
> 
> **Rejection is a valid result.** If editing cannot honestly satisfy a condition — a claim whose only evidence is private, a quotation that cannot be verified against the paper — reject that paper, name the condition, and move on.
> 
> Report, per paper: copied with a summary of the edits made (the `main/` → `publish/` diff is the evidence of what the gate did; keep it readable), or rejected with the condition that could not be met. Then the `publish/` commit hash and file list, and as the last lines your own harness result JSON — `cost_usd`, `num_turns`, `duration_ms`.
> 
> Approve a plan that is roughly that; if it is off, say what is off. Approving means acting — start it and check it through to Done, ask nothing. Report here when it is Done with the workplan topic and the per-paper verdicts.

## Schedule and dispatch timeline (all 2026-08-29 UTC)

| time | event |
|---|---|
| 08:01:38 | Developer asks Front in `front-schedule` for one `publish` fire at 08:16:00Z (message 2754) |
| 08:01:57 | Front: request **`r12`**, event **`e42`**, routine `publish` (2756) — direct schedule edit, no other agent |
| 08:12:09 | Developer moves `e42` to 08:12:00Z with `rtschedule move` and `launchctl kickstart`s `com.agdev.routine-dispatch` (user asked not to idle until 08:16) |
| 08:12:20 | production dispatcher: "marked e42 before action … dispatched and pushed e42" |
| 08:12:21 | `trigger.sh` posts the run line into `#front › front-routine-publish` (2758) |
| 08:13:01 | Front opens `#pj-studyarxiv › workplan-publish-2026-08-29` with the full mission (2761) and reports so (2763) |
| 08:13:44 | autolab plans: Work **S3-12**, sub-work **S3-13**, `start.flag` created itself citing the standing go-ahead, `work-s3-12/workrun-task1-s3-12` opened (2767) |
| 08:14:04 | **Front posts the start message into the workrun topic itself** (2770) — the routing p6 left unproven now works end to end without Omni |
| 08:14:04–08:19:21 | supercoder runs, reports (2777), resolves the topic (2778) |
| 08:20:03 | Front's final report to the Developer (2780) |

Front's mission text (2761) is a faithful restatement of the standing text
— all three conditions, edit-then-copy, never move, no push, rejection
valid, per-paper report, result JSON.

## The `publish/` commit

`df91fd3` "Publish 2608.23552 and 2608.23283: generic environment,
verified arXiv versions", **local only, not pushed** (`git status` clean,
`origin` = GitHub, one commit ahead of `fbf6517`). Files (+418 lines):

```
README.md
papers/2608.23283/manual.md  summary.md  test.md
papers/2608.23552/manual.md  summary.md  test.md
```

`main/` untouched: `git status` clean at `9931cfa`.

## Per-paper verdicts and the main → publish edits

| paper | verdict | edits (full `diff -u` main→publish) |
|---|---|---|
| 2608.23552 Prime Agent | copied | `summary.md`: "First posted: 2026-08-05; current version v1: 2026-08-24" → "First posted: 2026-08-24; current version v1 (2026-08-24)" — the routine checked arXiv's submission history and found the 08-05 date wrong. `test.md`: "See repository `autodev/studyarxiv-localtest-2608.23552` …" → "A separate, internally tracked repository holds the full raw run log …". `manual.md` unchanged. |
| 2608.23283 Apodex 1.1 | copied | `summary.md`: "submitted 2026-08-24" dropped from the byline, new line "First posted: 2026-08-24; current version v2 (2026-08-25)." added (there was no version statement at all). `test.md`: same internal-repo rewrite. `manual.md` unchanged. |
| 7 summary-only papers | rejected | one rule: publication requires a completed local test; summary/manual are documentation-derived working notes, not an independently checked finding. Listed in `publish/README.md` under "Not published" with the rule. |

The diff is four hunks of one or two lines — readable, and exactly the
kind of edit the braindump wanted (condition 1 rewrite, condition 2
added/corrected). Condition 3: the routine judged the only blockquotes to
be short, attributed README excerpts, not paper text.

`publish/README.md` (45 lines) explains the layout, lists the two papers
with their L1 level, restates the three conditions, and names the seven
rejected ids with the rule.

## Cost numbers (harness run records)

| run | role | cost_usd | num_turns | duration_ms |
|---|---|---|---|---|
| Front, schedule edit (`agfront/.local/agent/front/run-0260`) | front | 0.1740 | 9 | 17 501 |
| Front, open workplan (`run-0261`) | front | 0.1897 | 13 | 43 787 |
| Front, start workrun (`run-0262`) | front | 0.1358 | 10 | 23 673 |
| Front, final report (`run-0263`) | front | 0.1825 | 12 | 40 929 |
| autolab planning (`superdirector/run-0106`) | superdirector | 0.1363 | 9 | 41 457 |
| autolab task (`supercoder/run-0099`) | supercoder | **0.6728** | 38 | 315 281 |

Routine fire total ≈ **$1.49**, of which the review itself is $0.67 / 5.3
min; the Front legs are $0.68 across four short runs. Wall clock from
dispatch to Front's final report: 08:12:20 → 08:20:03 (7 m 43 s).

As in Step 2, the mission reported `{"cost_usd": null, …}` — a run cannot
see its own result JSON; the numbers above were read from the run records.

## Frictions and observations

1. **Front closed the loop itself.** Planning → start post → final report
   with no Developer/Omni post in any `pj-`/`work-` topic. This is the p6
   revalidation the plan asked for.
2. autolab created `start.flag` at planning time because the mission
   carried the standing go-ahead, and the task resolved its own topic
   after reporting. Front flagged the latter as contrary to autolab's
   introduction ("not closed until the requester agrees") — a real
   contract/behavior mismatch to note, not a failure.
3. The closeout of the Step 2 mission (`S3-10`) via autolab's own channel
   stalled: the entrance run hit a Plane rate limit on `mission_done`,
   posted "I'll report once it completes" and ended — a finished run cannot
   retry. Re-asked (2782) together with S3-12's closeout; result in
   `report.md`.
4. `e42` was moved forward by hand and the dispatcher kickstarted; still
   the production launchd job, not `--now`.
