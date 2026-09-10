# Refactor p3 — retire Plane

## Goal and scope

Remove the remaining operational dependence on Plane and remove its experimental deployment. Keep cagent's change requests in its own Zulip channel, use Zulip bot IDs for agent identity, and retire the old `tasks / plane` screen.

The division of responsibility is:

| information | authority |
|---|---|
| cagent change request and its progress/result | one topic per request in cagent's own channel |
| an agent's public contract and entrance | its `intro-<instance>` topic in `#agents` |
| conversations with that agent | the channel its introduction advertises |
| messaging identity | the Zulip bot user ID |
| desired placement and expected service state | Nautobot |
| actual process/service health | existing service observations |

An account, channel, or introduction existing does not prove that a listener is running. Keep registration and service health distinct.

This is a breaking change in a private experimental environment. Old Plane issues, APIs, and formats need no migration or compatibility layer. Obsolete experimental records and Plane-owned deployment data may be deleted. Choose implementation details freely; a replacement task dashboard, generic registry, new authorization framework, and strict non-destructive workflow are unnecessary. Keep credentials and machine-specific details in ignored files.

## Step 1 — Move cagent change records into its own channel

- Replace the Plane registration of `requested_change.md` with a readable record in a cagent-owned topic. Reuse that topic for discussion, decisions, and results. When a request originates elsewhere, retain a reference and return a link to the record; a request already in the appropriate topic need not create a duplicate.
- Identify the request and any return destination with message anchors or equivalent stable references. Renames and reused names must not redirect an old result into a new request. A deleted origin is absent. Use small explicit records only where code needs them; a ticket number and autolab-style task hierarchy are unnecessary.
- Make the channel and topic route discoverable in cagent's introduction and usable by its listener. Posting a registration or replaying the same serving should not recursively create requests or buy repeated agent runs.
- Preserve the current distinction between recording a desired change and executing a cluster change. This phase relocates the record; it does not add automatic reconciliation to registration. Existing observation and incident-reporting routes should continue to work.

Verify: register a request, read it back from the channel, continue the conversation, and find its recorded disposition without Plane. Use focused tests for repeated registration, rename/deletion, and a request that also asks for observation.

## Step 2 — Remove Plane from agent registration and desired state

- Remove Plane credentials and member-list calls from `nctl agents observe`. Retain the existing Zulip account/channel/subscription observations and their useful drift checks. Missing or inactive expected bot accounts should remain visible.
- Keep `DesiredAgent`, Zulip bot IDs, desired channels, placements, and service observations. Remove Plane-specific desired/observed fields and gap codes across nintent models, migrations, forms/tables, API/GraphQL schemas, ingest, nctl readers, and desired import/export.
- Keep introduction discovery in its existing consumers. Add intro checks to nctl only if a concrete need arises; this phase does not require a new introduction registry or a mirror of each posted contract in Nautobot.
- Update active desired-state inputs and existing scratch data for the new schema. Old Plane identity values can be discarded. Historical migrations and reports may retain Plane terminology; they are not active dependencies.

Verify: agent observation and ingest succeed with no Plane configuration or service; Zulip registration problems are still reported. Check migration and desired export/reapply under the new schema, and confirm that no Plane identity gap is emitted.

## Step 3 — Retire obsolete UI and integration code

- Delete the old `tasks / plane` view, navigation entry, `planeState.ts`, and its dedicated dispatch/state-change code. Existing agent/operation room views remain the way to inspect work; no replacement dashboard is needed.
- Trace callers before removing gateway code. Remove only the obsolete Plane-specific route or assumptions, retaining unrelated conversational windows, evidence access, and service functions.
- Remove `agag.plane` once no current consumer needs it, along with its dedicated tests/dependencies and unused configuration. Update package pins and deployment templates so reinstalling an agent cannot restore Plane credentials or requirements.
- Inspect older deployed checkouts, including the node left unchanged in p1/p2. Update or retire any remaining Plane-dependent path; do not assume the local source tree represents every deployment.

Verify: relevant suites and the frontend build pass; navigation has no Plane task screen. Search active source, configuration, startup jobs, and deployment templates for remaining consumers, distinguishing product dependencies from historical text and generic “control plane” terminology.

## Step 4 — Stop Plane, prove independence, and remove its deployment

- Inspect desired and observed state through Nautobot/nctl and read the ignored local environment notes. Identify the Plane service, placement, endpoints, startup mechanism, and dedicated storage. Update the desired declarations through the supported desired-state workflow so Plane is no longer expected.
- Deploy the changed packages and Nautobot schema, refresh affected services and introductions, then stop Plane. Run the verification below while it is unavailable; a missing Python import alone is not sufficient evidence.
- With Plane stopped, register and inspect one cagent change request, run agent registration observation/ingest and drift, and exercise a small autolab → forge → autolab request through completion. Inspect the existing UI and resulting artifact. Use a harmless cagent request whose registration does not require an actual infrastructure change; record it as such. Messages needed for this bounded demonstration are part of implementation.
- Resolve remaining Plane calls found during these checks. Then remove the Plane containers, dedicated volumes, startup/deployment files, and obsolete credentials. Identify stack ownership before deleting resources: other services' PostgreSQL, Redis, or object storage remain in use. No mandatory archive of disposable Plane data is required.
- Confirm that Plane is absent from the deployment and expected-service state, that startup configuration will not recreate it, and that current consumers no longer require it. Report unrelated drift separately rather than expanding this phase to repair the cluster.

## Verification and handoff

Run affected component suites and the applicable gates in `pj-clusterintent/README_DEV.md`, including Nautobot runtime/migration verification for the schema removal. Use focused tests for reference and retry behavior; repeat expensive live generation only when a failure warrants it.

Update developer documentation, agent guides/introductions, provisioning instructions, and ignored environment notes. Write `report.md` with removed dependencies, stopped-service evidence, deletion scope, verification results, and any remaining limitations. The phase is complete when Plane is removed and the retained workflows operate without it. Commit and push changed repositories and necessary submodule pointers.

## Implementation hints

- `pj-clusterintent/cagent/src/cagent_api/topics_serve.py:handle_handoffs` currently registers `requested_change.md` before processing `required_info.md`; an exception during registration can prevent the observation branch. `plane.py` is the current adapter. Inspect listener routing and guides alongside the replacement record writer.
- `pj-clusterintent/nctl/src/nctl_core/agent_registration.py:observe_agents` requires Plane configuration and fetches workspace members. Also inspect `config.py`, `sources/{desired,actual}.py`, `desired_export.py`, and `drift/{agent_evaluation,comparators}.py`.
- `pj-clusterintent/nintent/nautobot_intent_catalog/` contains `plane_user_id` in desired and observed models and their surrounding schemas/UI. Follow the actual ingest path and fixtures rather than removing only the model fields.
- `pj-clusterintent/ansible_agdev/roles/autolab_node/` still passes Plane credential settings into the shared agent role. Search that role and other provisioning paths too. p1/p2 removed application dependencies but deliberately left deployment and registration cleanup.
- `pj-agdev/agdevworld/src/views.ts` imports `planeState.ts`; that module calls `/api/plane/*` and builds an old gateway mission request. The existence of this frontend code does not establish that its backend route still works. Remove obsolete callers and inspect the real gateway before choosing its cleanup scope.
- p1/p2 provide message-anchor examples in autolab's `worklog.py` and forge's `record.py`. Share useful primitives without imposing either agent's lifecycle on cagent. If multiple readers consume a record, test the same writer identities and state rules in each.
- The Nautobot plugin is installed into the local container from Git, not mounted from the checkout. Follow `.local/localenv_memo.md` for revision-aware rebuild and migration, and verify the installed revision. The Plane stack is documented in `pj-agdev/.local/devenv.md`; keep those machine facts out of tracked documentation.
