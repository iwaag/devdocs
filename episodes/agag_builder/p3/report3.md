# agag_builder p3 — step 3: `agag provision`

## Done

pyagag commit `7598532` implements and documents the agent-side provisioning
command:

- `agag provision [root]` requires an agag `agents.toml` and local
  `.local/instance.toml`.
- The owner-class identity is supplied as a path through
  `AGAG_ZULIP_ADMIN_ENV`; `--admin-env` overrides it. Secret values are never
  CLI arguments.
- Before any write it derives the expected bot delivery address from the
  instance and administrator realm, then refuses if that user already exists.
- For a new identity it creates the generic bot, atomically writes
  `.local/zulip.env` with mode 0600, subscribes the bot to `#agents`, and
  creates its own channel with bot + administrator subscribed.
- A new generated `params/channel.md` supplies the channel description with
  `{instance}` substitution; `--description` overrides it. An existing
  channel is joined idempotently and its description is updated.
- The successful output names the bot id, local credential path, channel
  result, and the intro/listener commands.

`agag init --provision` chains generation and provisioning. `--like <root>`
copies the sibling's ignored `.local/agents.local.toml`, which gives a new
listener the locally installed Claude command without embedding a host path
in tracked output.

The final printed human checklist now contains only:

1. the once-per-realm provisioner identity and its credential-path setup;
2. a per-agent Plane account, only when wanted;
3. permanent launchd/Ansible listener installation after the trial.

README's manual bot/channel instructions were replaced by the command, and
the p1 report's reproduction section now points to `agag provision` rather
than preserving the Developer-key recipe.

## Verification

- Focused init/provision/Zulip suite: 106 passed.
- Full pyagag suite: 411 passed.
- CLI help for `agag`, `agag init`, and `agag provision` rendered correctly.
- A generated trial copied agautolab's local harness overlay and contained
  the new channel-description template.
- Live read with the new provisioner found the existing agecho bot (user 16).
- A live guard trial against that identity exited 2, named the existing user,
  created no credential file, and made no Zulip write.
- `7598532` was pushed to GitHub for downstream consumers.
