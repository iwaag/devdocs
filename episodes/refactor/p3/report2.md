# refactor p3 step 2 — Plane leaves agent registration and desired state

## What was asked

Remove Plane credentials and member-list calls from `nctl agents observe`,
keep the Zulip observations and their drift checks, remove the Plane-specific
desired/observed fields and gap codes across nintent, nauto, nctl and the
desired import/export, and update the active desired-state inputs for the new
schema.

## What changed

### The collector needs one credential now

`nctl agents observe` read two realms: Zulip for the account and its
subscriptions, Plane for the workspace member list matched on
`plane_user_id`. `PlaneReader`, the `[plane]` config section, `PlaneConfig`
and `Config.require_plane` are gone, and `collect_agent_registration` takes
the Zulip reader alone. The text renderer used to end each line with
`plane=yes|no`; it now says `active=yes|no`, which is the fact that was
already collected and never shown.

The Zulip half is untouched: the account keyed on `zulip_user_id`, one
subscription probe per *desired* channel, and the rule that a desired channel
which does not exist in the realm reads the same as one the agent is not on.

### The fields and the gap codes went with it

| where | removed |
|---|---|
| `nintent` `DesiredAgent` | `plane_user_id` |
| `nintent` `ObservedAgentRegistration` | `plane_present`, `plane_user_id`, `plane_role` |
| `nintent` batch envelope, table, filter, detail template | the same names |
| `nauto` ingest payload contract | `plane_present`, `plane_user_id`, `plane_role` |
| `nctl` `DesiredAgent` / `ObservedAgentRegistration` / GraphQL selections | the same names |
| `nctl` drift | `agent_plane_membership_missing`, `agent_plane_identity_undeclared` |
| `nctl` desired export | `plane_user_id` in the allowlist and in the emitted operation |

Migration `0033_remove_plane_agent_identity` drops the four columns. Nothing
is migrated: this is disposable experimental state in a private environment,
as the plan says.

What deliberately stayed: `DesiredAgent` itself, `zulip_user_id`, the desired
channels, placements, workspaces and the service observations. Introduction
discovery is still where its consumers already read it; no introduction
registry was added to Nautobot.

### The active inputs

`nctl.toml` lost its `[plane]` section — the `Config` model is strict, so a
leftover section would have failed every command, not just the observation.
`.local/desired-state.yaml` lost its six `plane_user_id:` lines.

The `plane` **service** is still declared (a `desired_service`, its
`desired_endpoint` and its `desired_service_placement`). That is step 4's
scope: this step removed Plane from *identity*, not from expected services.

## Verification

The plugin was rebuilt from the pushed revision rather than the checkout —
`--build-arg NINTENT_BRANCH=2ff73f8`, because a plain `docker compose build`
keeps the cached `pip install git+…` layer — and the installed revision was
read back from the container:

```
{"nintent_commit": "2ff73f8e95cd9dc8c07b06733f4dcfecd2ed38ac"}
```

- **Migration.** `nautobot-server migrate` applied `0033`; `showmigrations`
  lists it `[X]`, and a second `migrate` says `No migrations to apply.`
- **Observation and ingest, with no Plane configuration and no Plane
  service read.** `nctl agents observe` printed seven agents and `ingested
  through the Nautobot Job`:

  ```
  agecho-agautolab1: zulip=yes active=yes channels=[agents]
  agforge: zulip=yes active=yes channels=[general, ops]
  agfront: zulip=yes active=yes channels=[front, general]
  arxivsage-agstudio1: zulip=yes active=yes channels=[agents, arxivsage-agstudio1]
  autolab-agautolab1: zulip=yes active=yes channels=[general, ops]
  autolab-agstudio: zulip=yes active=yes channels=[general, ops]
  cagent: zulip=yes active=yes channels=[general, ops]
  ```

- **Zulip registration problems are still reported.** `nctl drift` emits no
  code containing `plane`, and still marks `agforge` and `agecho-agautolab1`
  `drifting` on `agent_zulip_channel_unsubscribed`. Liveness stays
  info-severity and never makes an agent drift.
- **Export and re-apply under the new schema.** `nctl desired export` emits 7
  `desired_agent` operations with no `plane_user_id`; re-applying the exported
  document previews `unchanged: 93, conflict: 0`, and the operator input
  `.local/desired-state.yaml` previews `unchanged: 89, conflict: 0`.

**Suites and gates.** nctl 1335 passed. nintent Django-free 147 passed
(10 expected skips). nauto 121 passed. Nautobot runtime gate `--keepdb`
`cases=263` OK and `--clean` `cases=263` OK. Compute conformance passed.

## Unrelated drift, repaired and reported

`nctl` `tests/test_reconcile_profiles.py::test_real_repo_file_validates` was
already failing before this phase touched anything: it pins a profile-name set
by hand and `agag_agent` had been added to
`ansible_agdev/vars/deployment_profiles.yml` without it. It is a one-name
fixture, in a file this step was already editing, so it was corrected rather
than left red across the rest of the phase — but it is not this phase's work
and nothing else about profiles was changed.

## Limitations

- The `plane` service, endpoint and placement are still declared as expected
  state, and the Plane stack is still running. Step 4 removes both.
- `ansible_agdev/roles/autolab_node` and `roles/agag_agent` still pass Plane
  credential settings into a deployed agent; that is step 3.
- Historical migrations (`0031`, `0032`) and old reports keep their Plane
  terminology. They are not active dependencies.
