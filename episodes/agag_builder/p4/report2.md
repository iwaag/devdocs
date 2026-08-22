# Step 2 report — provision the remote instance

Completed on 2026-08-22.

Implemented and published pyagag commit `558d4dd`:

- `agag provision --instance <name>` can select an instance without reading
  `<root>/.local/instance.toml`.
- `agag provision --out <path>` writes the generated bot environment to a
  controller-selected path; the default remains `<root>/.local/zulip.env`.
- `ProvisionResult` now reports the actual credential path.
- The init-time provisioning path explicitly retains the old local defaults.
- Tests cover an absent local instance file, the explicit remote instance, the
  explicit output path, mode `0600`, and rendered channel text.

Verification:

```text
tests/test_provision.py: 5 passed
pyagag main pushed to GitHub at 558d4dd
```

The first live invocation stopped before any Zulip write because the existing
agecho fixture predated `params/channel.md`. Added the current generated channel
description as agecho commit `c6e393e`, pushed it to GitHub, and retried.

The retry created the `agecho-agautolab1` bot and channel, subscribed the bot to
`#agents`, and wrote the ignored controller credential source at
`pj-agdev/.local/zulip/agecho-agautolab1.env` with mode `0600`. No credential
value was printed or committed.
