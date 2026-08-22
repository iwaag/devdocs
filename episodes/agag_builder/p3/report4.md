# agag_builder p3 — step 4: autolab provisioning capability

## Done

agautolab commit `2b2a0dc` gives every role run:

```text
AGAG_ZULIP_ADMIN_ENV=<pj-agdev>/.local/zulip/provisioner.env
```

This is the local credential **path**, never the provisioner key. The path is
derived from the agautolab checkout root, so no absolute host path entered a
tracked file. A test asserts the exact environment surface.

The supercoder guide now gives the capability in one line:

```text
agag init <name> --yes --provision --like <sibling-root>
```

and points at `agag --help` for usage rather than embedding a procedure.

agautolab's lock now resolves pyagag `7598532`. For one coherent agent set,
agforge `2c31c0e` and agfront `c7c4881` received the same lock update. All
three commits were pushed to GitHub, then the pj-agdev submodule pointers
were committed and pushed as `b6a7755`.

## Verification and deployment

- agautolab: 169 tests passed.
- agforge: 197 tests passed.
- agfront: 20 tests passed.
- Restarted the three launchd Zulip listeners after the GitHub-backed lock
  updates.
- All three launchd jobs reported `state = running` with new PIDs.
- Each listener registered fresh event queues and completed its startup
  sweep at 2026-08-22 12:34 UTC.

Deus Ex Machina note: updated and deployed autolab's provisioning capability
for autolab — handoff candidate.
