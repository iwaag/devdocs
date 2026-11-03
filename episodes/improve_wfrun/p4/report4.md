# improve_wfrun p4 step 4: coordination made visible, revision supported, Observer updated

Date: 2026-11-03. Plan: [plan.md](plan.md). agdevworld `96404c2`, corrected by `38adb4b` and `1053a73`; pj-agdev (agobserver) `706e1d1`.

## Views

- **Status in short** (`shared/status.ts`): `Queued · Awaiting warden's decision`, `Deferred · By
  warden, until …: <reason>`, `Queued · Starts when its execution node next claims`, `Starting · Its
  execution node took the job`; without coordination `Queued · Waiting for its execution node`. Used
  by the run lists, run views, `wfe run list|show` and the conversation views (`server/conversations.ts`
  passes the decision).
- **Run view**: Execution node with its provenance ("named by the request (a placement constraint)",
  "the workflow's choice") and every move (from → to, by, reason); a Coordination fact with the last
  decisions; in Detail, the active block says the decision the attempt started by and who shares the
  node. The activity line's note reads "starts when warden decided it does and its node claims it".
- **Home, Execution nodes**: every open attempt per node (not the default slot only), queued jobs by
  admission (awaiting warden / deferred / admitted), the measured load.
- **Observation** (`wfe observe`): `coordination` (configured, turn, stalled, awaiting, deferred,
  needs, pending events, overdue deferrals, last turn); per run `queue.job.admission|decision|
  decisionId|deferUntil|reconsider`, `waiting.responsible = coordination` for a job awaiting or
  deferred; `workers[].slots[].decision`, `workers[].load`, `awaiting`, `deferred`, `waiting[]` with
  the admission.
- **`GET runs/<wf>/<run>/exec`**: `coordinated` and `decisions` beside the jobs (with `decision`,
  `decision_id`, `defer_until`).

## Revision

- A working agent, recovery or a person asks for another look with `wfe coordinate reconsider
  --reason …` (the wfrun reference notes and the recovery guide say when); warden may note, defer,
  move between attempts (`start --worker`, `assign`) or stop (`wfe run stop`, with its actual outcome
  recorded by the run; the coordinate guide says a stop is not a safe suspension). A human-requested
  hold stays the person's (`recover-human-hold` unchanged; a person's assignment is a constraint).

## Observer (`agobserver`)

- Classification (`wfclassify.py`): a queued job's kind follows its admission (`awaiting-decision`,
  `deferred`, `admitted`, `claimed`) and stays a legitimate wait; a deferral past its time with no
  later decision is `uncertain` (family `coordination`, `reconsideration-overdue`), told to the owners
  after 600 s and never sent to recovery; `coordination_finding`: turns ending without a decision
  (WFE's `coordination.stalled`), or jobs awaiting a decision for 900 s with no turn open or opening,
  are one incident (uid `coordination`) naming the waiting runs, told to the owners at once; an
  executor outage's affected runs include every open attempt (`workers[].slots`), not only the first.
- Monitor (`wfmonitor.py`): the coordination finding is suspected beside the executor's; family
  `coordination` escalates with its grace; a coordination incident closes when a turn decided again or
  nothing awaits; `[recovery] exempt` names workflows never sent to recovery (`warden/coordinate`);
  the owners' advice names `wfe coordinate view|decisions|wake|start`.
- `docs/workflow.md`: the classification rows, the incident families and warden's part in the
  responsibility table; the `queue-stalled` row is marked as not implemented in code.

## Checks run

- agobserver `uv run pytest -q`: 127 passed (5 new: legitimate awaiting/deferred waits; an overdue
  deferral escalated and closed when decided again; a stall as one incident naming the runs, closed
  when a turn decides; a long wait with no turn is a stall and an open turn is not; the exempt
  setting).
- workflow_editor: `test/status.test.ts`, `test/observe.test.ts`, `test/coordination.test.ts` (the
  observation reports the stall, `admission`, `responsible: coordination`), `vite build` for both
  bundles.

## Not done / open

- The `queue-stalled` condition documented since observer p3 was never implemented; left as
  documented, not added now.
- Browser screenshots of the new facts are part of the deployment check (step 6).
