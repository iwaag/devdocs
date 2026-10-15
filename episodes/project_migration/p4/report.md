# project_migration p4 — final report: archsage and shared project access

**Outcome.** archsage is a managed project of the workflow editor (`developer/pj-archsage`, workspace `archsage`). Reading any managed project is a system capability (`wfe projects`, `GET api/projects`), usable by any agent or operator and by a future Front, with no archsage configuration. All six trial cases ran through the actual executor. The old listener is retired, and its Nautobot record is updated.

Step reports:

- [report1](report1.md): contracts, gaps, write-back routes, inventory;
- [report2](report2.md): common access, cross-project guides, the integration guard;
- [report3](report3.md): pj-archsage built and validated;
- [report4](report4.md): deployment and real runs.

Plan: [plan.md](plan.md).

| | State |
| --- | --- |
| agdevworld (editor) | `8db95c0` released. p4 commits: `f94f07a` (access, guides, guard), `1d13ff9` (area category), `8db95c0` (CLI label). Previous release `a642e27`. |
| agautolab | `3fbafad`: the `wfrun` guide names `wfe projects` |
| pj-archsage | `developer/pj-archsage` `main`, every run integrated; guides corrected after trials 2 and 3 |
| rts-vs-bot | `wedo/` supplemented by archsage; `backfill-wedo` follows archsage's guides; main after the trials `0ab8f4a` |
| wedo-ppt | `knowledge.yaml`: maintained by ppt (`365b7a8`) |
| Old archsage | Listener stopped, plist archived, introduction #16705, Nautobot service and agent `retired`, placement `disabled`; checkout and records kept |
| pj-agdev | pointers to agdevworld `8db95c0` and agautolab `3fbafad` |
| Agent cost | $13.84 for step 4 (budget $30) |

## The common access contract (`workflow_editor/docs/access.md`)

- **What it reads.** Discovery, structure, content (tree, read, search), execution (runs, one run, run files) and provenance. It covers every registered project, with or without a workspace on this host, and every registered repository (`repo:<owner>/<name>`).
- **Committed by default, from system caches.** The root's default branch on Gitea is read through `<area>/.local/access-cache/`.
  - A submodule is read at the commit its project **pins**, or with `--upstream` at its own tip. `show` relates the two (`same`, `behind`, `ahead`, `diverged`).
  - `--rev` and `--require <commit>` select and check a revision.
  - The working tree is read only with `--source worktree`, and says so.
  - No workspace is written or fetched into. `.local/` and ignored files are never content.
- **Provenance and failure.** Every result names the project, repository (Gitea name, registry key, mount path), commit, the commit that pins it, observation time and fetch state (`fresh`, `recent`, `stale`). Failures are `absent`, `inaccessible`, `uninitialized` or `fetch-failed`; nothing falls back silently.
- **Live and published runs.** The live record (this host's resolver) is shown beside the published snapshot (the committed run folder). A run accepted but stopped by the guard shows as accepted, with no published snapshot.
- **Bounded output.** Reads 256 KiB (up to 4 MiB), trees 2000 entries, searches 200 matches. Truncation is always said. It is also offered read-only through agdevworld on the LAN.
- **Two write-path additions.**
  - `execution.guides: ["<project>:<path>"]`: a guide of another managed project, read at claim from its committed `main` and recorded by hash.
  - A binding's `integrate: if-unchanged`: the integration stops as `target-changed` when the target changed under the binding since the run's merge base with it. Nothing is pushed, and a successor run reassesses.

## archsage's demonstrated responsibilities

| Responsibility | Shown by | Result |
| --- | --- | --- |
| Cross-project advice with revisions and honest unknowns | `consult/run-001` | study-rts and wedo-ppt cited with commits and selectors; the GitHub-only arXiv copy said to be `inaccessible`; no game-AI paper found, and said so |
| Requested study through acceptance and integration | `study/run-001` | `notes/03-rts-bot-difficulty.md` in study-rts `d2734db → 303ce45`; archsage's pin moved, rts-vs-bot's did not |
| A later consultation sees the integrated revision | `consult/run-002` | It read `303ce45` and named rts-vs-bot's older pin. It missed the new note until the Developer asked; the revision read it. The guides were fixed. |
| Wedo supplementation from execution records (eligible, root-local) | `rts-vs-bot/backfill-wedo/run-001` | 3 topic files and 5 ledger rows, from improve-game 001–005, labelled by evidence strength; integrated `d32ca1c` |
| No duplicate on repetition | `rts-vs-bot/backfill-wedo/run-002` | no edit; the ledger's record commits matched |
| No edit on a project-maintained target (shared) | `archsage/backfill-wedo/run-001` | wedo-ppt `maintenance: project`, so no edit, decided in the plan |
| Reassessment instead of stale publication | `rts-vs-bot/backfill-wedo/run-003` → `run-004` | An ownership change after preparation stopped integration (`target-changed`); the successor gave no edit, and nothing stale was published |

## Write-back ownership

| Destination | Owning run | Reaches its branch through |
| --- | --- | --- |
| Shared study or wedo repository | archsage's `study` / `backfill-wedo`; the repository is a pj-archsage submodule, bound editable in exactly that workflow | acceptance, then integration into the repository's branch and archsage's own pin. Consumers' pins are never moved; `wfe projects show <consumer>` shows them behind. |
| Knowledge inside another project's root | that project's workflow (rts-vs-bot `backfill-wedo`), run as archsage-agent with `archsage:agent/*.md` guides, binding only the knowledge folder (guarded) | that project's acceptance and integration |
| Anything else in archsage | `freework`, where every knowledge destination is readonly | — |

Eligibility is the target's own `knowledge.yaml`: `project`, or `supplemented` with `supplementer: archsage`. Anything else, including a missing file, is unknown and therefore not eligible.

## Operating procedure

1. **Ask archsage.** Use a `request-…` topic in `wf-archsage.consult`, `.study`, `.backfill-wedo` or `.freework`, or *Request a run* in the editor. A wedo supplement for a project whose wedo lives in its root is requested in that project's own channel: `wf-rts-vs-bot.backfill-wedo` for rts-vs-bot.
2. **Answer.** Plans and results are answered in the run's request topic or its run view. A follow-up or a correction is a further round in the same run.
3. **A new knowledge target.**
   - Write its `knowledge.yaml`.
   - For a shared repository, add it to pj-archsage as a submodule and bind it in the study or backfill workflow; that is a definition change, which also needs approval.
   - For a root-local folder, add a workflow to the owning project modelled on rts-vs-bot's `backfill-wedo`.
4. **After integrations.** Fast-forward a workspace (`pull --ff-only`, then `submodule update`) before editing archsage's guides or index by hand. Workspaces are not pulled by integrations.
5. **Reading without archsage:** `wfe projects list | show | tree | read | search | runs | run` (`wfe help projects`).

## Legacy disposition (applied)

| Item | Done |
| --- | --- |
| Listener `com.agdev.archsage-zulip` | Booted out, plist archived; no process or launchd entry remains |
| `#agents` introduction | Retirement introduction posted (#16705), naming the new channels and `wfe projects` |
| Nautobot | `retire-archsage.yaml` applied: 3 updates, 0 conflicts. Drift shows the service lifecycle inactive with no active placement. The agent's liveness reads the last node dump, taken before the stop. |
| Sages and queues | Domains and the two queue notes imported with provenance; store `autodev/archsage-sages` kept |
| Conversations | Five finished `study-*` topics in `#archsage-agstudio1` marked ✔. `relations` (a bookkeeping topic) left open. History kept. |
| Routines `study-{growbox,worldtrend,aisvgs}` | Guides v4 (#16706–#16708): no refresh request to archsage. A run reports its integrated commit, and archsage reads the study in place. |
| autolab studies | Registered as shared study repositories for project access; still autolab's |
| archsage checkout | README and introduction marked retired; checkout, records and venv kept |
| Documentation | README_DEV: *Project access*, *archsage (workflow-editor project)*, old section marked retired. Host notes updated. |

## Remaining limitations

- **Front.** Its guides (`argue`, `shared/requests.md`, presentation speaker headers) still invite `@**archsage**` and `sage:<name>`, and ask archsage to establish studies (`agproject`/`agroutine`). Re-pointing them is Front's episode.
  - The new routes Front needs exist: `wf-archsage.*` channels, and `wfe projects` for reading projects itself.
  - Nobody establishes new autolab studies now that archsage does not; that is a decision for the Front/study episode.
- pyagag's argue selector and its `sagesync`/`routine-study-` progress rules, the agentroom progress reader and the settings manifest's archsage character are history readers, left as they are. The agentroom plist still lists the long-retired arxivsage root.
- **Index freshness is the agents' work.** Trial 2b showed a consultation trusting a stale file list. The guides now say to list the source at the cited revision, and a study records what it adds. Nothing automatic refreshes the index, by design.
- **Unexplained answer loss.** Two browser answers to one rts-vs-bot question never arrived; no error was seen, and the cause is unknown (report4, finding 4). Later answers were recorded by the Omni Agent on the Developer's delegation.
- **Bindings are declarations.** Publication does not enforce access. The guard protects integration targets only.
- `--upstream` applies to the first submodule on a path.
- **Leftovers kept as evidence.** rts-vs-bot `backfill-wedo/run-003`'s accepted but unpublished supplement stays on its run branch, as the trial's evidence; it can be requested again from the ex3 runs.
- **aisvgs, for the Developer.** Round 3 is parked, and a personal e-mail address remains in `autodev/aisvgs` `reports/strand7-local-kit.md`, which is autolab's repository (the index records both).

## Future Front usage

Front can discover and read any project itself:

- `wfe projects list` to see what exists;
- `show` for workflows and intake;
- `runs`/`run` for state and results, live or published.

It should route knowledge questions to `wf-archsage.consult`, study requests to `wf-archsage.study`, and wedo supplementation to the archsage or owning-project workflow that the target's storage dictates. `wfe projects show <project>` shows each knowledge target's maintenance file, and its workflows' `execution.agent` shows archsage-agent.

## Deus Ex Machina note

- Did for archsage: the guide facts after trials 2 and 3 (consult, study, backfill-wedo) and the rts domain's note list — handoff candidate (an archsage `freework` run).
- Did for rts-vs-bot and wedo-ppt: the maintenance files and the trial-5 ownership change and its restoration, which were setup and trial events.
- Every trial deliverable (advice, study note, wedo content, outcomes) was the agents'.
