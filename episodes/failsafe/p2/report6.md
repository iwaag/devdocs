# failsafe p2 — step 6: consolidation

## Documentation and guides

- `README_DEV.md` (devdocs `2071742`):
  - the observer section: health checks, developer reviews and the trial
    aids;
  - the trace section: live execution records and the probe;
  - the autolab section: resumption is not acceptance, and the live
    records;
  - p1's timings, updated (the `unheld` grace, the judgment deadline, the
    detection targets).
- pyagag `docs/health-v1.md` (`3e9729e`): the `agag.execution.v1` and
  `agag.health.v1` contract.
- Observer's introduction (`b0c33e6`, re-posted as #12583):
  - checked runs, and the 3- and 10-minute bounds from the first doubt;
  - developer reviews and their policy;
  - a look every minute.
- autolab's introduction (`eb2c032`, re-posted as #12584): the agreement
  that closes a task comes after its shown result, in the task's own
  topic. A resume or continue closes nothing, and neither does an
  agreement posted in the plan's topic.
- The monitor module's docstring describes the health path.
- Front's three guides were updated in step 3 (`bcba734`).

## Dependency pins

pyagag `f1e5973` in agautolab, agfront and agobserver. The other consumers
(agforge, archsage, cagent, the relay, comfynotify) keep `f5c4359`, because
nothing they use changed:

- the live record is opt-in;
- the new trace fields are additive;
- `stall_candidates` is used by Observer only.

## Superseded behaviour removed

Removed on the covered path (the details are in `report.md`):

- `silent` and `quiet` for probed owners;
- the 300 s `unheld` grace;
- the 120 s look interval;
- the 900 s judgment deadline;
- the two-requests-ten-minutes-apart pattern for checked runs.

The conversation-only kinds remain for owners that are not probed.

## Local notes and cleanup

- `pj-agdev/.local/devenv.md` has a "failsafe p2 on this host" section:
  - versions, the health config and state files;
  - trial aids, trial conversations and missions;
  - a zsh waiting pitfall that cost one trial.
- Every trial fault file and `timing.json` are removed. The health record
  shows the operational timing.
- The trial missions are all `done`. The two review topics are left open
  for the Developer's ✔.

## Final report

`report.md`: the health contract, operational timings, trial evidence,
false interventions, duplicates, human interventions, probe and model
costs, developer handoffs and remaining limitations.
