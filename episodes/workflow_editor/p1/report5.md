# workflow_editor p1 — step 5 report: approval and end-to-end trial

## Outcome

Approval actions and stale-state display work. The complete acceptance
scenario ran end to end against real local repositories and passed all 48
checks (`checks/e2e.ts`, screenshots `e2e-*.png` in
`pj-agdev/.local/workflow-editor/screenshots/`). With it: 31/31 service tests
(5 new approval tests), the step 2–4 browser checks re-run green after the last
UI change, the type check is clean, the production build passes, and
single-process `npm start` mode was smoke-tested.

## Approvals

- The top area shows two pills: "Intent: approved / changed since approval /
  draft" and the same for Definition. Each names the declared approver, and its
  tooltip shows the approved and current digests and the time. "Author Approved"
  appears only when both are approved.
- The state is computed live from the draft (Web Crypto SHA-256 over the
  canonical projection). While the draft is unsaved, the pills say so, and both
  Approve buttons are disabled until Save. Approve definition is also disabled
  while there are validation errors.
- `POST …/approve {kind, approver}` reads the **saved** file, validates it,
  computes the digest and writes the record through the same round-trip save.
  The service refuses an empty approver, an empty intent, a definition with
  errors (409, with the errors) and unreadable content.
- The approver name is a declaration: it defaults to the registry's `approver`
  and is remembered per browser. Nothing authenticates it.

Service tests (`test/approval.test.ts`) cover records in the file with comments
kept, and description → definition stale while the record is kept. They also
check intent → both stale, reapproval, layout → both still approved, and
definition refused with errors while intent is allowed. Finally: name, intent
and readability are required, and a hand-edited wrong digest shows as stale.

## Acceptance scenario (all passed)

1. **Seed:** sources, `pj-demo` with `devdocs`, `study/*`, `wedo/*` and
   `assets/shared` submodules, two workspaces, origin a local bare repository
   (no Gitea).
2. **Workspace A:** intent, 3 goals, root + 6 submodules, 3 workflows shown. A
   fourth goal was added and saved.
3. **Workflow `e2e-release` in the UI:** bindings readonly/editable, five nodes
   of all four types, branch (study → build, agree), join (verify ← build,
   agree), delegate to `repo-setup`. Connections were made through the
   inspector and by port drag, then auto-arranged and saved. On reopen: same
   graph, nothing unsaved, file byte-identical.
4. **External then UI:** an IDE-style save changed a description and the UI
   followed. A UI rename was saved next to the external change.
5. **Validation and recovery:** an external edit to `repo-setupp` made the
   delegate card show "missing", gave exactly 1 error and disabled Approve
   definition. It was fixed in the UI. Malformed YAML then showed the error,
   kept the last valid graph and disabled Save; correcting the file recovered.
6. **Approvals:** intent and definition approved by `e2e-author`, and both
   records were in the file. Editing the Agree description made the definition
   "changed since approval" with intent still approved, both in the draft and
   after save. The stale record was kept. Editing the intent made both stale.
   After reapproving both, a node was moved and saved: both stayed approved and
   the approval records were byte-identical.
7. **Git publish and workspace B:** the steps were `git switch main`, commit and
   `push origin main` in A's devdocs, then commit and push of `project.yaml` and
   the devdocs gitlink in A's root (A clean afterwards). In B: `git pull
   --ff-only` and `git submodule update --init` (with per-command
   `protocol.file.allow=always`). B's `project.yaml` and workflow file are
   byte-identical to A's, and devdocs is at the recorded commit. B's project
   view shows the new goal and "Author approved". B's editor shows both
   approvals valid and the moved node at A's position.
8. Recorded here and in `report.md`.

## Deviations and fixes found by the trial

- **Early typing was lost.** On a just-created workflow, text typed into the
  intent before the file had loaded was overwritten by the load. The name and
  intent inputs are now disabled until the file is loaded. The step 3 check had
  hidden this by waiting for the name; the e2e check found it.
- Approve buttons stay enabled when already approved; re-approving just
  refreshes the record. Left as is.
- Publishing is manual Git, as the plan requires. The editor never commits or
  pushes. The trial's commits use the fixture identity, in the ignored fixture
  repositories only.
