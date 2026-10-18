# p14 Step 1 — the block store and resolver

## Inventory before the change

- **Source**: agdevworld `f998049` (installed release, clean tree). p13's
  local context was one override file per scope (`system.md`,
  `projects/<p>/project.md`, `projects/<p>/workflows/<w>.md`), each with a
  `*.revisions.jsonl`, written through `ContextStore.write` with a sha256
  `base`. The resolver gave each scope one `<scope>/override` source; its
  rendered heading carried the revision and the time of the last edit, so
  the prompt text changed on metadata the fingerprint ignored.
- **Consumers** of the override API: `server/api.ts` (`api/context/override[s]`),
  `cli/context.ts` (`wfe context set|clear|get|history|list`),
  `server/contextView.ts` (the matrix), `src/api.ts`, `src/contextParts.ts`,
  `src/contextView.ts`, `checks/context.ts`, `checks/contextEditors.ts`,
  `test/context.test.ts`.
- **Executor**: autolab `wfexec` runs on the same host as the area and calls
  the area's `bin/wfe` launcher; no executor runs on another host today.
  The CLI never goes through the service.
- **Local data on the live area** (inspected, not copied here): one system
  override set by the Developer (revision 3, after two p13 trial
  revisions), a cleared agops project override (2 revisions), a cleared
  agops workflow override (3 revisions), and five p13 snapshots
  (`ag.wfe-context.v1`).
- **Cluster state** (nctl status/drift before runtime work): Nautobot,
  worker and dumps healthy; 47 converged, 1 pre-existing drift
  (`agobserver service_missing`). The WFE service and executor are loaded.

## Conversion and backup approach (decided before changing data)

- **Backup**: the live `contexts/` was copied before any change to the
  area's ignored backups folder (`p14-step1-<time>`), and is copied again
  immediately before deployment (Step 5).
- **Conversion**: on the store's first use, each p13 override becomes the
  Developer-managed instruction block `developer` of its scope, with every
  p13 revision kept (same numbers, authors, times, notes; a removal stays a
  removal), an `external` revision if the file differs from its last
  revision, and a final `converted` revision. The old files move to
  `contexts/archive/p13/`; nothing reads or writes them afterwards, so there
  is one writable store. Cleared overrides keep their history as removed
  blocks. Conversion runs under a lock and is idempotent (a second process
  finds nothing to convert).
- **Snapshots** are not converted: v1 snapshots stay readable as recorded,
  so historical attempts keep their exact evidence. A p13 run's next
  attempt shows `system/override` removed and `system/block/developer`
  added once (a one-time session replacement for `session: run`).

## What was built (agdevworld `workflow_editor/`)

- **Model** (`shared/context.ts`, `ag.wfe-context.v2`): `ScopeTarget`,
  `ContextBlock` (id, title, manager, kind instruction|information, body,
  evidence, consult, details, editor; bookkeeping `checked`; `refresh`),
  `EvidenceRef` (ref, basis observed|reported|desired|recommendation,
  status ok|partial|failed|unknown, observed, coverage), `BlockRevision`,
  `BlockState` (revision, content `base`, size of the delivered section).
  `renderBlock` is the one rendering of a block; its hash is the source's
  sha256.
- **Store** (`server/contextStore.ts`): one JSON file per block, atomic
  replace under a per-block lock; per-block `revisions.jsonl`; operations
  `set` (create/change; omitted fields kept; idempotent), `check`
  (bookkeeping), `refreshFailed` (keeps content, refuses an empty block),
  `remove`; hand edits as `external` revisions; content-addressed details;
  the p13 conversion; snapshots readable in v1 and v2. `applyBlockOp` is the
  single operation entry for the HTTP API and the CLI.
- **Resolver** (`server/context.ts`): every block is a source
  `<scope>/block/<id>`, instructions before information, by id; removed
  blocks contribute nothing; an unreadable block is an `error` source.
- **Delivery vs bookkeeping** (specified in `docs/context.md`): a block's
  section names kind, title, manager, last editor and day (and run),
  evidence with observation minute, status and coverage, a failed refresh,
  details and consultation, then the body. A check, revision numbers, the
  time of day, detail contents and history are not delivered, so they
  never change the fingerprint or a session; everything delivered does.
  No age string is put in a prompt.
- **Provenance**: `ExecStore.attemptRun(attempt)` maps an executing attempt
  to its run for the editor record.
- **Contract**: `docs/context.md` rewritten for blocks (store, writing,
  evidence and age, delivery vs bookkeeping, detail and consultation
  routes); `docs/contract.md`, `README.md`, the authoring `AGENTS.md`
  template and autolab's `wfrun` guide name blocks and environment
  information.

## Checks

`node --test test/context.test.ts`: 11/11 —

- two managers' blocks are independent; a stale `base` conflicts; a human
  edit keeps the manager and records the human as last editor, and the
  agent's next write conflicts until it re-reads; identical publication is
  no revision; history keeps the human edit and the producing run;
- a check keeps base and fingerprint; a failed refresh keeps the body and is
  delivered; a human edit keeps the failure and the old observation time; a
  publication with evidence clears it and never invents an observation time;
  a failed refresh of an empty block is refused; details by reference;
  removal keeps history;
- hand edits are `external` revisions with a base of revision + 1;
- conversion of p13 overrides with revisions, archive, idempotence;
- resolver order, isolation between projects, unreadable guide;
- fingerprint unchanged by checks, restored by removal;
- claim snapshot equals the handed text, no block text or host path in
  run.json, later edits reach only the next attempt, block-level changes
  named with titles, a v1 snapshot stays readable;
- delegation scope; writes through the HTTP API with conflict, provenance
  and detail reads.

The browser code still uses the p13 override API until Step 3 (`tsc` over
`src/` and the browser checks fail until then); nothing is deployed in
between.
