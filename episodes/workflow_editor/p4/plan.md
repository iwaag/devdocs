# Workflow editor p4 — system integration with autolab alone

## Goal

This plan follows the [migration braindump](../design/migration/braindump.md) and the design agreed upon after its revision.

Build on the experimental Project, Workflow, and Run model so that a person can request development through agdevworld, have a single autolab agent perform research and implementation with human checkpoints, and save the results and work records to Gitea. Continue development of both the experimental RTS project and the editor itself through this path.

Integrate incrementally: establish the storage contract, resource management, single-agent execution, and then use on real projects. Treat the supervised IDE execution demonstrated in p3 separately from the process management, resumption, and agdevworld operations introduced here.

## Scope and exclusions

Implement the following in p4:

- Both ordinary-directory and submodule storage for devdocs, with an ordinary directory as the default for new projects.
- The new run layout: `devdocs/runs/<workflow-id>/<run-id>/`.
- Repository storage and sharing through Gitea, including creation and registration of existing repositories.
- An agdev dashboard showing all projects and shared repositories.
- Adding existing shared repositories from the Project Editor.
- Execution of study, do, and talk by autolab alone, including human waits and explicit resumption after interruption.
- Requests, progress inspection, answers, stopping, resumption, and result review through agdevworld.
- Registration of the RTS and editor projects under the new contract, followed by one real improvement run for each.

The following are out of scope:

- Integration with agfront, agforge, or archsage, and execution by multiple agents.
- Delegate execution, delegation across projects, and execution control for parent and child runs.
- Actual Zulip notification delivery or inbound integration, and Observer integration.
- Representing agents as a form of project.
- Reading or converting old formats, preserving old URLs, or synchronizing with legacy missions and tasks.
- Changing a project's devdocs storage mode after creation.
- Automatic deployment or discovery across hosts, and distributed scheduling.
- Unconditional automatic re-execution of interrupted work, and OS-level access isolation.

Do not maintain backward compatibility. Experimental run histories need not be migrated; register the target projects again under the new contract. Preserve the source code, artifacts, and knowledge needed for continued development. Dropping compatibility does not require implementing bulk deletion of existing data.

## 1. Responsibilities and sources of truth

| Area | Responsibilities and retained information |
| --- | --- |
| Project / Workflow / Run | Git-backed definitions and work records, their validation, and their updates |
| autolab | Agent launch in a workspace, and management of waits, exits, interruptions, and resumption |
| agdevworld | Global listings, editing, requests, answers, and result review |
| Gitea | Repository sharing and storage of published history and results |
| Execution host's local storage | Queue, execution attempts, process information, detailed logs, and pending publication state |
| Zulip, later | Coordination records that enable the next action: acknowledgment, questions, blockers, completion, and similar exchanges |

Devdocs is the source of truth for work records. Do not independently manage run state in Zulip or the registry. Saved files are the current work record even before publication to Gitea; display their unpublished status separately.

Use the experiment's models, validation, reducer, and run operations as the shared implementation. Build on the existing service and CLI, with autolab invoking the same operations through the CLI. Do not duplicate state-transition rules in Python or the browser. The agdevworld views use the same service.

Do not translate workflow runs into the existing autolab mission/task model. Connect migrated projects to the new execution entrance, ensuring that legacy listeners cannot execute the same request again. Compatibility with the old views and operating model is not required.

## 2. File contract

```text
project/
  project.yaml
  devdocs/
    workflows/
      build-game.yaml
    runs/
      build-game/
        run-001/
          braindump.md
          plan.md
          definition/
            build-game.yaml
          run.json
          report*.md
  study/
  wedo/
  .local/
```

- `braindump.md` preserves the person's original words. Do not replace them with an agent's summary.
- Retain the distinction between human input and an agent-authored `request.md`, including its provenance. An entrance for requests from external agents is not part of p4.
- `plan.md` and reports hold detailed explanations. Do not infer execution state from their prose.
- `definition/` holds a fixed copy of the executed definition. Changes to the source definition do not alter an ongoing run.
- `run.json` holds state, questions and answers, decisions, artifact references, and ordered history. Preserve the existing contract that replaying history produces the stored current state.
- Identify a run by Project, Workflow, and Run together. Machine-local absolute paths are not durable identifiers.
- Update path creation, listing, watching, artifact references, history views, CLI help, guides, and fixtures to the new layout. Do not discover or convert the old layout.
- Update schemas and operation contracts. Show old formats and broken records as unsupported or invalid, never as empty listings or successful states.

### Two devdocs storage modes

Declare `directory` or `submodule` explicitly in `project.yaml`. New projects default to `directory`. Report a mismatch between the declaration and the actual Git structure; do not convert it automatically.

Both modes use `devdocs/...` paths in the working tree. History reads and publication use a shared resolver to identify the Git repository that owns devdocs and the path within that repository.

- Directory: the owner is the project root repository, and the repository-relative path is `devdocs/...`.
- Submodule: the owner is the devdocs repository, and paths are relative to its root.
- Historical references must identify both the owning repository and the commit. Never silently fall back from historical content to current files.
- In submodule mode, support resolving the recorded gitlink from a root commit and make the displayed revision explicit.

To a workflow, devdocs is a logical documentation area; it need not be an independent repository. Adjust existing binding validation and resource views accordingly.

Interpret access declarations as path scopes. Writing a run's own records is a reporting capability available even with `root: readonly`. This exception does not grant permission to edit source code, other runs, or workflow definitions. Restrictions remain agent self-checks and must not be described as OS-enforced permissions.

## 3. Single-agent execution and resumption

### Supported nodes

| Node | Treatment in p4 |
| --- | --- |
| study | autolab performs research and records its findings |
| do | autolab performs implementation, validation, and other work |
| talk | Save a question, wait for the person, and resume after receiving an answer |
| delegate | Definition and display only; execution is unsupported |

Reject execution of workflows containing delegate nodes during preflight checks. Show the unsupported capability in the UI, and reject it in shared operations so that another entrance, such as the CLI, cannot start or delegate the work. Do not skip delegate nodes or mark them complete without execution. Design cross-project target identification, resolution, and parent/child run contracts later, separately from this execution foundation.

Use one autolab execution slot, processing one job at a time. Persist additional requests in a queue and execute independent branches sequentially. A join waits for every predecessor to succeed; failure and cancellation do not count as completion.

### Separate workflow runs from execution attempts

One workflow run can span multiple agent launches. Local execution management retains each launch's identifier, start and end times, process state, and log references. Devdocs records the durable facts and reasons needed to understand interruptions and resumption.

- When talk enters a human wait, confirm that the question is saved and end the execution process. Other eligible runs in the queue can execute.
- Persist an answer before enqueueing resumption. An ordinary answer schedules resumption; stopped or cancelled runs do not resume automatically.
- Keep answer recording, agent uptake, node completion, and result acceptance as distinct operations.
- On resumption, reread the fixed definition, run record, plan, artifacts, and working tree. Do not depend on the previous conversation session surviving.
- Retransmitted requests or answers and service restarts must not launch the same execution twice. Define receipt identifiers and recovery procedures for queue registration.
- Multiple executions must not edit the same workspace concurrently. Initially, reject starting another run in a workspace already held by a run awaiting a person; only execute eligible runs in other workspaces without that conflict.

### Interruption and stopping

Separate recorded work state from process state. `running` records that work started; it does not prove a process is alive. The UI distinguishes executing, awaiting a person, interrupted, and unknown.

Detecting an unexpected exit or an attempt whose status became unknown after a service restart must not automatically complete, fail, or rerun a node. Side effects may already have occurred. Inspect the working tree and records, then continue from an explicit resume instruction.

Distinguish stopping execution, which is resumable, from cancelling a run, which discontinues the work. A stop request alone must not appear as confirmed process termination; show the request until termination is confirmed. Reject updates and resumption after cancellation, and prevent a stopped, superseded attempt from updating the new attempt's records.

## 4. Updates and publication

Even with one agent, browser answers can race with CLI updates. Protect each run's read, expected-version check, state transition, and atomic write with shared mutual exclusion. It must work across the CLI and service processes; a version check alone does not replace locking.

Use the shared API/CLI for routine run operations. Direct external edits to `run.json` are not a safe concurrent-update route. External editing of definitions and Markdown remains available, with writers taking turns on the same file. Do not unconditionally overwrite unsaved UI drafts or artifacts being produced by an active execution.

File saves and Git publication are separate operations. Initial publication checkpoints are human waits, a safe point after stopping or interruption, completion, and result acceptance. Execution management must not begin publication while the process is still writing.

- Use Gitea for new projects and shared repositories. Remove local bare repositories as remote substitutes from actual operation.
- Track Gitea creation, initialization, push, and registration as separate steps so a partial failure can resume after completed steps.
- If a repository with the same name already exists, distinguish intentional reuse from a collision; do not overwrite unrelated content.
- Commit and push submodules first, then publish the root containing their references.
- Display work completion separately from publication success. On publication failure, retry publication without repeating the work.
- Review the intended changes before publication; do not indiscriminately include unrelated pre-existing changes. Show conflicts and inability to publish as requiring attention.
- Do not embed credentials or machine-specific paths in project definitions or run records.

## 5. Resource management and views

### Managed entities

| Entity | Meaning |
| --- | --- |
| Project | A development unit with a purpose and workflows |
| Repository | A Gitea storage unit, categorized as root, devdocs, study, wedo, or another kind |
| Workspace | A Project checked out into a working location on a particular host |

Add a small persistent registry on the service side. Store Project identifiers, Repository identities in Gitea, and Workspace associations. Do not determine identity solely from URLs or display names. Support registering Projects without a workspace and shared Repositories that no Project uses yet.

Project files remain authoritative for definitions, `.gitmodules` for submodule membership, and gitlinks for pinned revisions. Do not create independent editable copies of those facts or run state in the registry. Any cached usage relationships for listings must carry their source and observation time; unavailable information must not be reported as empty or unused.

Display directory-mode devdocs as a Project resource without counting it as an independent Repository. List a shared Repository once globally, with navigation to every Project that uses it.

### agdev dashboard

- Projects: purpose, workspaces, ongoing runs, human waits, and execution-host availability.
- Repositories: category, description, consuming Projects, and Gitea links.
- Entrances for creation and registration, and navigation to the Project Editor.
- Distinguish read failures, Projects without a workspace, and stale observations. An outage must not appear as a valid empty listing.

### Project Editor and run view

- The Project Editor displays the selected Project's resources, workspaces, workflows, and runs.
- “Add existing repository” searches the global registry, accepts a destination path, and adds the repository as a submodule. Reject collisions with existing paths.
- Do not change another Project's gitlinks. Each Project explicitly adopts shared Repository updates.
- Starting a run specifies the workflow, execution workspace, and human input.
- The run view displays the fixed definition, node states, questions and answers, artifacts, execution attempts, and unpublished status.
- Provide distinct actions for answering, stopping, resuming, cancelling, and accepting or rejecting results.
- Preserve file watching and rereading after reconnection. Progress updates must not discard unsaved definition edits.

Specify the connection between agdevworld and the editor service, the route to execution APIs, and workspace ownership. Establish a same-origin route and the access boundaries appropriate to the existing deployment instead of directly exposing a service designed for loopback use. Do not expand this into an API for arbitrary path operations across hosts.

## 6. Boundary for future Zulip integration

Specialize Zulip in coordination records. Messages must contain enough information for the recipient to decide the next action; merely shortening posts is not the goal. Reducing posting frequency is not a requirement.

Future notifications cover acknowledgment, questions, blockers, completion, and similar events, carrying Project, Run, and question identifiers with references to details. Notify only after the devdocs record is durable. Keep delivery success separate from work state, and never repeat work because delivery failed.

In p4, provide only the boundary for selecting notification-worthy events from committed record updates, using stable history-entry identifiers and event types. Do not implement Zulip sending, receiving, or a delivery retry queue. Future answers received through Zulip must retain the speaker and source-message reference in devdocs and use the same answer operation.

## 7. Implementation sequence

### Stage 1: Update the storage contract

1. Inspect current models, CLI/API operations, watchers, historical reads, and guides to identify the changes required.
2. Update contracts for the layout, schemas, devdocs modes, and historical references.
3. Support creation, validation, reads, writes, and history access in both modes.
4. Disable delegate execution entrances and clarify their separation from editing and display.
5. Update fixtures and tests to the new contract without adding old-format compatibility code.

Completion criteria: Projects can be created in both modes, and runs can be recorded and viewed in the new layout. Multiple commits reconstruct the definitions and state from their respective points in history. Delegate execution is rejected before starting.

### Stage 2: Gitea and global resource management

1. Inspect existing autolab Gitea integration and establish reusable boundaries for authentication and creation.
2. Implement creation, existing-repository registration, recovery after partial failure, and publication status.
3. Implement the Project / Repository / Workspace registry.
4. Implement the agdev dashboard and adding shared Repositories in the Project Editor.
5. Establish the route from agdevworld to the editor views and service.

Completion criteria: Global listings and per-Project listings serve distinct purposes. Both storage modes work in fresh workspaces obtained from Gitea. Adding a shared Repository does not change another Project's pinned revision.

### Stage 3: Execution by autolab alone

1. Implement process-wide coordination across CLI and service processes, using per-run locking and version checks for shared run updates.
2. Implement request receipt, the persistent queue, one execution slot, workspace ownership, and execution attempts.
3. Prepare the agent execution guide and CLI path, and connect study, do, and talk.
4. Implement resumption after answers, stopping, cancellation, interruption detection, and explicit resumption.
5. Connect publication checkpoints and retries after publication failure.
6. Complete the agdevworld UI for requests, answers, execution status, and result review.

Completion criteria: A synthetic task proceeds from a request through a human wait, process exit, answer, resumption, and completion. Service restart and unexpected interruption remain distinguishable, and recovery causes neither duplicate launches nor automatic repetition of work.

### Stage 4: Continue RTS and editor development

1. Inventory the source code, artifacts, and knowledge to retain from experimental workspaces, and the records that will not be migrated.
2. Inspect the current code and Git boundaries to determine each Project root. A separate repository for the editor is not mandatory; align the boundary with its actual development and build unit.
3. Save the necessary repositories to Gitea and prepare Projects and Workflows under the new contract.
4. Register workspaces obtained from Gitea and confirm that the source and artifacts are usable.
5. Execute one improvement task for each project through agdevworld, as specified or agreed upon by the person. Record the actual answers and result acceptance.
6. For changes to the editor itself, separate the running service from the source being edited. Update the service after building and validating the changes, and confirm that ongoing runs can be displayed again.

Completion criteria: A real improvement run completes for each of the RTS and editor projects, with human review, results and records in Gitea, and successful reads from a fresh workspace. Synthetic tests alone do not complete this stage.

## 8. Validation

Run tests of the changed contracts and integration tests first, followed by the relevant existing type checks, tests, and builds. Use dedicated data, registries, and repositories for browser and Gitea validation. Never reset the person's experimental workspace as a fixture.

Verify at least the following:

1. Creation, layout, bindings, historical reads, and retrieval from Gitea in both directory and submodule modes.
2. Definition changes after run creation do not alter active or historical runs.
3. Listing, watching, and artifact references use the new layout; old formats and invalid records receive explicit diagnostics.
4. Delegate execution is rejected consistently in the browser, API, and CLI.
5. Concurrent answers and progress updates, stale-version writes, and duplicate submissions do not lose records.
6. Talk questions, answers, uptake, completion, and result acceptance remain distinct.
7. A process does not remain running throughout a human wait, and eligible work in another workspace can proceed.
8. Service restart, abnormal exit during execution, immediate resumption after stopping, and delayed updates after cancellation cause neither duplicate execution nor false completion.
9. Gitea creation failures, push failures, and root-publication failures after a child Repository was published can be retried without repeating the work.
10. Projects without a workspace and unused shared Repositories appear in listings, and retrieval failures are distinguishable from valid empty results.
11. Adding or updating a shared Repository does not alter another Project's references.
12. Browser reconnection, run switching, unsaved drafts, and publication-state changes are handled correctly.
13. Real tasks for both RTS and the editor complete through agdevworld operations.

Interruption tests must include a process stopping after a work side effect but before recording completion. Rather than spending hours in an actual wait, verify recovery from persistent state across process exit and restart.

Use progress visibility within about two seconds in a normal local environment as a usability target. Record observation intervals and measured latency under a representative workload; do not claim a guarantee for all conditions.

## 9. Reporting and p4 completion criteria

Record each stage's implementation scope, validation, actual commands, remaining limitations, and the next stage's entry point in `report*.md` in this directory. Distinguish synthetic evidence from actual human-led development. Summarize completion status and operating instructions in the final `report.md`.

P4 is complete when all of the following hold:

- The new devdocs contract and Gitea operation work in both storage modes.
- The dashboard and Project Editor have distinct roles, and resources can be managed globally and within an individual Project.
- Autolab alone supports requests, work, human checkpoints, stopping and resumption, result review, and publication.
- One development task for each of RTS and the editor has been completed through agdevworld, and its results and records can be retrieved again.
- All of this works without front, forge, archsage, Zulip integration, or delegate execution.

If real tasks or human review remain outstanding, report foundation implementation as complete separately from real use remaining incomplete. Do not expand p4's completion criteria by adding future multi-agent coordination ahead of its own design phase.
