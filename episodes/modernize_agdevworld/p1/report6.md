# p1 step 6 (unplanned) — nintent desired state, and the assistant's identities

Not in `plan.md`; asked for after step 5 landed. Two halves: declare `agfront`
in nintent so the cluster knows the Front agent exists, and finish retiring the
assistant that p1 deleted — the accounts as well as the records.

**Result: `nctl drift` reports `drifting=0`.** The five remaining `unknown`
rows are agpc's stale service observations, unrelated and pre-existing.

## agfront in nintent

Four rows added to `.local/desired-state.yaml` and applied:

| kind | key | notes |
|---|---|---|
| `desired_service` | `agfront` | active |
| `desired_service_placement` | `agfront-agstudio` | `manual_toolchain`, `management_mode: manual`, `process_pattern: agfront\.zulip_listener`, **no endpoint** |
| `desired_workspace` | `agfront` | `iwaag/agfront.git`, agstudio, `pj-agdev/agfront` |
| `desired_agent` | `agfront` | `zulip_user_id: 15`, channels `[front, general]`, plus the Plane id below |

**No endpoint**, because the listener has no port. `observation.py` branches on
`management_mode == "manual"` and uses `process_pattern` *before* it ever looks
at an endpoint, so a portless process is modelled honestly rather than given a
fake port. All 23 pre-existing placements carry an endpoint; the field is
nullable and nothing objected.

### The two-pass apply

A single apply fails: the `desired_agent`'s placement reference is planned as
`conflict: unresolved desired_service reference: 'agfront'`, and one conflict
blocks the whole transaction.

`_PlannedReferenceResolver` *does* accept references to rows created earlier in
the same batch — but only at the top level. Resolving the placement reference
`{desired_service: agfront, instance_name: …}` goes through `_find` →
`_orm_values`, which resolves the **nested** `desired_service` against the
database and raises before the planned-upsert fallback is reached.

So: apply once with `desired_service_placement: null` (create 4), then restore
the reference and apply again (update 1). The file is left in its final,
correct form. Worth knowing before anyone declares another agent — it is a
property of the batch planner, not of this data.

### Reconcile

`nctl reconcile agstudio` planned **zero actions**, which is correct: nctl
plans nothing for a `manual` placement, and only its disappearance is drift.
The first drift run said `agfront: service_missing` purely because the node
observation predated the placement; `--yes --refresh-observation` fixed it.

Final state — service converged, workspace converged
(`presence=present identity=matched activity=idle freshness=fresh`), agent
converged with `liveness=polling`.

## agfront's Plane account

Created with the documented ritual (`.local/plane/plane_agent_account.py`,
`AGENT_SLUG=agfront`), user `0905f3e3-69a6-4372-9e4d-e78d165cba48`, workspace
role 20 like its siblings, credentials at `.local/plane/agfront.env` (0600).

Front does not touch Plane today. The account exists because nintent's
registration evaluation counts a missing Plane identity as the gap
`agent_plane_identity_undeclared` — the model's position is that an agent has
one, and an agent with no declared identity cannot be told apart from one whose
account went missing. That gap is now gone.

### A credential exposure, and its rotation

The creation script prints the new password and API token. The command that
ran it tried to redact them with `sed 's/^\(A=\|B=\).*/…/'` — BSD `sed` does
not support `\|`, so **both secrets were printed** into the session transcript.

Rotated immediately: `.local/plane/plane_agent_rotate.py` deactivated the
exposed token, minted a fresh one, and reset the password. Verified — the new
key answers `200`, the exposed key answers `403`. The values never reached a
tracked file or a persistent log, and the account was minutes old on a local
scratch Plane.

The lesson is not "be careful with sed": it is that a script printing a secret
to stdout invites this. Both scripts now say in their docstrings to send the
output **straight to the env file**, never to a terminal.

## Retiring devworld-assistant

- **Zulip** — bot user 10 deactivated (`DELETE /api/v1/bots/10`). Zulip offers
  no hard user deletion; deactivation is what retirement means there.
- **Plane** — `.local/plane/plane_agent_retire.py`, the new inverse of the
  creation script: 1 API token deactivated, 17 project memberships, 1 workspace
  membership, user `is_active=False`. Verified: its key now answers `403`.
  Nothing is hard-deleted, because Plane issues and comments carry the user as
  their author — removing the row would either cascade into that history or be
  refused by it.
- **Credential files** — `.local/zulip/devworld-assistant.env` and
  `.local/plane/devworld-assistant.env` deleted.
- **`provision-realm.py`** — the bot removed from its `bots` tuple, with a
  comment saying why. Re-running it would otherwise recreate the identity that
  was just retired.
- **Desired state** — one `op: delete` batch (`.local/retire-devworld-assistant.yaml`)
  removed four rows: the `devworld-assistant` agent, the
  `agdevworld-assistant-agstudio` placement, the `agdevworld-assistant`
  service, and its `agstudio:8091` endpoint. Deletes run in reverse
  `KIND_ORDER`, so all four resolve in one transaction.
- **Actual state** — no separate step needed. `ObservedAgentRegistration` has a
  `CASCADE` one-to-one on `DesiredAgent`, so the observation row went with it,
  and the fresh node observation no longer reports a service that is not
  declared.

The `agdevworld` **desired_workspace stays**. The frontend repository is still
there and still declared; only the agent is gone.

## Left alone

`nctl drift` still reports five `unknown` rows on agpc
(`ace-step`, `comfyui`, `music-gen`, `swarmui`, and the node itself) with
`service_observation_stale`. Pre-existing, unrelated to this episode.
