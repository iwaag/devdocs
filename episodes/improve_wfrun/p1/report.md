# improve_wfrun p1: final report — consistent discovery and execution of published workflows

Date: 2026-11-02. Plan: [plan.md](plan.md); problems: [problems.md](problems.md); step reports:
[report1.md](report1.md) … [report5.md](report5.md).

## Outcome

A workflow that a run creates and integrates is discoverable and executable without anyone pulling
the registered workspace. The editor, project readers, intake, the notifier and delegation read the
project's **published source** (its committed default branch on Gitea), and an ordinary run takes its
definition and its code from one resolved root commit, recorded in `run.json`.

Shown on the live service with a dedicated project (`wfrun-trial`, workspace kept at its first
commit): freework created `stamp`; once integrated it was ready within minutes; ponderer's conductor
delegated "heron" to it; the child `stamp/run-001` ran the dispatched revision and its stamp reached
main; a second run executed on the remote worker gpupc. On production, gimmickimg's
`partial-animation` got its channel two minutes after the release, without a restart or a pull.

## Revisions

| Repository | Revisions |
| --- | --- |
| agdevworld (`workflow_editor/`) | `0220144` step 1 reproduction · `8d90c56` step 2 · `040f4a7` step 3 · `4c240ce` step 4 · `db4e7ae` docs · `011006b` discovery performance (regression fix) · `145c852` wording |
| Released WFE | `145c852` on the coordinator and the agpc worker (previous `47a83cc`); executor unchanged `a9b82f8` |
| pj-agdev | `e3c2052` (agdevworld pin) |
| devdocs | step reports, this report, `README_DEV.md` *Published workflows* |
| Test project | `developer/pj-wfrun-trial` (new; check actor `improve-wfrun-p1-check`) |

## What was delivered

- **Source contract** (`server/published.ts` on the existing access caches): resolve a project to its
  default branch fetched now, or a named root commit; a read-only published view of that commit used
  for listing, validation and the frozen bundle; discovery reads labelled with their fetch state; an
  integration marks the project's caches due. Fetch failure is refused for execution, labelled for
  discovery, never replaced by workspace content.
- **Run creation**: definition and copy from the same commit; every pinned submodule (absent from the
  workspace or nested) in the copy; default-branch integration targets; the receipt keeps the reserved
  revision; `definition.source` = `published` / `trial` / `workspace`.
- **Delegation**: target resolved on the published branch, revision recorded (`target.source`), child
  created at it; `wfe delegate targets` for readiness; the result assessment's `ran —` line names the
  workflow and revision the child ran; tool-help guidance that creating and running a workflow are
  separate results.
- **Editor**: Published (default) and Workspace (editing) lists; published definitions readable in
  place; workspace relation (same/behind/ahead/diverged/unknown, dirtiness apart) with Update
  workspace; request form Published / Trial (exact editing bytes on the published code); run view
  Source fact and Trial chip.
- **Workspace sync** (UI and `wfe workspace status|sync`): fast-forward only, rechecked before each
  change, per-repository steps, partial updates reported and continued by running it again.

## Checks actually run

- Full suite at each step and in every release build: final **673/673** (657 before p1).
- New tests: `publishedSource.test.ts` (12), `publishedDelegation.test.ts` (4), `workspaceSync.test.ts` (6),
  covering every row of the plan's acceptance table in fixtures (stale, dirty/ahead/diverged,
  revision race, delayed delegation with duplicate delivery, submodules, live discovery, source
  failure, editing trial, workspace sync with partial failure, existing runs).
- Replay check of every live workspace with the new and the previous code before release: 40/40 usable.
- Deployed: trial above (records checked directly, not only the conductor's reply #565), production
  channel provisioning, `wfe delegate targets`, `wfe workspace status`, headless-browser checks of the
  deployed editor and run view, agpc read-back.

## Problems found during the episode

- **My regression, fixed the same hour**: `db4e7ae` made the executor's `wfe intake process` and
  `wfe notify due` exceed 300 s (every validation re-read all peers with Gitea API calls). Intake and
  notifier passes failed for about 20 minutes (04:14–04:33 JST); the cursor-based intake caught up on
  the next pass. Fixed in `011006b` (intake status 3.2 s, previously 17.6 s on `47a83cc`). The test
  suite does not cover discovery at live-registry scale; the release now records the timings.
- **gimmickimg's `partial-animation` is invalid** (`confirms-type`: a `review` node set to confirm the
  result). It is now visible as published and NOT READY instead of missing; it cannot run until
  gimmickimg fixes it. Integration does not check the validity of definitions a run adds. Left to
  gimmickimg (its content), reported here.
- A `--trial` CLI flag collided with `wfe challenge … --trial`; the request flag is `--editing`.
- Pre-existing, unrelated: two notifier deliveries are refused every pass with "Message must not
  contain null bytes" (ponderer conduct/run-038 and lowerpoly lod-chain/run-003 io topics).

## Limits and what was not verified

- Workspace sync and editing trials were exercised in fixtures and the UI only; no live workspace was
  moved (gimmickimg is still 6 commits behind, wfrun-trial 10 behind on purpose).
- Revision race and delayed delivery were exercised in fixtures; on the live service the default
  branch did not move between dispatch and child creation.
- A 390 px horizontal overflow seen once on the deployed project screen did not reproduce.
- Discovery may read a cache up to 30 s old (labelled `recent`); invalidation covers integrations made
  by this host's executor, not pushes made outside WFE.
- Rollback needs data care: the previous release cannot replay a dispatch with `target.source`
  (ponderer conduct/run-069). The procedure is in the coordinator's local notes.
- A host without Gitea still creates runs from the workspace, recorded as `workspace` with the reason.

## Follow-up (problem 5 and around it)

- Copy retirement: published copies are now clones of the access caches with every pinned submodule,
  which makes copies of projects with large submodules bigger than before (archsage's
  `study/study-aesthete` is the only live case). Retiring finished copies without losing their record
  or unpublished outputs, then their run branches, is the next phase.
- Validate definitions a run adds at integration time (or report them as not runnable at once), so a
  case like `partial-animation` is caught by the run that created it.
- A scale check of discovery (live registry size) in the release procedure.
- The wfrun-trial project and its room can be kept as a fixture for later phases or unregistered
  (`wfe project unregister wfrun-trial`) when no longer needed.
