# Workflow editor p12 — project Boast

## Objective and agreed scope

Implement [braindump.md](braindump.md): give each project a freely authored
showcase inside the Project Editor. The entry point is `boast/index.html` in
the project root. WFE supplies space and serving; projects own the content,
appearance, interactions and operation.

For this phase, add a `boast` workflow to the managed RTS and Playprey projects
and delegate to it near the end of their practical work workflows. This is
an explicit choice for these two projects, not a requirement for all projects.

The shared introduction is limited to:

> `boast/index.html` is the page displayed in the Project Editor's Boast area.
> It can freely introduce the project's capabilities and outputs using HTML,
> CSS and JavaScript. Related files can live in the same directory.

Do not accompany this introduction with creation obligations, update schedules,
required sections, recommendations about content, or workflow instructions.
Keep optional technical reference separate. Do not add Boast tasks to shared
agent guides, generated project templates, or unrelated projects.

## Working conventions

- Follow `localrule.md`: generated files are English; communication with the
  Developer is Japanese. Implementation, commits, pushes, deployment and
  agent-operable verification proceed without added human approval steps.
- Deliver a deployed result that the Developer can inspect with few actions.
  Serve it to other PCs on the home LAN, not only localhost.
- Backward compatibility is not required. Reuse current mechanisms where useful
  without retaining obsolete restrictions or introducing an approval process.
- Keep machine addresses, host paths, credentials and raw runtime evidence in
  ignored local files. Tracked documentation uses portable paths and identities.
- Read current service state through pj-clusterintent's Nautobot or `nctl`
  before runtime work. Use `pj-clusterintent/nctl/README.md` for the commands.
- Respect explicitly stopped runs and preserve unrelated work. Reconcile this
  plan with current p11 code: older delegation documentation still describes
  approval gates that the current integration contract has removed.

Writing this plan does not execute the implementation or start project runs.

## Starting points

Paths below are relative to the projects directory. Resolve managed project
locations and identities from the current registry. The inspected RTS workspace
uses project id `rts-vs-bot`; verify that identity before changing workflows.

| Owner | Starting point |
| --- | --- |
| Project Editor | `pj-agdev/agdevworld/workflow_editor/src/projectView.ts`, `src/styles.css` |
| Committed and worktree reads | `workflow_editor/server/access.ts` within agdevworld |
| HTTP and media serving | `workflow_editor/server/api.ts`, `server/presentServer.ts`, `server/shares.ts` |
| Change notifications | `workflow_editor/server/watch.ts`, `src/live.ts` |
| Run ownership and integration | `workflow_editor/docs/{storage,integration,delegation,shares}.md` |
| Release | `pj-agdev/agdevworld/scripts/wfe-release.sh`, agdevworld's `README_DEV.md` |
| RTS workflows | `devdocs/workflows/{build-game,improve-game}.yaml` in the managed project |
| Playprey workflows | `devdocs/workflows/{onboard,compare}.yaml` in the managed project |

## Step 1 — implement the page contract and serving

1. Serve `boast/index.html` and its relative assets. A self-contained HTML file
   works without configuration. Separate CSS/JS/images and prebuilt static
   applications work too; WFE does not build project applications on demand.
2. Default to published `main`, using the existing project access/cache layer.
   Resolve a revision once and serve the HTML and its repository assets from
   that same commit. Expose fetch failure and stale cache truthfully.
3. Offer an explicit preview of the selected workspace's working tree. Keep
   preview and published content distinguishable; a workspace may lag behind
   integration, and a child's copy does not contain its parent's unpublished work.
4. Add Boast-specific static serving with correct MIME types, relative URL
   resolution, cache/revalidation behavior and byte ranges for media. Reuse or
   extract suitable primitives from presentation serving rather than starting a
   temporary presentation server for every project.
5. Serve only the selected content root, resolving traversal and symlink escapes.
   Load page scripts inside an iframe rather than WFE's document. Start with a
   sandbox that supports the intended scripts, links and downloads without giving
   embedded code access to the editor DOM or its write APIs. Check direct-tab
   serving as well as embedding; separation must not disappear when opened alone.
6. Support shared-area images, videos and downloads through an appropriate
   display route. Existing download-only responses are not sufficient for inline
   media. Resolve `shared://` references using host-local configuration; keep
   payloads outside Git where appropriate. Do not depend on temporary Run
   presentation URLs for the persistent showcase.

Deliverable: serving implementation and an optional technical reference describing
paths, publication sources, media and browser capabilities, without operating
instructions for projects.

## Step 2 — add the Project Editor surface

1. Place Boast below the project heading/summary, across the available width in
   both Simple and Detail modes. Provide reload and open-in-new-tab controls.
2. Keep the iframe mounted during ordinary project/run refreshes and mode
   changes. The current `projectView.ts` rebuilds main content; merely retaining
   an iframe object while detaching it may still reload its browsing context.
   Verify actual playback and interaction continuity in a browser.
3. Use a useful default height and optional child-to-parent height messages.
   Provide optional copy-text and shared-reference resolution helpers. Validate
   the message source and request shape; this bridge is not a general WFE API.
4. Test copying on the actual LAN HTTP origin. When browser clipboard access is
   unavailable, expose selectable text and a manual-copy fallback.
5. Show a compact neutral state for an absent page and a distinct failure state
   for an unreadable page. Neither prevents project editing. Offer an update
   indication when published content changes rather than restarting an active
   demo or video automatically. Explicit reload refreshes external media too.

Deliverable: usable Boast panel with no mandatory content format or custom SDK.

## Step 3 — establish project-owned Boast workflows

1. Inspect the current definitions, project instructions, repository bindings and
   artifact locations in RTS and Playprey. Add `devdocs/workflows/boast.yaml` to
   each project, callable both directly and by delegation through ordinary intake.
2. Give each workflow the task of creating or updating that project's showcase
   from the supplied request and available results. Leave presentation and content
   choices to the project agent. Do not turn the example screenshots, buttons,
   PC paths or videos from the braindump into required fields.
3. Put a delegate node near the end of RTS `build-game` and `improve-game`, after
   verification and establishment of a usable result. In Playprey `onboard`, place
   it after delivery publication; in `compare`, after adoption of the result to
   feature. Adjust the exact edges to the current definitions. Leave `freework`,
   knowledge backfill workflows and other projects unchanged.
4. Let the parent handle the child's ordinary questions and result handling with
   the current delegation capabilities. Where the definition still uses
   `accepts`, use `parent`; do not add a human confirmation for Boast updates.
5. Pass the result's actual commit/branch or published output reference, relevant
   evidence and durable launch/download location in the delegation request.
   The child must not silently substitute older `main` content for that result.
   Avoid duplicate application changes: the child primarily owns `boast/`.
6. Establish and prove the publication order. Prefer integrating the parent
   result before publishing the corresponding Boast update. If work is delegated
   before parent integration, prepare the child from explicit result references
   and integrate the showcase only once the advertised result is available.
   A durable independently published output can already be available before its
   source integration. Record the concrete sequence supported by current WFE;
   do not make the parent wait for a child that waits for the parent's completion.
7. Handle actual Git conflicts through existing agent-operated merge/retry tools.
   Check that subsequent parent integration preserves the child's Boast changes,
   and that a later request does not accidentally republish an older showcase.

Deliverable: two project-owned workflows and working delegation edges. Their
operation remains editable by their projects; no universal update policy is added.

## Step 4 — validate, deploy and exercise the workflows

1. Run focused tests for revision-consistent assets, preview selection, absent
   versus failed reads, content-root boundaries, message handling and media ranges.
   Use browser checks for iframe continuity, links, video seeking, narrow layouts
   and the clipboard fallback. Follow the repository's required release checks.
2. Read current Nautobot/nctl state and installed versions. Build and deploy WFE
   through agdevworld's normal release path, update project definitions and intake
   configuration as needed, and verify the installed service and launcher versions.
   No separate approval checkpoint is introduced by this plan.
3. Run each project's new `boast` workflow to create its initial page using real
   current outputs. Let the project agents author the pages. Integrate and publish
   the results, then inspect them in the deployed Project Editor.
4. Exercise a representative late-stage delegation in each project using fresh,
   bounded trial work. Verify dispatch, child input, return, parent continuation,
   integration order and the final public page. Validate all changed workflow
   graphs; do not restart held historical runs to obtain evidence.
5. Verify LAN access and durable artifact links after temporary presentation
   servers stop. Check serving through the normal `/wfe/` proxy as well as any
   development route, including asset paths and open-in-new-tab behavior.
6. Commit and push the changes and necessary submodule pins. Write `report.md`
   with changes, tested revisions, actual results and remaining limitations.
   Keep local inspection URLs and evidence in ignored local notes, and give the
   Developer direct links for the deployed RTS and Playprey project views.

## Completion evidence

- Both project views display freely authored `boast/index.html` pages, with
  working assets and meaningful ways to inspect their actual results.
- Both `boast` workflows work independently and through the selected delegation
  paths, with no extra human approval or creation requirement.
- Published pages describe available results; parent and child integrations
  preserve each other's work, and playback survives unrelated editor refreshes.
- The shared introduction remains descriptive only. Other projects receive no
  Boast obligation, template task, reminder or prescribed update cadence.
- The deployed LAN result is ready for inspection; implementation and operational
  evidence distinguish verified behavior from untested assumptions.
