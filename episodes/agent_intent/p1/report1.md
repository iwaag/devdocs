# agent_intent p1 — Step 1 report: the DesiredAgent model

AI-generated (Omni Agent, 2026-08-17).

## What was built

`DesiredAgent` in nintent (`nautobot_intent_catalog/models.py`), plus the whole
path that makes it usable rather than merely present: the desired-state batch
writer, the read-only UI, the pinned nctl GraphQL read, and `nctl desired
export`.

### The model

Deliberately thin, in `DesiredService`/`DesiredWorkspace` style:

| field | type | note |
|---|---|---|
| `name` | CharField | |
| `slug` | SlugField, unique | the identity everything else keys on |
| `lifecycle` | CharField | the existing six-value enum, reused verbatim |
| `desired_workspace` | FK → `DesiredWorkspace`, optional, PROTECT | where the listener runs from |
| `desired_service_placement` | FK → `DesiredServicePlacement`, optional, PROTECT | how it is deployed, when it is a placed service |
| `zulip_user_id` | PositiveIntegerField, nullable | |
| `plane_user_id` | CharField, blank | blank until step 2 creates the account |
| `desired_zulip_channels` | JSONField (list) | channel names the agent must be subscribed to |

Two decisions worth naming:

- **Identity keys are numeric/opaque ids, never emails.** The Zulip realm hides
  real addresses from events (`pj-agdev/.local/devenv.md`, agforge precedent),
  so `zulip_user_id` is the join key and an email column would be a trap that
  looks authoritative and is not.
- **Liveness is not in the model.** It is observed, informational, and never a
  desired value. Only *registration* — accounts, memberships, subscriptions —
  is desired state here. Step 4/5 carry liveness as a freshness class outside
  this model, and it is deliberately not a gap.

Both foreign keys are optional because a `DesiredAgent` may be declared before
its workspace, placement, or accounts exist — that absence is precisely the
drift gap step 5 will report, so refusing to store it would hide the thing
being measured.

`clean()` rejects a `desired_zulip_channels` value that is not a list of unique
non-empty strings; a `jsonb_typeof(...) = 'array'` check constraint holds the
same invariant at the database boundary, mirroring the placement `config`
precedent.

### Writer

New batch kind `desired_agent`, identity `slug`, ordered last in `KIND_ORDER`
so its workspace and placement references resolve. The
`desired_service_placement` reference is a dict identity
(`{desired_service, instance_name}`), like `consumer_placement` on service
bindings; `desired_workspace` is a bare slug. Deleting a workspace or a
placement that an agent points at is now a planned conflict naming the agent,
not a runtime `PROTECT` surprise.

### Read side

`desired_agents` joins the single pinned desired GraphQL query and the typed
`DesiredSnapshot`, and `nctl desired export` emits the new kind. The exporter's
existing "every kind must appear" test is what forced this: an export that
omitted agents would be a backup that silently loses them.

### UI

`/plugins/intent-catalog/agents/` list and detail, filterset, table, nav entry,
and detail template — the same read-only shape as the twelve models already
there. Retained UI routes go 24 → 26.

## Deliberately not done in this step

- No `node_agent` profile binding revival. The plan marks it optional; nothing
  in p1 consumes an agent→LLM-provider edge, and reviving `PROFILE_BINDING_NAMES`
  for a consumer that does not exist would be speculative schema.
- No rows created. Enrollment (agforge first) belongs with steps 2–3, where the
  Plane account ids exist to put in them.

## Evidence

| gate | result |
|---|---|
| nintent Django-free fast (`python3 -m unittest discover -s nautobot_intent_catalog/tests`) | 147 tests, OK, 10 expected skips |
| nctl ordinary (`uv run pytest -q`) | 1315 passed |
| Nautobot runtime reuse (`./devtests/test_strategy/run_nautobot_runtime_gate.sh --keepdb`) | `cases=263`, OK |

The runtime gate also ran `makemigrations --check` against the exact local
source: **"No changes detected"** — the hand-written `0031_desiredagent`
migration matches the model, including the check constraint.

New tests, not just passing old ones:

- batch envelope: known fields accepted, an email-shaped unknown field rejected;
- batch apply: an agent bound to its workspace commits and reads back;
- an unresolved workspace reference is one conflict, not an aborted batch;
- deleting a workspace with an agent on it is blocked in the plan, naming it;
- the channel-list validation rejects a bare string, empty entries, duplicates,
  and non-strings;
- export: the agent operation's exact key and values, and the new kind count.

## Deployment

nintent `3abdc3c` and nctl `c5de139` pushed to GitHub. The Nautobot container
installs nintent from GitHub, so the deployment loop — rebuild, restart,
`nautobot-server migrate` — was carried through rather than left at the push;
see the migration evidence appended below.

## Live verification on the running container

- The rebuilt image reports `{"nintent_commit": "3abdc3c..."}` — the container
  runs the pushed commit, not a stale layer.
- `nautobot-server showmigrations nautobot_intent_catalog` ends
  `[X] 0031_desiredagent`; a re-run says "No migrations to apply".
- GraphQL answers `{ desired_agents { … } }` with `[]` — the type is exposed
  and there are, correctly, no rows yet.
- `/plugins/intent-catalog/agents/` returns 302 to the login page, so the route
  resolves (a missing route would 404 before auth).
- `nctl desired export` → `nctl desired apply -f` previews 81 operations, all
  `unchanged`, 0 conflicts: the export still round-trips with the new kind in
  the writer contract.

No `DesiredAgent` rows were created. Enrollment starts at step 3, once the
per-agent Plane accounts of step 2 exist to record.
