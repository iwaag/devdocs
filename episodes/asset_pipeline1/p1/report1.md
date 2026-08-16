# Report 1 — Repair stale guide references (agautolab)

Status: **done**. `agautolab` suite green (78 passed, unchanged count).

Commits `7889c4a` → `a468621` moved every guide to `<dir>/guide.md` and
rewrote the contents, but no call site followed. Because the tests monkeypatch
`GUIDES` at a tmp directory and write the guide file they expect, the suite
stayed green while every real mission-/run- serving died with `GuideError`
before its agent run. That gap is the reason this step is verified against the
**real** guide tree rather than against the suite alone.

## What changed

`src/agautolab/zulip_listener.py`, three guide paths:

| call site | was | now |
|---|---|---|
| `front_prompt` | `mission_front/guide_mission_topic.md` | `mission_front/guide.md` |
| `run_coding` | `mission_coding/guide_task_split.md` | `mission_superdirector/guide.md` |
| `run_work` | `run_coding/guide_run_coding.md` | `run_coding/guide.md` |

The middle one is a placeholder by design: the `mission_coding` guide
directory no longer exists at all, and Step 2 replaces this call site with a
superdirector run in the project folder. Pointing it at the guide that
survived keeps the repository in a state where nothing raises `GuideError`,
which is what "repair" means for this step.

`tests/test_zulip_listener.py` — the `wire()` helper and
`test_front_prompt_carries_autolabs_own_extra_line` /
`test_guide_refuses_to_start_without_the_file` now write and expect
`mission_front/guide.md`.

## `devlogs/` → `devlog/`

`project_init.init_project()` clones `<project>-devlog` into `devlog/`, and
both superdirector guides told the agent to look in `devlogs/`. An agent
following that instruction finds nothing and concludes the project has no
history — the guide's own words for "the project has just started". Aligned on
the directory that actually exists:

- `agent/guides/mission_superdirector/guide.md`
- `agent/guides/create_answer_superdirector/guide.md` (untracked before this
  step; it is the Step 6 guide, committed here so the `devlog/` fix is not
  split across two commits)

## Verification

The suite (78 passed) proves the call sites still compose their prompts. What
proves the *repair* is loading each path against the shipped guide tree with
`GUIDES` left alone:

```
('mission_front', 'guide.md')             404 bytes  OK
('mission_superdirector', 'guide.md')     962 bytes  OK
('run_coding', 'guide.md')                190 bytes  OK
('create_answer_superdirector', 'guide.md') 406 bytes OK
```

`guide()` raises on a missing *or empty* file, so a non-zero length is the
whole contract.

## Noted, not fixed

`zulip_listener.main()` catches `ZulipError` at line 425 but never imports it
— the `except` clause would raise `NameError` if the initial channel
subscription refresh ever failed. Out of this step's scope; recorded here so
it is not lost. (Step 2 touches this module heavily and can absorb it.)
