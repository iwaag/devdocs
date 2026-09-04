# Report 3 — overlays: where `agy` is (2026-09-05)

Step 3 of `plan.md`. Overlay files are ignored, so nothing to commit; this
report is the record.

## What changed

`agautolab/.local/agents.local.toml` and `agfront/.local/agents.local.toml`
each gained

```toml
[local.harness.agy]
command = "~/.local/bin/agy"
```

with a comment saying why: `~/.local/bin` is not on the three listener
plists' PATH, and `_resolve_command` expands `~`. The plists were not
touched — an overlay line is read per run, a plist change needs
`bootout`/`bootstrap` and a chance to get the asynchronous dance wrong
(devenv.md), and it would have re-served a pending mention once more.

## Proven

Resolution through the real overlays gives `command
/Users/eiji/.local/bin/agy` for both projects and both profiles (see
report2). Whether a launchd-started process can use the CLI's OAuth token
is step 6's question; the plan's `env -i` probe says it should.
