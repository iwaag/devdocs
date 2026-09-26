# give_context_easier p1 — Shared context repositories

Source: [braindump.md](braindump.md). Implementation hints: [preresearch.md](preresearch.md).

## Goal and working approach

Let the Developer create and manage shared context repositories from agdevworld, discover them in a collapsible Front Room panel, and insert a pinned reference into a conversation with one click. Every agent can discover and read the same published material without per-repository configuration changes.

- Reuse Gitea, the agentroom relay and pyagag's `agrefs`. Prefer a small shared catalog repository as the source of registration metadata; keep the content in its own repositories.
- This is a private experimental environment and a breaking phase. Backward compatibility is unnecessary. Replace existing interfaces, configuration and disposable state where useful. Avoid broad security hardening, blanket non-destructive rules and extra approval workflows.
- The implementer chooses schemas, API routes, UI composition, caching and migration details. The steps describe outcomes, not a mandatory internal workflow.
- Keep credentials and machine-specific facts in ignored configuration. Human-owned published references remain originals; agents read them and put derivatives in their own workspaces. Browser publication is an explicit Developer action.
- Earlier research contains alternatives and historical approval failures. Treat these as evidence and hints, not new prohibitions. This plan includes repository creation, basic editing and uploads; a link to Gitea alone does not complete that scope.

## step1 — Confirm the integration points and reference contract

- Read the prerequisite documents and research, then inspect the current implementations. Check local service state and observation timestamps through Nautobot or `nctl` before relying on environment notes.
- Identify all active agent instances and relevant roles, including Observer and cagent, plus remote execution and new-agent provisioning paths.
- Define catalog entries with a stable source ID, display name, description, repository URL, default branch and active/archived status. Keep display-name edits independent of reference identity.
- Retain `<source>@<commit>[:<path>]` as the portable reference. Resolve a selected version to a full commit ID; display a short version in the UI. Publication updates future selections, not references already posted or adopted by running work.
- Confirm the relay's Gitea creation/publication credential and human-owned destination. If setup is missing, make the required setup concrete while proceeding with independent implementation.

Done: ownership, discovery, publication and revision selection are clear, with concrete code entry points.

Hints: `pyagag/src/agag/refs.py` already implements immutable snapshots, text/search/tree reads and image paths. The relay's `settings.py` demonstrates a Git-backed catalog with retained revisions. `agautolab/src/agautolab/project_init.py` demonstrates idempotent Gitea creation, but its organization and credential belong to agent output.

## step2 — Implement shared discovery and reference resolution

- Add catalog loading to `agrefs` and expose the same read model to the relay. Configure the catalog location once per instance or deployment, rather than registering every content repository on every agent.
- Import the existing `protoprey-refs` registration. Local explicit sources may remain if useful, but compatibility with the old configuration is not required.
- Cache the catalog and content snapshots. Distinguish current, last-known and unavailable results; keep previously fetched pinned references usable during an outage.
- Define refresh behavior so a newly registered repository becomes discoverable without listener restarts or manual configuration edits. Fetch content on demand rather than cloning every repository for every run.
- Preserve archived entries for old reference resolution while excluding them from the default picker. Handle empty repositories, duplicate IDs and unavailable revisions explicitly.

Done: independent consumers discover a newly registered source and read identical bytes at a pinned commit.

## step3 — Give every agent access

- Integrate shared discovery into common agent configuration and provisioning, and distribute the catalog location through existing deployment mechanisms.
- Update tools and concise usage guidance for every relevant role. Include Observer and cagent; cagent has a separate tool implementation and needs an actual reference-reading tool, not only a configuration file.
- Make names and descriptions discoverable without injecting every repository's contents into every prompt. Agents choose relevant material and read it with the provided tools.
- Preserve source, commit and path through Front's plans, delegation, task inputs and reports. Reuse the existing reference guidance and records rather than introducing a second attachment protocol.
- Update pyagag dependency locks and install the new version in all consumers, including the relay and remote consumers in scope.

Done: each active agent has a working discovery/read path, and a newly provisioned instance inherits it. Record verification per instance and role; do not infer access from package installation alone.

Hints: `agag.agent.chat_environment` sets `AGREFS_HOME`. Four current consumers have separate `refs.toml` registrations. Front already carries resolved commits into requests, and autolab records adopted references in `direction/REFERENCES.md`. Binary metadata from `agrefs show` does not prove an image was viewed; use the harness's image reader.

## step4 — Add the context picker to the Front Room

- Add a show/hide context panel near the composer, with persisted visibility, search by name/description, update information and a refresh action.
- Clicking an entry resolves its version and inserts a reference at the saved caret or selection. Preserve the surrounding draft, restore focus and update the existing input/change state. Selection does not send a message.
- Provide README and file-tree browsing, with insertion of a specific file reference. Let the user explicitly refresh a selected reference to a newer version.
- Preserve Japanese IME behavior, keyboard navigation, narrow-screen usability and the current reply-target controls. Show loading, empty, stale and failure states without erasing the draft.
- Make the picker available in the Front Desk and the shared Arguing Room where the same composer is used; keep selection scoped to the current draft.

Done: a user can find a repository or file and add its exact reference to an existing draft without typing an ID or losing text.

Hints: `src/frontDeskInput.ts` is a DOM textarea and currently offers replacement via `set()`, not caret insertion. `FrontDeskScene.ts` serves both rooms through adapters. A DOM panel is a reasonable choice for search, selectable text and file browsing; no scene rewrite is required.

## step5 — Add creation and basic management

- From the panel, create a human-owned repository from a name, description and initial README; register it in the catalog after its first publication. Also support registering an existing repository.
- Support display-name/description edits, basic Markdown editing, file addition/upload, explicit publication and opening the repository in Gitea. Preserve ordinary Git authoring as another way to publish.
- Implement creation and publication as relay operations with appropriate local credentials, without requiring a model run. Record what the Developer published and return the resulting commit.
- Make retries finish a partially completed creation/registration instead of producing duplicates. Detect edits against an outdated base revision and present a recoverable conflict rather than silently overwriting a newer publication.
- Support archive/reactivate for catalog entries, retaining old reference resolution. Full repository deletion and a general-purpose browser IDE are outside p1.

Done: the user can create a context, publish text and an image, edit it later and manage its visibility entirely from agdevworld.

Hints: the current relay has no upload parser or Gitea write route. Reuse its server structure but choose a suitable upload representation. Initialize new repositories with a commit: an empty repository cannot produce a pinned reference. The research's credential status is historical and must be checked at implementation time.

## step6 — Verify, deploy and report

- Run focused tests for discovery/refresh, revision identity, archived and cached reads, interrupted creation retries, publication conflicts and unavailable sources. Reuse `pyagag/tests/test_refs.py` and the relay tests.
- Build the frontend and check the actual UI: panel toggling/search, caret insertion, multiple references, IME, file selection, upload/publication, failed requests and retained drafts. Demo/fixture routes can cover most interactions without agent runs.
- Deploy updated consumers and relay, rebuild the web image and verify service health. Use the local environment notes for deployment details and Nautobot/`nctl` for cluster state; report observation age and any unverified remote path.
- Prove one complete workflow: create a repository through the UI, publish text and an image, insert a pinned reference, have Front read it and delegate it to another agent, and verify that agent reads the same originals without a new per-source configuration entry.
- Publish a second revision and prove a new selection sees it while the earlier reference still reads the earlier bytes. Verify discovery/read access across the remaining active agents and an independently configured consumer.
- Update shared guidance and local deployment notes. Write `report.md` with evidence, deployed revisions, remaining gaps and any Omni Agent work performed for an in-system agent. Commit and push changed repositories and required submodule pointers.

Done: creation, management, discovery, UI insertion and cross-agent consumption are demonstrated together, with no outstanding per-repository registration work for the user.
