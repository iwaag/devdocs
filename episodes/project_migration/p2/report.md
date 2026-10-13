# project_migration p2 — final report: pj-jevlab under the workflow editor

**Outcome.** pj-jevlab, a project on Jev-style decision models built outside the system, is a managed project of the workflow editor. It has four workflows, and one study run went from request to an accepted, published result. p1's checklist was enough to migrate it without any change to the editor's contract; what it lacked was about running, not about migrating.

The `experiment` workflow was taken out of this phase's scope by the Developer on 2026-10-13, after its first run had written its plan (see *Open*).

Step reports: [report0](report0.md) (preflight), [report1](report1.md) (contract gap check), [report2](report2.md) (migration and registration), [report3](report3.md) (the study run). Plan: [plan.md](plan.md).

| | State |
| --- | --- |
| pj-jevlab | `trustedagent/pj-jevlab` `7343f59`, public |
| agdevworld | `783cfcb`, installed (was `92de1da`) |
| agautolab | run guide amended (background work at the end of a turn) |
| Workspace | `jevlab`; `study/run-001` completed; `experiment/run-001` held (stopped) at the Developer's request |

## What was done

1. **Preflight and gap check.** The installed contract accepted `areas: study, lab` at the top level as they were; the only visible difference from agsfx is that a top-level `study` is shown in category *other*. No system change was needed for the migration.
2. **Migration**, one commit on top of the person's last one:
   - added `project.yaml`, `agent/guide.md` and one guide per workflow, `devdocs/README.md`, and the workflows `study`, `experiment`, `improve-tools`, `freework`;
   - removed `todo.md` (its purpose line is in the devdocs README);
   - nothing moved: `study/`, `lab/` and the 200+ commits of episodes keep their paths and cross-references.
3. **Registration.** `wfe project register` and `obtain`. Intake and channels (`wf-jevlab.*`) came by themselves; approvals were recorded for the Developer.
4. **One real run.** `study/run-001` extended `study/recent_study` with 9 papers from 2026-10-08 to 10-13. It was accepted by the Developer and published, and the published files carry no host details. Details in report3.

## What the checklist lacked

p1's checklist, item by item, as this migration used it:

| p1 item | Here |
| --- | --- |
| Read the project's own runbook first | **Amended.** There was none. The workflows came from the shape of the episodes (survey; pre-registered experiment with a cap; tool or video building). |
| Decide each knowledge folder: shared repository or local area | Sufficient. Both are local areas, left in place. |
| Replace `CLAUDE.md` with `agent/` guides | Sufficient (there was none). The project's own local rules (`.local/localrule.md`) became facts in `agent/guide.md`. |
| Validate the converted clone with the installed release before pushing | Sufficient, with a throwaway registry (`wfe --registry <scratch> register`). |
| Push, register, obtain, add to `zulip.json` intake | **Amended.** Intake is automatic for registered projects now; every project also needs a `freework.yaml`. Approvals are written into the workflow files, so the workspace has a commit to push after approving. |
| Write host facts into the workspace's ignored `.local/` | **Amended.** A fresh workspace also lacks the project's ignored `.work/` folders (venvs, weights); the guide must say so, and `.local/localenv.md` must be written before the first run. |
| Check every published file of the first run for host details | Sufficient; clean this time. |

New items:

- **Find the project's working copy first.** pj-jevlab was developed on this host. Its original working copy had 15 unpushed paths, including a whole episode, so the migration would otherwise have forked it. The Developer pushed them and retired that copy.
- **A design agreed before running is the plan.** A `confirms: plan` node is invalid. Plan agreement is the system's `_plan` step, and the guide asks that the plan hold the hypothesis, the criteria and the cost cap.

## What changed in the system

| Change | Why |
| --- | --- |
| `wfrun` guide: a session's background work dies with its turn; wait in the foreground (agautolab) | `study/run-001`'s second attempt was interrupted. arXiv answered 429, and the agent backgrounded its retry and ended its turn. The third attempt, under the new text, waited correctly. |
| Services on the LAN (agdevworld `783cfcb`, released) | The Developer's new local rule. The editor binds every interface and answers any host name, taking browser writes from the page's own origin. nginx publishes `:8093` with the request's Host. The relay binds every interface. The browser finds both on the host it opened the page from. Presentations bind every interface with a LAN `publicBase` and `answerBase`. Tests: editor 261, relay 363. |

## Open

1. **`experiment/run-001` is held, and a held run still holds its workspace.** It wrote a sound plan (a byte-level reproduction of set6 p2's model-free validation). The Developer answered q1 in the browser asking to hold it, and the run was stopped before any work: no process runs. By the contract, a run that has begun and is neither completed nor cancelled holds its workspace, so `wfe request` into `jevlab` is refused until it is resumed and finished, or cancelled (`wfe run cancel`, final). Other projects are not affected.
2. **The Omni Agent cannot act for the Developer by default.** Answering a question for them and resuming a held run were each refused by the permission classifier, until the Developer approved them by hand. That will recur every time unless a permission rule is added.
3. **arXiv rate-limits this host heavily.** The export API and the search pages both returned 429, which cost about 40 s a query. The two general web searches of the original method were skipped for it.
4. **Cosmetic:** `wfe status` and run links still print `http://127.0.0.1:8098/…`; the LAN address is `http://agstudio.local:8093/wfe/`.
5. **Repository size** (about 220 MB of working tree) was left as is. The guide asks that regenerable output go to `.work/` or `.local/`, and no ignore rule was added.
