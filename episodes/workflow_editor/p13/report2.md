# p13 Step 2 — execution and recorded delivery

## What changed

- **Claim** (`ExecStore.execution`): resolves the run's bundle once, renders
  it, and saves bundle and text as one content-addressed snapshot in
  `<area>/.local/contexts/snapshots/`. `attempt.begin` records
  `settings.context {snapshot, fingerprint, sources[{key, scope, category,
  delivery, sha256}]}`; the claim returns `execution.context.text`, the
  snapshot's exact text. The preview, the record and the launch payload are
  one resolution, so a concurrent edit cannot make them disagree. A source in
  `error` drops the job with the reason (as a missing guide did).
  `settings.guides` is no longer written; older records still parse.
- **Bootstrap** (autolab `wfexec.prompt`): common contract, then the context
  text, then "# This attempt" with the brief. The separate
  `# Project guide:` injection is gone, so a guide named by project and
  workflow appears once. Continuations are unchanged (they carry no context;
  the session holds it). Reconstruction gives the whole bootstrap with the
  claim's context.
- **Delivery record**: `attempt.context delivered|undelivered` (new run op,
  fenced by the open attempt, recorded by `wfe exec context <attempt>`).
  The executor records `delivered` after writing the prompt to the harness,
  `undelivered` when the harness closed its input first or the launch
  failed. Delivery is stated as delivery only.
- **Sessions**: `SessionConfig.context = {fingerprint, sources}` over the
  effective sources (run input and the contract excluded; the contract keeps
  its own hash). `configChanges` names each source added/changed/removed
  (`configuration changed: context guide/agent/g.md changed`).
  Metadata-only edits keep the conversation. Sessions recorded with guides
  only are replaced once, with the reason "created before scoped context".
- **Read views** (`server/contextView.ts`): per attempt the recorded
  identities, the snapshot (or why it is unavailable — another host, or an
  attempt before p13 with its guides by hash) and the delivery; the next
  attempt's bundle; the differences; what the session would do. API
  `GET …/runs/<wf>/<run>/context`, CLI `wfe context run <run>`. Nothing in
  them writes.
- **Delegation**: a child resolves its own project/workflow; a test shows the
  parent project's override does not reach it.
- autolab's wfrun contract will say that a "# Context" section follows it;
  that text change is applied at deployment (the live executor reads the
  guide file at every claim, and the installed WFE release does not yet
  produce the section).

## Checks

- WFE: `npm test` 394/394 (incl. `test/context.test.ts`), `tsc` clean.
- autolab: `tests/test_wfexec.py`, `tests/test_wfconversation.py` 22/22
  against the WFE source: the settings test checks the context section
  follows the contract once, the recorded source identity and the
  `attempt.context delivered` entry; a changed guide replaces the session
  with the context reason; a resume gets the continuation without the
  contract or context again.

## Commits

agdevworld `0311800`, agautolab `e66e1dd` (source; not deployed yet).
