# Report 2 — `agents.toml`: `agy` profiles in agautolab and agfront (2026-09-05)

Step 2 of `plan.md`. Landed in **agautolab `f2ec660`** and **agfront
`361e2ee`** (each commit also carries the step 4/5 changes for that
project, because a profile on an unknown harness is `E_UNKNOWN_HARNESS`
under the old pin and the suites would not have been green in between).

## What landed

Both files declare two models and two profiles:

```toml
[models."antigravity/gemini-3.8-flash-medium"]
[models."antigravity/claude-sonnet-4-6"]

[profiles.agy]
harness = "agy"
model = "antigravity/gemini-3.8-flash-medium"

[profiles.agy-claude]
harness = "agy"
model = "antigravity/claude-sonnet-4-6"
```

The second profile is the plan's "if wanted": Agent ≠ Model, the record
says which ran, and a Claude model through the Antigravity account is a
different thing to measure than a Gemini one. No role moved; `sonnet` stays
the default everywhere. `GET /projects` lists profiles from the file, so
both show up with no gateway change.

## Proven

- `resolve_role(..., profile_override="agy" | "agy-claude")` through each
  project's real overlay resolves to `harness agy`, `provider antigravity`,
  the native model name, `command /Users/eiji/.local/bin/agy`, and an empty
  environment (the overlay's `google_api_key_file` does not leak in).
- agautolab's `test_the_agy_profiles_are_declared_and_resolve` pins that
  from the committed file.
