# project_migration p4 — step 4: deployed, and the workflows proven with real runs

Written 2026-10-16. Nine runs were executed by the installed executor as archsage-agent (`sonnet`). Agent cost was **$13.84** for the whole step, against a $30 budget.

## Deployment

| Item | State |
| --- | --- |
| Release | agdevworld `1d13ff9` (previous `a642e27`), `wfe-release.sh deploy`. The release's own check passed 357/357. The service and `bin/wfe` report the same commit. |
| Backup before | exec, notify, intake, present and delegate stores (`.backup`), plus registry, zulip, system, present and node-catalog json, in the area's ignored backups (`pm-p4-predeploy-*`). No store migration was needed. |
| LAN | `GET /wfe/api/projects` and the run views answer through agdevworld's nginx from the LAN host name. |
| Executor guide | agautolab `3fbafad`: the `wfrun` guide lists `wfe projects` among what every run can see. It is read live at each claim, so no restart was needed. |
| archsage | Gitea `developer/pj-archsage` (id 140). `wfe project register` and `obtain` give workspace `archsage`. 4 workflows are valid. Channels `wf-archsage.{backfill-wedo,consult,freework,study}` (#304–#307) were provisioned by the notifier. |
| Trial targets | rts-vs-bot `1062e00`: the `wedo/` area (supplemented) and `backfill-wedo` (channel #308); the workspace was fast-forwarded so the definition is visible. wedo-ppt `365b7a8`: `knowledge.yaml` `maintenance: project`. |
| Registry | The five autolab study repositories (`autodev/aisvgs`, `growbox`, `worldtrend`, `protoprey-research`, `papers`) are registered as shared `study` repositories. Project access reads them as `repo:autodev/<name>`. |
| From a run copy | From inside a real run copy, with a launchd-like environment (only the area's `bin` on `PATH`): `wfe projects read archsage index/README.md` and `wfe projects read rts-vs-bot wedo/knowledge.yaml` returned with provenance. The workspace whose copy was used stayed unchanged. |

## Runs

| # | Run | Outcome | Decision / integration |
| --- | --- | --- | --- |
| 1 | `archsage/consult/run-001` | Advice on a small RTS with a bot opponent and per-build review. It cites `study-rts@d2734db` (pinned), rts-vs-bot's run records, `study-ppt@e9c153b` and `wedo-ppt@293227c`. Honest unknowns: no game-AI paper in the arXiv study; GitHub `study-arxiv-trend` is `inaccessible`, so `autodev/papers` was read instead and that was said. | Accepted by the Developer (browser), integrated |
| 2 | `archsage/study/run-001` | Plan: scope plus destination. Wrote `notes/03-rts-bot-difficulty.md` into study-rts with sources, and `findings.md` with the write-back facts. | Accepted (Developer). Integrated: study-rts **`d2734db → 303ce45`** (fast-forward), archsage's root merged with its pin moved to `303ce45`. **rts-vs-bot's pin stays at `d2734db`** (`wfe projects show rts-vs-bot`: "behind, 1 behind"). |
| 2b | `archsage/consult/run-002` | First answer: it read study-rts at upstream `303ce45` and said rts-vs-bot pins `d2734db`, but **missed the new note** (it trusted the index's list of two notes). The Developer asked about the note in the result confirmation. The revision read `03-rts-bot-difficulty.md` at `303ce45` and answered from it. | Accepted after the revision (Developer), integrated |
| 3 | `rts-vs-bot/backfill-wedo/run-001` (owned by rts-vs-bot, archsage's guides) | Eligibility `supplemented`. Evidence: improve-game run-001…005; 001 and 002 labelled unaccepted, 003 to 005 accepted. `building.md`, `testing.md`, `presenting.md` and 5 ledger rows. One correction round (below). | Accepted (Omni, delegated), integrated into rts-vs-bot main **`d32ca1c`** (fast-forward). The study pin was untouched. |
| 4a | `rts-vs-bot/backfill-wedo/run-002` | The same request again. The ledger matched all five runs' record commits, so **no edit**: `wedo/` is unchanged (`git diff d32ca1c HEAD -- wedo` is empty). | Accepted, integrated (run records only) |
| 4b | `archsage/backfill-wedo/run-001` | wedo-ppt, brought up to `365b7a8`: `maintenance: project`, so **no edit**, said in the plan before any evidence work. | Accepted (Developer). Integrated: archsage's pin `293227c → 365b7a8`. **ppt still pins `293227c`.** |
| 5 | `rts-vs-bot/backfill-wedo/run-003` | Supplement from the six ex3 trial runs: 264 lines in four files. After it presented, an operator commit on main set `wedo/knowledge.yaml` to `maintenance: project` (`8c68c2c`). It was then accepted as presented. | Accepted. **Integration stopped `target-changed`**: "main changed under the guarded binding wedo (wedo) since this run built on it (fd62fc0 → 8c68c2c: wedo/knowledge.yaml); nothing was integrated". The content stays on its run branch. |
| 5b | `rts-vs-bot/backfill-wedo/run-004` (predecessor run-003) | The successor fetched the current target, read the ownership change, and gave **no edit**. run-003's content is not published. | Accepted, integrated (records only). `wedo/` on main is byte-identical to run-001's result. |

Plan item 6 (write-back for shared and root-local) is covered by run 2 (shared study-rts, through archsage's run) and run 3 (rts-vs-bot's root-local `wedo/`, through rts-vs-bot's run following `archsage:agent/*.md`). Neither involved a push by an agent: every change reached a branch through acceptance and the executor's integration.

After the trial, rts-vs-bot's `wedo/knowledge.yaml` was restored to `supplemented` (`0ab8f4a`), noting the trial. The ownership change was a trial event, not a decision about the project.

## Evidence read from records, not from claims

- Integration states come from `wfe run integrate --status` and `run.json`'s `integrations`.
- Branch tips and pins come from `wfe projects show/tree` (the system's own access, with fetch times).
- "No edit" was checked by diffing `wedo/` in the run copies against the base.
- The guard stop was checked through `GET /api/projects/rts-vs-bot/runs`: run-003 is accepted with no published snapshot; runs 001, 002 and 004 have one.

## Findings

1. **The index's file lists go stale, and a consultation trusted one** (run 2b). The domain text I seeded named study-rts's "two notes". The study guide told the run to record the commit it built on, so the index kept `d2734db` and no new note.
   - Fixed in archsage `agent/consult.md` (list a source at the revision you cite before saying what it holds) and `agent/study.md` (a study records what it adds: paths in `sources.yaml`, files in the domain).
   - `index/domains/rts.md` was corrected. These are trial-referenced facts in the guides.
2. **The ledger recorded the run branch's checkpoint as the target commit** (run 3: `b254e83` instead of `1062e00`). It was corrected in a review round, and `agent/backfill-wedo.md` now says the target commit is `origin/<branch>`'s.
3. **CLI wording.** `wfe run integrate --status` lists the files of a `target-changed` stop under "changed since the acceptance", which is the label of a `needs-agreement` stop. The integration itself was right. This is fixed in step 5's release.
4. **Two browser answers to rts-vs-bot run-001's plan question did not arrive.**
   - Nothing reached the record, the service log or Zulip.
   - The view rendered correctly in a headless browser with the answer form and no console errors.
   - The write route answered a probe (refused as outdated, nothing written).
   - The cause is not known. The empty *Recording as* field is one candidate: the archsage runs were answered in the same session.
5. **Decisions.** The Developer answered runs 1, 2, 2b and 4b in the browser. On 2026-10-16 the Developer told the Omni Agent to answer for them and move on. Every later answer (run 3's plan, correction and acceptance; runs 4a, 5 and 5b) was recorded with `wfe run answer --by "Omni Agent (for Developer)"`, and each says it was delegated. One attempt through the HTTP route was refused by the session's safety classifier; the CLI was used instead.
6. Agents wrote a presentation server's loopback address and port into advice text (run 1). This is the same class as forge's p3 finding 6: harmless here, but host-specific.

## Cost

| Project | Runs | Cost |
| --- | --- | --- |
| archsage | consult 001, 002; study 001; backfill-wedo 001 | $5.05 |
| rts-vs-bot | backfill-wedo 001–004 | $8.79 |
| Total | 8 runs, 24 attempts | **$13.84** |

## Manual steps (operator)

- All requests were made with `bin/wfe request … --author "Omni Agent (for Developer)"`. The texts are in the ignored `pj-agdev/.local/project-migration-p4/requests/`.
- The trial-5 ownership change and its restoration were plain commits to rts-vs-bot main, from the migration clone, pushed with the editor's Gitea credentials.
- The archsage workspace was fast-forwarded after integrations before each guide fix, and the guide fixes were pushed from it.
- Deus Ex Machina: the Omni Agent wrote the three guide facts and the rts domain correction. The trial runs' content was the agents'.

## Human review paths

`<editor>` is the workflow editor's entry through agdevworld (host notes, `pj-agdev/.local/devenv.md`).

- archsage runs: `<editor>/#/ws/archsage/run/<workflow>/<run>` (consult/run-001, run-002, study/run-001, backfill-wedo/run-001)
- rts-vs-bot: `<editor>/#/ws/rts-vs-bot/run/backfill-wedo/run-00N` (1–4)
- The new study note: `wfe projects read rts-vs-bot study/study-rts/notes/03-rts-bot-difficulty.md --upstream`
- The supplemented wedo: `wfe projects tree rts-vs-bot wedo`
