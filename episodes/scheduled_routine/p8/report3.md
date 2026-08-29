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

## Schedule and dispatch timeline (all UTC, 2026-08-29)

| time | event |
|---|---|
| 08:01:16 | Developer asks in `#front › front-schedule` (2754) for one `publish` fire at 08:16:00Z, request until 09:31:00Z |
| 08:01:57 | Front adds request **`r12`**, fire **`e42`** (2756); direct schedule edit |
| 08:12:20 | production dispatcher (`com.agdev.routine-dispatch`) "marked e42 before action … dispatched and pushed e42" — its 5-minute tick fires anything due before the next tick, hence 4 min early |
| 08:12:21 | `trigger.sh` posts as Developer into `#front › front-routine-publish` (2758) |
| 08:13:01 | Front reads the standing text and opens `#pj-studyarxiv › workplan-publish-2026-08-29` (2761) — the mission text is a faithful, lightly condensed transcription of the standing request |
| 08:13:44 | autolab plans: Work `S3-12`, sub-work `S3-13`, `start.flag` created on its own (standing go-ahead), channel `work-s3-12`, topic `workrun-task1-s3-12` (2767, 2766) |
| 08:14:04 | **Front itself posts the start into `workrun-task1-s3-12`** (2770) — the p6 gap (Front never started a workrun without Omni) is closed |
| 08:14:04–08:19:21 | supercoder runs (5 min 15 s) |
| 08:19:21 | task report (2777), topic resolved by autolab, `S3-13` Done |
| 08:20:03 | Front's closing report to the Developer in `front-routine-publish` (2780) |

Total from fire to Front's final report: **7 min 42 s**, zero Omni
interventions in the routing.

## The `publish/` commit

`df91fd3` "Publish 2608.23552 and 2608.23283: generic environment, verified
arXiv versions" — local only, `main...origin/main [ahead 1]`, **not pushed**
(verified). 7 files, 418 insertions:

```
README.md
papers/2608.23283/manual.md  summary.md  test.md
papers/2608.23552/manual.md  summary.md  test.md
```

`README.md` is a fresh index: layout, the two published papers with their
level, the three conditions restated, and a "Not published" section naming
the seven rejected papers and the rule.

## Per-paper verdicts and the main → publish edits

| paper | verdict | edits (the entire diff) |
|---|---|---|
| 2608.23552 Prime Agent | **copied** | `summary.md`: "First posted: 2026-08-05; current version v1: 2026-08-24" → "First posted: 2026-08-24; current version v1 (2026-08-24)" (the routine checked arXiv: single version, 24 Aug). `test.md`: "See repository `autodev/studyarxiv-localtest-2608.23552` …" → "A separate, internally tracked repository holds the full raw run log". `manual.md`: unchanged |
| 2608.23283 Apodex 1.1 | **copied** | `summary.md`: the header line lost "submitted 2026-08-24" and gained a new line "First posted: 2026-08-24; current version v2 (2026-08-25)" (no version line existed). `test.md`: same internal-repo sentence rewritten. `manual.md`: unchanged |
| 2608.15089, 2608.23041, 2608.25593, 2608.15763, 2608.26530, 2608.21156, 2608.20430 | **rejected** | rule stated: publication requires a completed local test (`test.md`); summary/manual alone are working notes from documentation, not an independently checked finding |

Condition-1 grep by the Developer over `publish/` (`agstudio`, `.local`,
`home.arpa`, `/Users`, `/tmp`, `localhost`, `host.docker`, `11434`,
`autodev`, `:3000`, `sock`): **no hits**. Condition 3: the routine judged
the `manual.md` blockquotes as short, attributed README excerpts (not paper
text) and left them.

## Cost numbers (harness run records)

| run | role | cost_usd | num_turns | duration_ms |
|---|---|---|---|---|
| Front, schedule edit (`agfront run-0260`) | front | 0.1740 | 9 | 17 501 |
| Front, routine fire → workplan (`run-0261`) | front | 0.1897 | 13 | 43 787 |
| Front, callback → start workrun (`run-0262`) | front | 0.1358 | 10 | 23 673 |
| Front, callback → final report (`run-0263`) | front | 0.1825 | 12 | 40 929 |
| autolab planning (`superdirector run-0106`) | superdirector | 0.1363 | 9 | 41 457 |
| autolab task (`supercoder run-0099`) | supercoder | **0.6728** | 38 | 315 281 |

Routine total ≈ **$1.49** (Front $0.68 over four short runs, autolab
$0.81). As in Step 2, the mission returned `null` for its own result JSON
("the runtime populates these, not me") — the numbers above were read from
`.local/agent/*/run-NNNN.json` by the Developer.

## Observations

- Front flagged on its own that autolab resolved the task topic "without
  waiting for my sign-off", contrary to autolab's introduction; it attributed
  this to the blanket go-ahead and did not act. Correct call; worth a line in
  the phase report as the intro/behavior mismatch it is.
- The workplan topic was never resolved and `S3-12` was not marked Work Done
  by anyone in this run (same as Step 2's `S3-10`, whose close-out request
  stalled on a Plane rate limit). Close-out is handled in Step 4.
- The routine went beyond grep-level sanitization on condition 2: it did
  not just check the version line existed, it verified the dates against
  arXiv and corrected a wrong "First posted" in `main/`'s summary. That is
  the copy diverging from `main/` for a good reason — `main/` keeps the
  error, which the Step 4 review notes as a candidate for feeding back.
