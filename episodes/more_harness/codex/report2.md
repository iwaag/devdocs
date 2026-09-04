# Report 2 — `agents.toml`: `codex` profiles in agautolab and agfront (2026-09-05)

Step 2 of `plan.md`. Landed in **agautolab `5a604d5`** and **agfront
`7089850`** (each commit also carries the step 4/5 changes for that
project, as with agy: a profile on an unknown harness is
`E_UNKNOWN_HARNESS` under the old pin, so the suites would not have been
green in between).

## What landed

Both files declare two models — with their effort, which is what the
harness turns into `-c model_reasoning_effort="…"` — and two profiles:

```toml
[models."openai/gpt-5.6-terra"]
effort = "medium"

[models."openai/gpt-5.4-mini"]
effort = "low"

[profiles.codex]
harness = "codex"
model = "openai/gpt-5.6-terra"

[profiles.codex-mini]
harness = "codex"
model = "openai/gpt-5.4-mini"
```

`codex-mini` is the plan's cheap one for `summarizer` trials. No role
moved; `sonnet` stays the default everywhere. `GET /projects` lists
profiles from the file, so both show up with no gateway change.

## Proven

- `resolve_role(..., profile_override="codex" | "codex-mini")` through each
  project's real overlay resolves to `harness codex`, `provider openai`,
  the native model name, `model_options {"effort": "medium" | "low"}`,
  `command /Users/eiji/.local/bin/codex`, and an empty environment (the
  overlay's `google_api_key_file` does not leak in).
- agautolab's `test_the_codex_profiles_are_declared_and_resolve` pins that
  from the committed file, effort included.
