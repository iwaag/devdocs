# Report 6 — proof: front, the gateway window, and one workrun on `agy` (2026-09-05)

Step 6 of `plan.md`. Everything the plan asked to see was seen, in about
four minutes of wall clock, all through the listeners started in step 5
(launchd, not a shell). Timestamps below are UTC; the local clock was past
midnight, 2026-09-05.

## Front on `agy` — `#front > front-agy-trial`

`[roles.front] profile = "agy"` uncommented in agfront's overlay, one post
as the Developer (message 4836, 15:54:39). Front's reply landed at 15:57:33:
it named itself and the agents it can reach (autolab, agforge, arxivsage,
agecho, agping), correctly and without contacting anyone. The wait was the
restart backlog — three pending mentions served first on the old config,
one of which (`work-s3-1/workrun-task1-s3-5` → `front-routine-localtest`)
already ran on agy as `run-0542`, `done`, 97 s.

`agfront/.local/agent/front/run-0543.json`:

| field | value |
|---|---|
| harness / provider / model | `agy` / `antigravity` / `antigravity/gemini-3.8-flash-medium` |
| outcome | `done` |
| duration_ms | 20847 |
| num_turns | 1 |
| usage | input 67337, output 2346, thinking 1862, cache_read 32496 |
| cost_usd | absent |

So the OAuth token is reachable from a launchd-started process, as the
plan's `env -i` probe predicted; no key plumbing was needed.

## Gateway window on `agy` — `POST /window`

`[roles.front] profile = "agy"` in agautolab's overlay, one `curl` to
`:8791/window`. `window/run-0018`: `outcome: done`, 19.4 s, `num_turns 1`,
`usage` present, `is_error: false`, and a two-sentence answer that read
the README and described the node. The gateway needed no change.

## One `workrun-` task on `agy` — `pj-runsmoke2`

`[roles.supercoder] profile = "agy"` in agautolab's overlay. A one-task
mission was opened in `pj-runsmoke2 > workplan-agy-trial` (planning ran on
`sonnet`, as configured); autolab opened `work-r2-4/workrun-task1-r2-4`,
and one post there started the task at 15:56:22. It was reported done at
15:58:02:

- `main/AGY_TRIAL.md` exists **in the project workspace** with the two
  requested lines, committed as `f79e1f0 agy trial`, pushed to Gitea.
  Nothing new in `~/.gemini/antigravity-cli/scratch/` (still only the
  Omni Agent's `c.txt`, `e.txt` from the morning probes) — `--add-dir <cwd>`
  did its job.
- **The progress display worked through the translation**: the topic shows
  sixteen `🔧` lines — `view_file: <path>`, `run_command: git -C main …`,
  `write_to_file: …/report.md` — and one `💬` line, exactly the claude-shaped
  form autolab's `RunProgress` reads. `view_file`'s path came through, which
  means the guessed `AbsolutePath → file_path` mapping was right.
- `agautolab/.local/agent/supercoder/run-0216.json`: `harness agy`,
  `outcome done`, 93.8 s, `num_turns 1`, usage input 195733 / output 12764
  / thinking 9416 / cache_read 976199, no cost.
- The 20-minute timeout became `--print-timeout 1190s`; the run needed 94 s,
  so the deadline itself was not exercised.

Two observations, not harness problems:

- **`num_turns` is 1 for every agy run**, however many tools it called: agy
  counts user messages, not model calls. The plan's "`num_turns`" is
  recorded as agy reports it; a reader comparing it with claude_code's
  (15–21 for similar runs) should know the units differ.
- **The supercoder closed its own task** ("task R2-5: commented yes, Done
  yes; resolving this topic") instead of waiting for the requester's
  acceptance. The mission text said "this is my explicit go-ahead; do not
  ask again", which it seems to have read as acceptance in advance. Worth a
  look under `sonnet` before blaming the model; noted, not pursued.

## The failure path

A deliberately bad model through the real CLI and `run_harness`
(`antigravity/gemini-9.9-nonexistent`, `--mode plan` so the argv carried no
bypass flag): exit 1, record `outcome: failed`, `failure: "agy exited 1:
invalid model selection (--model "gemini-9.9-nonexistent" …): … Available
models: …"` — the catalog quoted, `num_turns 0`, `duration_ms 0`. (Run from
this shell rather than a listener, because the harness's own classifier
blocks `--dangerously-skip-permissions` in argv; the listeners are not so
constrained, as the three runs above show.)

## The parameter keys, for the record

From the CLI's conversation store after the workrun: `run_command`
carries `CommandLine` and `Cwd`; `view_file` `AbsolutePath`
(+`StartLine`/`EndLine`); `write_to_file` `TargetFile`/`CodeContent`;
`list_dir` `DirectoryPath`; `grep_search` `SearchPath`/`Query`;
`find_by_name` `SearchDirectory`/`Pattern`; `read_url_content` `Url`.
pyagag's `AGY_PARAMETER_KEYS` already had all of these; its comment now
says they are seen rather than guessed (pyagag follow-up commit after
`955529e`).

## State left behind

- Both overlays are back to `sonnet`; the agy lines are commented next to
  the gemini ones, ready to uncomment.
- `pj-runsmoke2` has mission R2-4 (Done) and `work-r2-4`; harmless smoke
  residue, like the earlier `work-r-8`.
- One extra `front` sonnet run and one agy run were paid for the restart
  backlog, as step 5 warned.
