# project_migration p5 — Step 7: scan consumer cutover and deployed proof

Completed 2026-10-16. Frontend `agdevworld` at `e5488f6` was committed,
pushed and deployed with `docker compose up --build -d web`. WFE's installed
release remains `38ebdff`: this step changed the importer and frontend,
not the WFE package. Host addresses, image/runtime evidence and screenshots
are in ignored `pj-agdev/.local/project-migration-p5/takeover/` notes.

## Contract and behavior

`cluster:fetch` now reads the live WFE run index and the selected scan's
`snapshot.json` through the existing read-only file API. It opens no agent
request, holds no cagent HTTP token, parses no prose URL and uploads nothing.

The newest scan by creation time must be completed. Missing, unfinished or
failed scans and malformed/unreadable source envelopes cause a nonzero exit
without replacing existing files. Partial host observation remains usable
when all three read envelopes are valid; metadata retains the incomplete
coverage and observation outcomes. This intentionally does not silently
select an older successful scan.

The three existing panel envelopes are still published under ignored
`public/cluster/`; `snapshot.json` adds scan identity, generation/fetch
times, coverage, per-host outcomes and source timestamps. Nodes/workspaces
views display scan time, age, partial state and a stale label after 24 h.
The display's timestamp is the scan generation time, not the import time.

## Live evidence

- Actual executor run `cagent/scan/run-002` collected all six default-scope
  hosts and produced `complete: true`, generated at 03:39:41 UTC.
- The trial guest had already been destroyed/pruned and was correctly
  excluded; the pre-existing Observer drift was reported, not repaired.
- Requester Omni Agent independently reviewed this scan, recorded the
  confirmation through WFE, stopped its presentation with verified
  process/port cleanup, and integrated it through the supported interface.
  Integration result: private `pj-cagent` main `a5f07ad` (later integrations
  advance main further). No human action is fabricated.
- The importer fetched this artifact through the LAN web proxy and wrote
  the expected envelopes and provenance. An actual browser rendered six
  nodes, requested the new metadata and raised no page errors; the
  workspaces view also fetched its envelope and metadata without errors. A private
  screenshot verifies the timestamp/footer visibly renders.
- After the old HTTP API stopped, another import through WFE still
  succeeded. Those additional outputs were kept in ignored evidence, not
  substituted for a newer observation. The deployed static view remains
  an honest snapshot of the stated scan time.

## Checks and defect found

Importer tests: **2 passed**; frontend TypeScript/Vite build passed; the
Docker build/deploy succeeded; browser smoke check passed. Tests cover a
newer failed/waiting scan, incomplete observation, invalid source data and
snapshot identity, preservation of previous output and provenance.

The first live API check exposed a test-fixture mistake: the HTTP run index
is an array, while the CLI wraps it in an object. Both importer and fixture
were corrected in `e5488f6`; the real proxied import then succeeded before
retiring the API. No failed import is claimed as a successful refresh.

## Verification ownership

The Developer had already given actual plan agreements and the guest-
destruction confirmation in the trial records. During takeover, Omni Agent
requested and technically reviewed the refresh and the remaining trial
results under the instruction to continue the migration. This is operator
verification; it is not represented as a new Developer-performed browser
trial. The deployed nodes view and each workflow's request/run view are
ready for direct human review.

Omni notes: recorded the reviewed scan confirmation, presentation cleanup
and integration for cagent — handoff candidate.
