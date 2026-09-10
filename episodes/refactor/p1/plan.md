# Refactor p1 — autolab work in Zulip

## Goal and scope

Prove that one autolab development request can be planned, executed, revised, resumed, and accepted using Zulip as its work record, without Plane reads or writes on that path. Keep code and durable project documents in Git. Use this result to decide the next step toward retiring Plane.

Limit this phase to autolab's project/work lifecycle and the existing agdevworld views and completion actions needed to use it. Keep Plane running for forge, cagent, and other remaining consumers. Removing the whole Plane stack, migrating its history, and redesigning the UI belong to later phases.

This is a breaking change in a private experimental environment. Old work records and formats need no compatibility support or migration. Obsolete experimental work may be discarded or archived. Choose implementation structure, naming, record formats, and task decomposition freely; security hardening, a general task database, and a generic workflow engine are unnecessary. Preserve useful project source and documents unless replacing them is part of the work. Keep credentials and machine-specific details in ignored files.

## Step 1 — Make Zulip sufficient to execute autolab work

- Store the current plan and each task's executable description in their Zulip conversations. Include enough information to find the parent request, read results, and distinguish unfinished, completed, cancelled, and accepted work. Ordinary text is suitable for explanations; use small explicit records where code needs an unambiguous answer.
- Replace Plane-backed task lookup, plan read-back, result recording, and mission completion on the autolab path. Project initialization must also work without creating or finding a Plane project. Keep the existing Git workspace behavior.
- Keep the existing conversational planning and execution entrances unless a simpler replacement is useful. Update guides and tools so the agent knows where the current work lives and how to change it.
- Recover pending work from Zulip after a listener restart. Local caches and generated workspace files are disposable, not a second authoritative work ledger.

Verify: a fresh project and a multi-task request can be prepared and served with Plane access unavailable to autolab. Check that task text and results can be recovered from Zulip.

## Step 2 — Support replacement through conversation operations

- Support retiring an old plan/task conversation and opening its replacement. Renaming the old topic, resolving it, or archiving a dedicated work channel are available primitives. Deletion is acceptable for disposable work. There is no requirement to preserve a Plane-style task identity across revisions or implement every replacement operation in this phase.
- Make one replacement workflow usable by the agent. Record what the replacement carries forward and what it changes; completed work can be referenced rather than recreated as completed task rows.
- Separate conversation identity from its reusable display name. An opening message ID is a candidate anchor; channel IDs can identify dedicated work containers. Resolve references from the chosen anchor when needed. A late reply for an old task belongs to that task, even if a new topic uses its former name. A deleted origin is absent, not the new work with the same name.
- Retired work must leave the execution queue. Renaming or archiving does not itself stop a running process: the implementation may defer replacement until it finishes, or let it finish against the old work. Forced interruption is outside scope.

Verify with focused tests: replacement using a reused topic name, a late callback to the old work, restart recovery without duplicate serving, and a deleted origin. One supported replacement route is enough; no exhaustive matrix of Zulip operations is required.

## Step 3 — Connect observation and acceptance

- Adapt the existing agent/operation room and completion traversal to discover the new autolab work relationships and outcomes without Plane. Display the current plan, tasks, results, and replaced work sufficiently for a human to understand them; reuse existing UI.
- Keep execution success, human acceptance, and conversation closure distinguishable. Completing the selected request should close its relevant work, leaving replacement requests and shared project/routine channels outside that scope.
- Remove autolab-only Plane setup, tools, and documentation made obsolete by the change, including project retirement behavior. Keep shared Plane support where forge or cagent still uses it. Mixed requests may still need Plane for those agents; that is outside the proof for this phase.

Verify: inspect the request and its replacement in the existing UI, accept the finished request, and confirm the closure scope. New autolab work should not appear as broken merely because it has no Plane issue.

## Step 4 — Deploy, demonstrate, and report

- Inspect current service state through Nautobot or `pj-clusterintent/nctl` before deployment; use the ignored environment notes for startup details. Deploy changed packages and refresh affected listeners and agent introductions.
- Exercise a small development request through the normal conversation entrance: plan, execute, replace unfinished work, resume, report, and accept. A direct autolab request is sufficient; Front supervision and forge delegation need not be added to the live scenario. Agent-triggering messages in this bounded demonstration are part of implementing the plan.
- Run relevant tests and the frontend build if changed. Use controlled tests for late replies, deletion, and restart timing; distinguish those results from the live demonstration. Verify the autolab path without Plane access, using a client that rejects calls or equivalent isolation rather than stopping Plane for its other consumers.
- Write `report.md` with what changed, evidence, remaining limitations, and whether the new path is simpler enough to justify further Plane removal. Update relevant developer/agent instructions and ignored environment notes. Commit and push changed repositories and necessary submodule pointers.

## Implementation hints

- `pj-agdev/agautolab/src/agautolab/mission.py` currently owns Plane plan/task read-back, task reconciliation, predecessor checks, and result writes. `zulip_listener.py`, `anchor.py`, and `mission_done.py` consume this model. Replacing the model is preferable to reproducing every existing policy automatically; retain ordering only where the work needs it.
- `project_init.py` calls Plane even for a main-only project; inspect `project_archive.py` as well. Removing the obvious task API calls alone will not make autolab independent of Plane.
- `pyagag/src/agag/selfnote.py` stores `[rootchat]` as channel/topic text, and autolab's `[work]` stores a Plane issue ID. The existing handling of the `✔ ` prefix does not provide arbitrary rename tracking. Inspect shared callback routing, served markers, and execution-option inheritance when changing references. Keep shared changes limited to what this path requires; no legacy-format adapter is needed.
- In `pj-agdev/agdevworld/agentroom/src/agentroom/`, start with `ops.py`, `closing.py`, and `close.py`. Completion currently reads Plane children and writes Plane before closing topics. Preserve the useful relationship traversal while replacing the autolab data source.
- A renamed topic is not necessarily retired: listeners may still match its prefix or own its whole channel. Explicitly make the chosen retirement operation observable to the listener. Creating a topic with only the agent's own posts also does not necessarily start it; inspect the existing start behavior.
- Channel recreation needs subscriptions and folder placement, and a reused channel name must not determine identity. Whether an archived channel's name can be immediately reused has not been verified locally. Rename the old channel first or use a fresh name if simpler.
- Existing reports under `devdocs/episodes/agdevworld/front_desk/p4/` describe reused tasks, multiple parent references, resolved/open topic twins, and archived channels. These are useful test cases, not requirements to preserve the old architecture.
- Remaining Plane dependencies include forge plan/toolset storage, cagent change registration, and `pj-clusterintent/nctl/src/nctl_core/agent_registration.py`. Nautobot also declares Plane identities. Leave these for a later phase and record the boundary clearly; do not describe this phase as complete Plane removal.
