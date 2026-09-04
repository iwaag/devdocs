# Report 3 — overlays: where `codex` is (2026-09-05)

Step 3 of `plan.md`. Overlay files are ignored, so nothing to commit; this
report is the record.

## What changed

`agautolab/.local/agents.local.toml` and `agfront/.local/agents.local.toml`
each gained, directly under the agy entry,

```toml
[local.harness.codex]
command = "~/.local/bin/codex"
```

for the same reason: `~/.local/bin` is not on the listener plists' PATH,
and `_resolve_command` expands `~`. The plists were not touched.

## Proven

Resolution through the real overlays gives `command
/Users/eiji/.local/bin/codex` for both projects and both profiles (see
report2). Whether a launchd-started process can use `~/.codex/auth.json`
is step 6's question; the plan's `env -i HOME PATH` probe says it should.
