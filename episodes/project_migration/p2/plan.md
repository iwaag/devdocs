# project_migration p2 — bring pj-jevlab under the workflow editor

## Goal

Take a second existing project, `trustedagent/pj-jevlab` on Gitea, and make it a managed project of the workflow editor, using p1's checklist as the only guide. The point is to find out whether the checklist is enough for a project nobody prepared for the system. Anything it lacked goes into the report as a corrected checklist.

Basis: [braindump.md](braindump.md) and [../p1/report.md](../p1/report.md) (its "Checklist for the next migration").

Decisions taken with the person on 2026-10-13 (the person accepted the proposal as written):

- **The layout is kept.** `study/`, `lab/` and `devdocs/` do not move; 200+ commits of reports cross-reference these paths. Both folders become local areas in place.
- **Three workflows** (plus the system's `freework`): `study`, `experiment`, `improve-tools`. Video presentations are made by the `experiment` workflow's explain node as a presentation, not by a workflow of their own.
- **Execution check stays light**: one real `study` run. `experiment` is checked up to its plan agreement unless step 3 finds a cheap recomputation worth running.
- **Everything on Gitea stays public**, as in p1. This phase runs on this host only.

Out of scope: re-running the heavy experiments, a converter for other projects, a second execution host, turning an area into a repository, backward compatibility of any kind.

## What was found (2026-10-13, read-only)

- `pj-jevlab`: public, 207 commits, `main`. Root contains `study/`, `lab/`, `devdocs/episodes/`, `todo.md`, `.gitignore`. No submodule, no `CLAUDE.md`, no `project.yaml`, no workflows, no agent guide. `todo.md` names a `fruits/` folder that does not exist.
- Episodes show three implicit workflows: surveys into `study/` (`study/*`, with reinvention checks), experiments with a plan, a pre-registered design and a cap on requests and hours (`experiments/set1`–`set6`, `benchmark`, `game_test`), and tool or presentation building (`viewer`, `video_presentation`).
- Experiments call local model servers by loopback address and use Apple-silicon runtimes; weights and checkouts live in ignored `.work/` folders and are not in the repository. What this host can re-run is unknown until step 0.
- The working tree is about 220 MB (`.npz` results, a 16 MB request cache) against a 30 MB pack. About 250 tracked files contain an absolute home path, a host name or an address; most are logs, tables and reports.
- `.local/localrule.md` is referenced by plans but is ignored and absent from the clone. Its content is unknown.
- (Step 0) Every project has a `freework` workflow, and Zulip intake is automatic for registered projects. The project's original working copy sits on this host and holds unpushed work. See [report0.md](report0.md).
- (Step 1) The contract supports what is needed: `wfe validate` passes on a trial; the one visible difference is that a top-level `study` area is categorised *other*. See [report1.md](report1.md).
- The contract already supports what is needed: `areas:` in `project.yaml` (`docs/contract.md`, *Local areas*). Category follows the path (`study/…` study, `wedo/…` wedo, otherwise other), so a top-level `study` is shown as other.

## Step 0 — preflight

Read-only; changes nothing.

- Installed release and `wfe status` for the existing projects; Zulip intake and notifier state (`wfe intake status`, `wfe notify status`); `nctl status` for services this plan touches.
- Which model servers and weights the experiments assume (`lab/**/README`, `plan.md`), and which of them this host has. The result decides step 3's second run.
- Ask the person for `.local/localrule.md`, or record that the project proceeds without it.

Deliverable: `report0.md`.

## Step 1 — gap check against the contract

In a scratch clone, add a trial `project.yaml` (`areas: study, lab`) and an empty workflow, and run the installed release's `wfe validate`.

- Expected: passes. Record what does not: top-level area as `study` (category), a folder that is both an area and contains nested READMEs, devdocs in directory mode with existing `episodes/`.
- If something fails, change the system only as far as this project needs, release it with the usual checks (`npm run check`, `wfe validate` in rts-vs-bot, ppt and agsfx), and say so in the report. Otherwise change nothing.

Deliverable: a gap table in `report1.md`; a release only if a gap demands one.

## Step 2 — the migration commit

Precondition (report0): the person decides what happens to the unpushed work in the original working copy before the commit is made.

Work in a clone outside the operating area (ignored, under the area's `migration/` folder), as p1 did. One commit on `main`, pushed to the same repository; history kept.

```text
pj-jevlab/
  project.yaml            ag.project.v2, id jevlab, devdocs: directory,
                          areas: study, lab
  agent/guide.md          purpose, folders, what each holds, host rules
  agent/study.md, experiment.md, improve-tools.md
  devdocs/README.md       purpose line from todo.md; explains the old layout
  devdocs/workflows/study.yaml, experiment.yaml, improve-tools.yaml,
                          freework.yaml
  devdocs/runs/           empty until step 3
  devdocs/episodes/       unchanged
  study/, lab/            unchanged
  .gitignore              extended (see below)
```

- `todo.md` is absorbed into `devdocs/README.md`; the `fruits/` line is dropped unless the person wants the folder.
- **Host details.** Before the push, `git grep` the tree for home paths, host names, addresses and tokens. Sort the hits: secrets (none expected) are removed; paths in historical logs and reports are listed and left as published history, as p1 did, unless the person asks otherwise. The guide tells the agent to refer to hosts by role, not by address.
- **Size.** Decide in this step, with the person, how results are kept from now on: the `.gitignore` gets patterns for regenerable caches and large raw outputs of new runs; tracked history stays.
- The guide states the execution rule found in step 0 (what runs here, what is only reported as untried).

Workflows (policy `default`, intake `requests`; `execution.agent` and profile as ppt's unless the person says otherwise):

| workflow | nodes | notes |
| --- | --- | --- |
| `study` | survey (study) → synthesize (do) → explain (talk) | sources and claims separated from independent measurements; findings go to `study/` with citations; run notes in the run folder |
| `experiment` | run (do) → analyze (do) → report (do) → explain (talk, `confirms: result`, presentation allowed) | the system's plan step is the design: the guide requires the hypothesis, the criteria and the **cap on requests and hours** in the plan, and plan agreement (`_plan`) is enforced by the system; a `design` node with `confirms: plan` is not valid (`confirms-kind`) and is not needed |
| `freework` | talk (system default) | bound to `study` and `lab` editable, as in agsfx and ppt |
| `improve-tools` | implement → verify → present → human-check → cleanup | the p1 shape, for `lab/viewer` and the video builder |

Bindings: `study` editable in `study`, `lab` readonly; `experiment` editable in `lab`, `study` readonly; `improve-tools` editable in `lab`. Definitions are agreed with the person before step 3.

Registration, on this host:

```sh
wfe project register trustedagent/pj-jevlab
wfe project obtain jevlab
wfe status --workspace jevlab      # structure complete, three workflows valid
```

Intake is automatic for registered projects: check that `wfe intake status` lists the `jevlab` channels and that the dashboard shows the project and its two local areas.

Deliverable: pj-jevlab is a registered project with a workspace; `wfe validate` passes for all three workflows; nothing newly committed contains a secret.

## Step 3 — real runs

**3a. One `study` run**, requested through Zulip (`wf-jevlab.study`). Subject: a narrow follow-up to `study/recent_study` (papers or releases newer than its cut-off date), so the run can be judged against an existing survey. Observe: the area bindings in `wfe run access`, publication of `study/`, the explain talk, the final report, and what the guide failed to tell the agent. Then check every published file for host details (p1's last checklist item).

**3b. `experiment` up to design.** Request a small follow-up whose design needs the person's judgment. Observe the plan agreement and the cap statement only. Execute it only if step 0 found a cheap recomputation from cached results, and say so in the report either way.

## Step 4 — report

`report.md`: what was migrated and how, what was dropped, whether any system change was needed (step 1), the runs and their records, a **corrected checklist** (each p1 item marked sufficient / amended / new), and what a third migration or a heavy-experiment project will need.

## Checks before deployment or a push

- `wfe validate` in `rts-vs-bot`, `ppt`, `agsfx` and `jevlab` whenever the system or a project changes.
- `npm run check` in `workflow_editor/` if the system is touched.
- `git grep` for home paths, host names, addresses and tokens in everything newly pushed.
- Zulip and executor state read through `wfe intake status`, `wfe notify status` and `nctl status` before and after step 2.
