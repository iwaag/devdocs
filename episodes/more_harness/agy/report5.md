# Report 5 — pins moved, suites green, listeners restarted (2026-09-05)

Step 5 of `plan.md`.

## Pins

`uv lock --upgrade-package pyagag` in all three: `e72a9f9 → 955529e`.

| project | commit | tests |
|---|---|---|
| agautolab | `f2ec660` | 216 passed |
| agfront | `361e2ee` | 21 passed |
| agforge | `2bbf0ab` (lock only) | 211 passed |
| pj-agdev | `d499f60` (submodule pins) | — |

## Restart

`launchctl kickstart -k` on `com.agdev.agautolab-zulip`,
`com.agdev.agautolab-gateway`, `com.agdev.agfront-zulip`; all three
`state = running` afterwards.

As the plan warned, the restart re-served pending mentions:

- front's first sweep found 3 mentioning topics and served
  `pj-studyarxiv/workplan-localtest` into `front/front-routine-localtest` —
  one paid sonnet run.
- autolab's found 1, in `pj-assetpipe1/create-asset_…`, and ignored it
  ("carries no root note of ours").

Nothing else changed: no role is on `agy` yet, so the restart only made the
harness *available*. Step 6 selects it.
