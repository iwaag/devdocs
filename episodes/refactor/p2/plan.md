# Refactor p2 — forge work in Zulip

## Goal and scope

Move forge's plan and execution records from Plane to Zulip. Prove that autolab can request an asset from forge, receive it, incorporate it into a project, and complete the request without Plane on that path.

Include the existing observation and completion surfaces needed for this workflow, plus the unfinished-work closure defect recorded in p1. Leave cagent change registration, nctl/Nautobot Plane identities, the legacy `tasks` view and gateway route, and Plane service removal for later phases. Front supervision is not required for the demonstration.

This is a breaking change in a private experimental environment. Old records need no migration or backward compatibility; disposable experimental work may be retired or deleted. Choose formats, module boundaries, naming, and task decomposition freely. Reuse p1's ideas where useful without copying its whole state model or building a generic workflow framework. Security hardening is outside scope. Keep credentials and machine-specific evidence in ignored files.

## Step 1 — Keep unfinished work reachable

- Fix ordinary completion so a blocked or failed Work acceptance also holds back closure of its dependent topics and dedicated channel. The selected parent request remains open. Independent finished branches may still close; a simple broader hold is also acceptable.
- Apply the dependency rule both when previewing and when a write fails during execution. Report what remains open and allow a later retry. Explicit cancellation, replacement, and disposal remain separate from successful completion.

Verify with focused tests: an unfinished task, acceptance failing during apply, and a successful retry. Confirm that the unfinished task's conversation remains usable and its channel is not archived.

## Step 2 — Make forge's conversations its work record

- Store the plan, selected toolset names, execution relationship, outcome, and result references in the `assetplan-` / `assetrun-` conversations. Keep explanatory content readable; use small explicit records for facts code must resolve without guessing.
- Replace Plane registration, plan lookup, result comments, and state transitions. Remove forge-only project selection, labels, credentials, and instructions that become unnecessary. Keep existing generation tools and conversational entrances unless simplifying them helps this change.
- Identify work and its origin with message anchors or an equivalent stable reference, not reusable topic names. Give each replacement a new identity. Support one practical retirement/replacement route: late replies belong to the old request, and a deleted origin is absent rather than the new request using its name.
- Keep asset bytes in the existing object store. Record the durable object key and the generation outcome in Zulip; temporary download URLs are delivery conveniences, not the only surviving asset reference.

Verify: plan and execute with Plane unavailable to forge, recover the plan and toolset selection from Zulip, and inspect the recorded result. Check success, failure, and a fresh attempt after failure without inheriting an old success verdict.

## Step 3 — Preserve asynchronous generation and delivery

- Keep submit → notifier callback → collection working across short agent sessions and listener restarts. Associate the pending job, generation workspace, and original requester with the stable work identity.
- Existing local `pending.json` / `watching.json` and generation files may remain execution state. Rebuilding all render state after deleting local files is not required. The plan, work identity, outcome, and delivered asset references belong in Zulip.
- Read the correct plan/toolset selection for the attempt being collected, even if the request has since changed. A replacement must not collect the old job as its own or receive its result accidentally. Deferring replacement while generation is pending is acceptable; forced job interruption is unnecessary.
- Preserve one effective result delivery to the requester and ordinary callback resumption of autolab. Duplicate callbacks or restart recovery should not submit another generation or produce duplicate completion notifications.

Verify with controlled fixtures: pending job followed by collection, restart while waiting, repeated callback, and rename/replacement with a late result. Expensive long-running generation is not required merely to test callback timing.

## Step 4 — Connect views and completion

- Teach the agent/operation room and shared completion to read forge's Zulip work record and follow the autolab → forge relationship. Reuse current views; show the plan, pending/finished outcome, and result access without requiring a Plane credential.
- Distinguish generation success, request acceptance, and conversation closure. Use forge's actual lifecycle rather than inheriting autolab's parent/child counting rule. Closing a request must not include its replacement or archive a shared agent/project channel.
- Remove the relay's Plane completion branch and configuration if forge was its last consumer. Retain shared Plane code still used elsewhere. Avoid preserving an unused abstraction solely for old work formats.

Verify in the UI: inspect and finish a combined autolab/forge request; unfinished generation remains reachable. Exercise acceptance with the actual human-writer identity in fixtures, not only the agent's identity.

## Step 5 — Deploy, demonstrate, and report

- Inspect service state through Nautobot or `pj-clusterintent/nctl`, then deploy changed packages and restart affected services using the ignored environment notes. Update the affected agent introductions and guides.
- Run one small development request through autolab's normal entrance. Let autolab ask forge for a useful asset, receive the result through the ordinary callback, incorporate it into the Git project, and finish. Inspect the artifact and accept the request through the existing UI. Messages needed for this bounded demonstration are part of the implementation work.
- Run relevant tests and the frontend build if changed. Demonstrate Plane independence with unavailable credentials, a rejecting client, or equivalent controlled isolation on the exercised path. Import checks are useful supporting evidence, not a complete proof of absent network calls. Plane may keep serving its remaining consumers.
- Write `report.md` covering changes, verification, live evidence, remaining limitations, and remaining Plane consumers. Distinguish controlled callback tests from live generation. Update developer documentation and ignored environment notes, then commit and push changed repositories and necessary submodule pointers.

## Implementation hints

- Under `pj-agdev/agforge/src/agforge/`, start with `plane.py`, `works.py`, `anchor.py`, `assetplan_topic.py`, and `assetrun_topic.py`. Registration stores both the plan and a `[TOOLS]` footer in Plane; execution reads it back. `FORGEAUTO` is no longer a queue-selection requirement, so do not recreate it without a use.
- `assetrun_topic.workspace_dir()` currently uses the Plane issue ID. `prepare_workspace()` preserves `watching.json`, `result/`, and `intermediate/` while refreshing other inputs. Change identity and plan lookup without losing the pending job's workspace or mixing separate attempts.
- The listener posts the ComfyUI watch command after the generator returns. `remembered_trigger()` preserves the original requester because the collecting run is triggered by the notifier. Preserve that distinction; the notifier is not the asset's recipient. The generator need not acquire its own chat tool.
- `deliver_to_origin()`, shared `[rootchat]`/`[served]` routing, and autolab's callback handling are the reference chain to inspect. Updating only forge's `[work]` note does not make a name-based return path follow arbitrary renames.
- `pj-agdev/agautolab/src/agautolab/worklog.py` and `anchor.py` show p1's message-anchor model. Reuse small shared primitives if helpful; forge does not need missions, task serials, or predecessor gates merely because autolab has them.
- Under `pj-agdev/agdevworld/agentroom/src/agentroom/`, `close.py:plan_actions` and `Closer._apply` currently allow topic/channel closure after a Work is blocked; only the final conversation is held back. `closing.py` has the remaining forge Plane discovery, and `main.py` installs the Plane client. `autolab.py` is an example of reading an agent's record without importing its package.
- Read p1's report for the acceptance-writer defect and existing replacement evidence. Some p1 comments call replacement “refactor p2”; replacement is already implemented in p1. Treat the code and recorded evidence as the starting point.
