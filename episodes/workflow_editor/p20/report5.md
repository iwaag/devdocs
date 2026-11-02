# Step 5: workspace maintenance out of the ordinary flow

agdevworld `d053143`.

## What changed

- Project page: no tabs and no sync banner in Simple mode. Checkout facts and Update workspace are in Detail mode's "Developer details: this host's checkout", which says nothing about workflows depends on it, and links each checkout file to a read-only view.
- Request form: runs the applied workflow. Trying unapplied changes moved to the draft (Try draft); the browser route for trying a checkout's file is removed.
- `server/localEdits.ts`: inventory and import. A file equal to a revision the project had is only behind and is no edit. Others are `uncommitted`, `local-commits` or `untracked`, each with its own base. Import changes nothing in the checkout and publishes nothing. The project page offers it, in Simple mode too.
- CLI: `wfe draft list|show|start|save|compare|try|apply|discard|edits|import`. The help of `wfe workflow`, `wfe request --editing` and `wfe workspace` now says they operate on the checkout and when their effect reaches runs.
- Docs updated to one contract: `docs/drafts.md` (new), `access.md`, `storage.md`, `runs.md`, `contract.md`, `context.md`, `README.md`, `templates/AGENTS.md`.

## Deliberately kept

- HTTP routes `workflows[/<file>]` and `.../approve`, `.../instantiate`, used by the checkout view and by `wfe`/test paths for people working in a checkout. The GUI no longer writes through them.
- `project.yaml` editing still writes the checkout, and the project page's top half still reads it (`api.project`), so the page needs an available checkout. The plan scopes this phase to workflows. Report as a limitation.

## Checks

`npm run check` at this commit: 733/733, including typecheck and both builds (`pj-agdev/.local/workflow-editor-p20/check1.log`). The migration inventory across registered projects is run against the deployed release in step 7.
