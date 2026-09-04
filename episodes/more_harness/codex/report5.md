# Report 5 — pins moved, suites green, listeners restarted (2026-09-05)

Step 5 of `plan.md`.

## Pins

`uv lock --upgrade-package pyagag` in all three: `955529e → 9f6797e`.

| project | commit | tests |
|---|---|---|
| agautolab | `5a604d5` | 218 passed |
| agfront | `7089850` | 21 passed |
| agforge | `2ceb949` (lock only) | 211 passed |
| pj-agdev | `bf9da25` (submodule pins) | — |

All pushed. Each project's venv answers `codex` in
`STREAMING_HARNESSES` and `HARNESS_PROVIDER["codex"] == "openai"`.

## Restart

`launchctl kickstart -k` on `com.agdev.agautolab-zulip`,
`com.agdev.agautolab-gateway`, `com.agdev.agfront-zulip`; all three
`state = running` afterwards (16:50:43Z).

As the plan warned, the restart re-served pending mentions:

- front's first sweep found 1 mentioning topic and served
  `pj-studyarxiv/workplan-publish-2026-08-29b` into
  `front/front-routine-publish` — one paid sonnet run.
- autolab's found 1, in `pj-assetpipe1/create-asset_…`, and ignored it
  ("carries no root note of ours"), as last time.

Nothing else changed: no role is on `codex` yet, so the restart only made
the harness *available*. Step 6 selects it.
