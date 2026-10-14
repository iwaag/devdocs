# p9 review correction — retain acceptance checks for run deliverables

The Developer requested the fix after a light review of [report.md](report.md).

## Problem and correction

Integration treated the entire run folder as administrative content. A report,
plan or presentation could therefore be changed after result acceptance and
still reach the shared branch without another agreement. The review reproduced
this with a changed `report.md` that was integrated successfully.

agdevworld `c100f56` narrows that exemption to the run's own `run.json`.
Submodule gitlinks remain exempt in the parent because their accepted content
is checked in their owning repositories. All other run-folder files participate
in both the acceptance comparison and the check of a combined result.

Changing a report, plan or presentation after acceptance now returns
`needs-agreement`; normal administrative state updates still integrate. This
also applies to a report first written after acceptance: the updated result
needs agreement before integration. `docs/integration.md` records the contract.

The superproject pointer is `ce580c3`. No executor code or storage schema changes
are needed. The final report's unfinished `03:0x JST` timestamp was replaced
with the initial deployment date; its initial release history remains recorded.

## Verification

- `node --test test/integration.test.ts`: **7/7 passed**.
- New regression cases cover both devdocs storage modes (directory and
  submodule). They change `report.md`, `report-build.md`, `plan.md` and
  presentation content after acceptance. Each change requires agreement and
  leaves the shared root, study and devdocs targets unchanged.
- Restoring the reviewed content permits integration, including the expected
  administrative `run.json` updates and the accepted report on the remote.
- `npm run check`: type check clean, **287/287 tests passed**, production build
  successful.
- Before deployment, `nctl status` was healthy, the executor was idle, no
  attempt or integration was open, and the incident run's copy was ready.

## Deployment

- The committed release was built through `wfe-release.sh build`, which again
  passed type checking, **287/287 tests** and the production build.
- With no open attempt, the executor was temporarily stopped for activation
  and then bootstrapped again. The service and installed CLI both identify
  agdevworld `c100f56`; the normal browser entry responds HTTP 200.
- The migrated incident run remains in its dedicated copy, held at seq 11.
  No migration, result acceptance or integration was performed on that run.
- Component source and the superproject pointer were committed and pushed.
