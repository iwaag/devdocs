# Sage p2 — archsage can establish a study

## Goal and approach

Source: [braindump.md](braindump.md). Give archsage the tools and conversational workflow to establish the studies its sages need: research plan, project channel, study workspace and internal repository, routine guide, sage definition, knowledge connection, and an accurate completion report.

archsage decides the research substance and owns setup through completion. autolab prepares workspaces and performs research. Front runs routines. A request to create a study prepares it; a request to create and research also proceeds to an initial routine run. Existing studies and sages can be completed or connected without creating replacements.

This is a breaking phase in a private experimental environment. Backward compatibility is unnecessary. Replace obsolete paths and reshape disposable experimental state as useful. Choose command names, schemas, package boundaries, and implementation order freely; the names below are suggestions. No production security framework or elaborate preservation machinery is needed. Keep credentials and machine-specific configuration in ignored files, and retain the existing human review/push boundary for public study publication.

## Step 1 — Give archsage reusable project and routine tools

- Extract the reusable parts of `agfront/src/agfront/project.py` into pyagag, or use an equally small shared implementation. Keep Front able to open ordinary projects. Let archsage invoke study creation directly with the provisioner credential configured on its process; substantive posts use the calling agent's identity.
- Support opening a study, inspecting what exists, and continuing incomplete setup. Return channel IDs, document/setup message IDs, and remaining work. A retry should find the same artifacts rather than duplicate channels or setup requests. Reuse suitable existing studies on the agent's judgement.
- Resolve project subscribers explicitly: human owners, Front, autolab, the caller, and the board's Observer. The current helper assumes the caller is Front; granting it to archsage alone misses Front.
- Add routine creation and guide update operations, for example `agroutine create/update --guide-file ...`. Resolve the `routine` folder by name, create the public `routine-<name>` channel, and subscribe owners, Front, and the board's Observer. Post each guide revision as a complete new post in `guide`, under archsage's own authorship, and read it back.
- Explain each tool's purpose, inputs, outputs, and recovery in its help. Tools perform mechanical operations; archsage chooses domains, research questions, and guide content.

Verify new creation, existing-artifact inspection, and retry after a partial failure. Check guide read-back and board discovery, including delivery to listeners already running when a channel is created.

## Step 2 — Make study setup and its return path work

- Carry a machine-readable study pattern in the setup request. Have autolab recognize it before ordinary `init_project()` scaffolding, establish `README_PROJECT.md`, and prepare the study layout. Avoid relying on prose the agent reads only after initialization.
- Have autolab create the internal `main/` repository, research plan, knowledge README/index structure, and appropriate methods/reports locations. Record enough setup information to reconstruct the workspace marker. Public `publish/` can be attached later.
- Enable archsage to delegate, receive setup reports, and resume the originating conversation. Reuse `agentchat`, root notes, stable anchors, served receipts, and existing callback recovery. Its current mention handler ignores everything outside argues, so adding a send command is insufficient.
- Preserve the requesting conversation when setup starts from an argue. Decide whether to continue there or open a linked conversation in archsage's own channel; make the ownership and return destination explicit.
- Treat an accepted setup request as pending until autolab reports the actual repository and workspace. archsage then completes the remaining connections and reports the result to its requester. A run can end while waiting; it need not hold an expensive model session open.

Verify setup from an empty workspace produces the study layout without unwanted ordinary-project repositories. Exercise a normal callback, a callback after topic resolution, and restart while setup is pending. Completion must reach the original requester.

## Step 3 — Complete sage attachment, refresh, and persistence

- Add commands to attach a study to an existing sage and update its definition/guide. Record the study/project identity and whether its knowledge source is internal `main` or public `publish`.
- Default new studies to internal `main`, so knowledge becomes usable without waiting for a public repository or human push. Preserve existing intentional publication sources. Update prompts and documentation that currently describe every tree as published knowledge.
- Sync the tree on attachment and support requested refresh after research. Report the actual revision and whether knowledge is still empty. Reattachment must use the newly selected source: the current `sync_sage()` simply pulls an existing clone without checking that its origin matches the definition.
- Make runtime-created definitions and guides durable through Git, with commit/push and a documented reconstruction path. Keep internal connection details in ignored configuration or an appropriate private store. Choose the storage arrangement rather than blindly committing runtime files into the public code repository.
- Expose intro publication through a granted command and refresh the published sage list after a successful definition change.
- Let archsage inspect `tostudy/` and incorporate selected questions into research plans. Keep unresolved questions until the connected, refreshed knowledge actually addresses them. Queue deletion can follow that verification; sending a research request is not sufficient.

Verify new and existing sage attachment, source replacement, refresh failure, restored definitions, intro discovery, and answers citing files from the expected revision. A sage remains a knowledge reader with its study queue; archsage manages setup and synchronization.

## Step 4 — Teach the agents the complete workflow

- Update archsage's intro and guide: it can establish or complete a study, describe the resulting artifacts, and continue delegated setup to completion. Give examples of new creation, attaching an existing study, and adding a study for an existing sage.
- Grant the implemented project/routine, delegation, sage management, and intro commands, install them on the runtime PATH, and supply their usage information. Do not substitute a long mandatory procedure for tools and clear responsibility.
- Update Front's entrance, desk, and argue guidance to route study establishment to archsage. Front continues to own routine execution. Remove the outdated claim that project-channel creation always requires the Developer.
- Write routine guides that give Front the research goal and enough context to ask autolab for a bounded mission. Follow current requester acceptance and integration behavior; older episode reports predate those fixes.
- Distinguish setup completion, research completion, and refreshed knowledge in reports. When initial research is requested, carry it through Front's routine flow and refresh the associated sage afterward.

No scheduler or automatic queue-draining service is part of p2. Research selection remains conversational. Confirm the normal request can progress using the installed tools, without Omni Agent filling gaps by hand.

## Step 5 — Deploy, prove the three entry cases, and report

- Inspect service state through Nautobot or `pj-clusterintent/nctl` and refresh stale observations before live validation. Reuse the experimental services, update affected dependency pins, and reload affected listeners. New logical sages need no separate agent accounts or services.
- Run focused tests for the changed operations and routing. Include partial setup/retry, pattern initialization, callbacks across restart, source replacement, and guide read-back. Verify that registering a routine alone does not start research.
- Exercise three cases through ordinary agent entrances: a new study, an existing study missing its sage/routine connections, and an existing sage missing its study. Candidates from the preliminary research are `aisvgs`, `worldtrend`, and `growbox`; inspect their current state before choosing them.
- For at least one case, request an initial bounded research run, follow it to requester acceptance and integration, sync the tree, and ask the sage a question answered by the new findings. Record conversation IDs, repository revisions, and cited knowledge files as evidence. A setup-only case should remain correctly reported as ready but not researched.
- Check that the new routine appears on the existing board and the new sage is discoverable through the intro. Record any manual intervention as a handoff candidate rather than treating it as proof of autonomous completion.
- Update developer documentation and ignored environment notes. Write step reports where useful and a final `report.md` covering delivered behavior, evidence, remaining gaps, and deferred queue automation. Commit/push changed repositories and necessary submodule pointers.

## Implementation pointers

- [preresearch.md](preresearch.md) inventories the existing studies, commands, and historical failures. Recheck its observations; it is context, not an additional rulebook.
- `pj-agdev/agfront/src/agfront/project.py`: channel creation, `setup_request`, root notes, and research-plan posting. It currently refuses an existing channel and has no continuation path.
- `pj-agdev/agautolab/src/agautolab/project_init.py`: `README_PROJECT.md` suppresses ordinary scaffolding. Runtime callers in `zulip_listener.py` initialize before the model reads the request.
- `archsage/src/archsage/listener.py`, `roles.py`, `cli.py`, and `sages.py`: own-channel dispatch, argue-only mention handling, run context, and current add/sync operations. `params/intro.md` and `agent/guides/archsage/guide.md` describe the old responsibility.
- `pyagag/src/agag/zulip.py`, `topics.py`, and `listen.py`: channel/folder helpers and shared conversation/recovery behavior. Reuse the mirror; older advice to restart after every subscription may no longer apply.
- `pj-agdev/agdevworld/agentroom/src/agentroom/routines.py`: discovery by public `routine-` channel prefix; the newest `guide` post is the complete guide, and a resolved guide retires the routine. Guide posts can be truncated by Zulip, making read-back useful.
- `pj-agdev/agautolab/agent/project_pattern.md`: the current study/publication contract. Public repository creation is not required for an internal study; public publication still follows the existing review and human push workflow.
- The preliminary research found runtime sage definitions left untracked and studies incompletely connected. Use these as evidence for persistence and resume support, not as a reason to build a general provisioning framework.
