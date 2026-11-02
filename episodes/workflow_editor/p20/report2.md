# Step 2: ordinary viewing follows published definitions

agdevworld `d053143`.

## What changed

- `#/ws/<ws>/wf/<file>` is now the published graph, not the checkout's file. It works for a workflow the checkout lacks. Text is a secondary action in the header.
- `src/workflowView.ts` is one renderer with an explicit source: `published`, `draft`, `run`, `checkout`. A band states the source in Simple mode too. Revisions, paths and digests are in the Detail mode's line.
- The project page has one list. The Published / Workspace tabs and the published YAML panel are gone.
- Freshness: open views poll `published-head` every 3 s; the service reads at most 4 s old and coalesces per project. The view re-reads when the commit changes, on reconnect and on becoming visible. Implemented bound: about 7 s plus a fetch. Measured in the browser: 2.5 s from push to the open graph (one sample per run of `ui.mjs`; load is measured in step 6).
- Fetch failure: `fetch.state: stale` keeps the last graph with a stale notice and the last check time. Never a fallback to the checkout. Missing, moved, malformed and unreachable each have their own wording (`workflow_editor/docs/drafts.md`). None offers Save.
- Run definition: `#/ws/<ws>/run/<wf>/<run>/definition[/<revision>]` reads `bundle` and `revisions` from the run's own record. The run page's link is now "Definition used by this run", in Simple mode too. It compares with no checkout any more.

## Checks

- Browser, scratch service against the real Gitea test project, with a checkout 48 commits behind and dirty (`pj-agdev/.local/workflow-editor-p20/check/ui.mjs`, screenshots beside it): graph of a workflow absent from the checkout, read-only state, text, push visible within the bound, run keeps its revision after the project moves on, 390 px without horizontal scroll.
- Not yet checked here: stale, malformed and unavailable sources in the browser, and a retired run (step 6).
