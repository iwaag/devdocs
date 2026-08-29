# Step 2 — backfill the two verified papers

## Mission text (Developer, `#pj-studyarxiv › workplan-testmd-backfill`, message 2735)

> Mission: backfill graded `test.md` for the two papers with completed local tests. One task is enough.
> 
> Autolab's project pattern (`agent/project_pattern.md`, "Repository-backed local tests") now defines `main/papers/<id>/test.md` — the distilled, publishable result of a local test — and a four-level scale: `L1` built locally and most basic function confirmed; `L2` a paper-described workflow completed end to end; `L3` a small original verification experiment measured performance; `L4` a reproduction of the paper's experiments or beyond. Read that section first.
> 
> Do, in `main/`:
> 
> 1. Write `main/papers/2608.23552/test.md` (Prime Agent) and `main/papers/2608.23283/test.md` (Apodex 1.1) from the existing run logs in `localtest-2608.23552/report.md` + `localtest.yaml` and `localtest-2608.23283/report.md` + `localtest.yaml`. Each `test.md`: level reached, upstream repository and revision tested, environment in generic terms only (e.g. "Apple-silicon Mac, local Ollama, model `<name>`" — no hostname, path, port, or credential placeholder), the evidence for that level, and what a later run would have to do to raise the level. Do not link into `localtest-<id>/`; name its repository (`autodev/studyarxiv-localtest-<id>`) instead, so the file survives being copied out of `main/`.
> 2. Assign the levels honestly. Both tests ended with a one-shot smoke command returning a fixed token; if that is all the evidence supports, both are `L1` — do not inflate.
> 3. Repurpose the `localtest` column of `main/papers/INDEX.md` to carry the level: change the two `verified` cells to the level you assigned, leave the `no` cells, and rewrite the column's description line to say `no`, `L1`…`L4`, or an in-progress `localtest.yaml` state.
> 4. Also update `README_PROJECT.md`'s `publish/` line to match the pattern: an edited copy produced by the publish routine, `main/` stays intact, never pushed by autolab.
> 5. Commit once and `git push origin` (origin is already `agstudio.local`; do not type out a URL).
> 
> Report in the task topic: the two `test.md` contents, the INDEX diff, and quote your own harness result JSON — `cost_usd`, `num_turns`, `duration_ms` — as the last lines of the report. Do not run or re-run any local test.

## Plan and run

autolab (superdirector) answered at 07:55:32Z (message 2740): one Work
`S3-10`, one sub-work `S3-11`, both papers pre-assessed as **L1** in the
plan itself ("no paper-described workflow was exercised"). Approved
(2741); the task was started by posting into
`work-s3-10 › workrun-task1-s3-10` (2743). The supercoder ran 07:55:54Z →
08:00:01Z and resolved the topic itself (2748/2749).

## The two `test.md` files as landed (`main` commit `9931cfa`, pushed)

Both follow the same shape — level line with the reason it is not L2,
upstream repo + revision, generic environment, evidence, what would raise
the level, and a closing sentence naming the localtest repository instead
of linking to it:

- `papers/2608.23552/test.md` (30 lines): **L1**; `PrimeIntellect-ai/prime-agent`
  release `v0.8.1` (tag commit `5146337…`); "Apple-silicon Mac, local
  Ollama, model `qwen3.8:27b-mlx-bf16`"; evidence = installer + `--version`
  = 0.8.1 + one-shot `--offline --no-session --no-tools` prompt returning
  exactly `OLLAMA_PRIME_OK`, exit 0; to reach L2 = run one of the paper's own
  `ipython`-tool long-horizon workflows end to end.
- `papers/2608.23283/test.md` (27 lines): **L1**; `ApodexAI/FrontierAgent`
  at `2b82a43…`; same generic environment; evidence = dependency setup,
  documented Docker image build, one bounded ReAct turn returning exactly
  `FRONTIER_OLLAMA_OK` with `stopped_by: no_tool`; to reach L2 = a genuine
  multi-turn tool-using task.

Grep of both files for `agstudio`, `.local`, `home.arpa`, `/Users`, `/tmp`,
`localhost`, `host.docker.internal`, `11434`: no hits. Honest L1, as expected.

## INDEX diff

Two cells `verified` → `L1` (2608.23552, 2608.23283); the column
description rewritten to "`no` if there is no local test, `L1`…`L4` once
`main/papers/<id>/test.md` records the level …, or an in-progress state
from that repository's `localtest.yaml` (for example `waiting_external`)".
No other row touched. `README_PROJECT.md`'s `publish/` bullet was also
rewritten to the edited-copy wording (that file is outside every
repository, so no commit).

## Cost numbers (harness run records, `agautolab/.local/agent/*/run-*.json`)

| run | role | cost_usd | num_turns | duration_ms |
|---|---|---|---|---|
| planning (`superdirector/run-0104`) | superdirector | 0.2419 | 13 | 190 565 |
| approval turn (`superdirector/run-0105`) | superdirector | 0.0510 | 4 | 7 644 |
| task (`supercoder/run-0098`) | supercoder | **0.4136** | 33 | 240 418 |

Mission total ≈ **$0.71**, ≈ 7.5 min of model time, wall clock 07:52 →
08:00Z.

## Frictions

1. **The mission cannot quote its own result JSON.** The task said plainly:
   "the harness result fields … aren't values I can compute myself — they're
   appended by the harness after the run completes", and left placeholders.
   The p7 recommendation ("have missions quote their own cost") is therefore
   not satisfiable by mission text alone; the numbers live in
   `.local/agent/<role>/run-NNNN.json` and were read from there by the
   Developer. A listener-side change (append the result meta to the Zulip
   report) is the real fix — out of scope here, noted for the phase report.
2. The run spent its first ~8 tool calls searching the workspace and then
   the filesystem for `agent/project_pattern.md` (found at the agautolab
   checkout). The pattern doc is not in the project workspace; the mission
   named it by relative path. Cheap to avoid next time by saying where it is,
   or by letting the guide point to it.
3. No push-auth friction: `main`'s origin was already `agstudio.local`
   (set during the p7 follow-up) — "main was already level with Gitea".
4. No permission-classifier stops on Omni's `agentchat` calls this step.

## Close-out

Mission close-out (resolve the workplan topic, mark `S3-10` Work Done) was
requested from autolab in its own channel (`#autolab-agstudio1 ›
closeout-s3-10`, message 2750) rather than by running `mission_done` as
Omni — the p7 report's second Deus Ex Machina, avoided this time.
