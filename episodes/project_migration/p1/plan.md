# project_migration p1 — bring pj-agsfx under the workflow editor

## Goal

Take an existing project that was developed outside the system, `trustedagent/pj-agsfx` on Gitea, and make it a managed project of agdevworld's workflow editor: a `project.yaml`, workflows that make its implicit practice explicit, a workspace on this host, Zulip intake, and one real run of each main workflow. Where the system's contract is in the way, change the system; where the project's layout is in the way, change the project. This is the first migration, so the report must say what a second one will need.

Basis: [braindump.md](braindump.md). Decisions taken with the person on 2026-10-13:

- **Everything on Gitea is public and stays public.** Nothing is made private. The old pj-agsfx rule "never commit request text" came from the public wedo-sfx; it is dropped. A request is the run's `braindump.md` and is published with the run, as in every other project.
- **This phase stays on agstudio.** autolab's executor runs here; no second execution host, no ssh to agpc. Work that needs agpc's GPU venvs (CLAP/AST, Audiobox, PAM, Qwen) is out of scope. Sound generation goes through ComfyUI's HTTP API when a run needs it, or is deferred; CPU procedural synthesis runs here.
- **The folder layout follows the system's conventions.** The current layout of pj-agsfx is not preserved; its history and its knowledge are.
- Local knowledge folders are allowed: `study` stays a plain folder of the root repository. Making such a folder a repository of its own later is **not** in this phase.

Out of scope: a converter for other projects, agpc as an execution host, multiple agents, the learned assessors, Google Drive delivery (replaced by presentations), backward compatibility of any kind.

## What was found (2026-10-13)

- pj-agsfx: `study/` (9 surveys, plain folder), `lab/` (experiments and tools: review video, listen UI, drive upload, pre-review runner), `devdocs/episodes/` in AG style (start, generation, flexible_standardize, review_video, composition, auto_prereview), `wedo-sfx/` as a root-level submodule with an **absolute** URL, `CLAUDE.md` as the only agent guide, `todo.md`. No `project.yaml`, no workflows.
- wedo-sfx README is already a request runbook in eight steps: intake, case search, route, produce and self-check (with the p2 pre-review screen), review package, feedback, record, escalate. It is the `create` workflow in prose.
- Research episodes repeat one shape: literature survey into `study/`, local practice in `lab/`, listening audit, then the reusable part into wedo-sfx cards, guide, calibration or an escalation.
- The editor's contract allows a binding only to `.`, `devdocs` or a `.gitmodules` path (`repository-missing`, `shared/validate.ts`). Access is already a path scope (`shared/access.ts`), and publication commits changed paths of the root, so a plain folder needs only to be accepted and shown.
- Installed release: agdevworld `5f7a198` (service and `bin/wfe` agree). Registered projects: `rts-vs-bot`, `ppt`. `gitea.json` owner is `developer`; pj-agsfx lives under `trustedagent`, so registration uses the explicit `owner/name` forms.
- The experiments ran on agpc (RTX 8000, CUDA venvs, headless ComfyUI :8199, Drive via GNOME). agstudio reaches agpc's ComfyUI over HTTP (as mediagen did); it has no CUDA venvs.

## Step 1 — the system accepts local knowledge folders

Change the workflow editor (agdevworld `workflow_editor/`) so a repository binding may point at a plain folder of the project's root repository.

- Contract (`docs/contract.md`, *Repository bindings*): `path` may be `.`, `devdocs`, a `.gitmodules` path, or an existing directory of the root repository that is not inside a submodule. Kind `directory` (the kind devdocs already has in directory mode). A path that is none of these stays `repository-missing`; a declared directory that does not exist in the workspace is `directory-missing` (error: the definition names something the project does not have). Access is unchanged: path scopes, `readonly` / `editable`.
- Workspace inspection (`server/workspace.ts`): list directory bindings' folders as local areas of the root, with `category` from the conventional parent (`study/…` → study, `wedo/…` → wedo, else other) and the root's dirty count scoped to that path.
- Project Editor: show local areas apart from shared repositories (no Gitea link, no "used by other projects"; "local to this project"). Dashboard: unchanged, it lists Gitea repositories only.
- Compile, run snapshot, `wfe run access`, publication: verify they need no change (a directory is part of the root; publication commits the root's changed paths). Write the test that proves it.
- Tests: `test/validate.test.ts` cases for a directory binding (valid, missing, inside a submodule), a workspace fixture with a local `study/` folder in the seed, one browser check of the Project Editor's list.
- Release: `scripts/wfe-release.sh deploy <area>` once `npm run check` passes; RTS and ppt must still validate (`wfe validate` in both workspaces).

Deliverable: a workflow can bind `study/study-sfx` as a plain folder; the editor says what it is; the release is installed.

## Step 2 — the migration commit of pj-agsfx

Work in a clone outside the operating area (`pj-agdev/.local/agdev/migration/pj-agsfx`, ignored), as the RTS migration did. One commit on `main`, pushed to the same Gitea repository (history kept). The layout after the commit:

```text
pj-agsfx/
  project.yaml            ag.project.v2, id agsfx, devdocs: directory
  .gitmodules             wedo/wedo-sfx → ../wedo-sfx.git (relative)
  agent/guide.md          from CLAUDE.md: what is particular to this project
  agent/research.md, create.md, improve-tools.md
  devdocs/
    README.md
    workflows/research.yaml, create.yaml, improve-tools.yaml
    runs/                 empty until step 4
    episodes/             the existing episodes, unchanged
  study/study-sfx/        the former study/ (plain folder; its README and 01–09)
  wedo/wedo-sfx/          the submodule, moved
  lab/                    experiments and tools, unchanged apart from paths
  .gitignore              .local/, __pycache__/
```

- `CLAUDE.md` is removed; its content becomes `agent/guide.md` (the harness reads CLAUDE.md regardless of the run's guides, which would give the agent two guides). `todo.md` moves into `devdocs/README.md` as the project's purpose line.
- Paths quoted inside wedo-sfx (`pj-agsfx/<path>@<hash>` links, the runbook's commands under `lab/…`) stay valid because `lab/` does not move; the runbook's references to `study/` are updated in wedo-sfx (one commit there, pushed first, then the gitlink).
- The "no request text" rule is removed from wedo-sfx's README and from the new guide; the promotion rule keeps its other parts (generalised description, a recipe that regenerates from tracked inputs, the verdict per criterion).
- `drive_upload` is left in `lab/tools` but no guide points at it; presentations replace it (step 3).

Registration, on this host (`WFE_REGISTRY` = the area's registry):

```sh
wfe repo register trustedagent/wedo-sfx --category wedo
wfe project register trustedagent/pj-agsfx
wfe project obtain agsfx            # → .local/agdev/pj-agsfx
wfe status --workspace agsfx        # structure complete, three workflows valid
```

Then `zulip.json` → `intake.projects.agsfx = {workspace: agsfx}`; the notifier creates folder `wf-agsfx` and the channels on first use. Check that the dashboard shows the project, its local area and wedo-sfx (used by agsfx).

Deliverable: pj-agsfx is a registered project with a workspace; `wfe validate` passes for all three workflows; nothing in the repository contains a host path or a token.

## Step 3 — the three workflows and their guides

Written in step 2's commit, agreed with the person before step 4 (definition approvals are theirs).

**research** (`study → practice → explain`; policy `default`; intake `requests`):

| node | type | repositories | description (gist) |
| --- | --- | --- | --- |
| survey | study | study editable, wedo readonly | survey sources on the subject; record reusable findings in `study/study-sfx` with citations; run-specific notes in the run folder |
| practice | do | lab editable, wedo editable, study readonly | try the method locally in `lab/experiments/<run>`; what holds beyond this run goes into wedo-sfx (card, guide row, calibration, escalation) with its evidence |
| explain | talk | study, wedo | explain what was learned and where it is; ask whether the study may finish |

Bindings: `root: .` readonly, `docs: devdocs` editable, `study: study/study-sfx`, `wedo: wedo/wedo-sfx`, `lab: lab`.

**create** (`produce → screen → present → review → record`; policy `default`; intake `requests`):

| node | type | gist |
| --- | --- | --- |
| produce | do | intake and case search are in `plan.md`; route (precedent / analogy / generation / new method) stated there; render the candidates into the run folder |
| screen | do | technical checks and the analytical pre-review stage (S1 + S5) from `lab/experiments/auto_prereview/p2`; write the prediction (`self_eval`) before anything is presented |
| present | do, appearance present/teal | build the review package (review video, or the listen UI as a static page) and register and start it with `wfe present`; the explanation says what to listen for, one question per criterion |
| review | talk, `confirms: result` | ask the person to judge the presented revision; a request for changes loops through an added produce/present pair by adjustment |
| record | do | promote the case into wedo-sfx (case, card evidence, guide row, calibration row); write an escalation when feedback shows a gap |

Bindings: root readonly, devdocs editable, wedo editable, study readonly, lab readonly.

**improve-tools** (`confirm-scope → implement → verify → human-check`; the RTS `improve-game` shape): for `lab/tools` and the pre-review runner. Bindings: lab editable, wedo editable (a runbook line may change), the rest readonly.

Guides (`execution.guides`): `agent/guide.md` (the project, its folders, what each repository holds, the agstudio-only rule: CPU synthesis here, generation through ComfyUI's API when reachable, otherwise say so in the plan and continue with what runs), plus one short file per workflow with what is particular to it. `execution.agent: sfx-agent`, profile as ppt's (`sonnet`) unless the person says otherwise.

## Step 4 — one research run

Request through Zulip (`wf-agsfx.research`, a `request-…` topic). Subject: the cheapest open escalation in wedo-sfx, `symbolic-texture` (a comparison of existing routes for a symbolic wind cue), which needs no GPU venv. Observe: the directory binding in the agent's `wfe run access` answers, publication of `study/study-sfx` and the wedo-sfx gitlink, the explain talk, the final report. Record what the guide failed to tell the agent.

## Step 5 — one create run

A real request from the person in `wf-agsfx.create`. Follow it through produce, screen, presentation (listening in the browser, one click), the result confirmation, and the case promotion. If generation is needed and ComfyUI on agpc answers, use it through the API; if not, the plan says so and the route is procedural or sample playback.

## Step 6 — report

`report.md`: what was migrated and how, what was dropped, the system change and its release, the two runs with their records, what a second migration will need (a checklist, not a tool), and what waits for the agpc episode (GPU stages, a second execution host).

## Checks before deployment or a push

- `npm run check` in `workflow_editor/` and the touched browser checks (step 1).
- `wfe validate` in `rts-vs-bot`, `ppt` and `agsfx` after the release.
- No file pushed to Gitea contains an absolute host path or a token (`git grep` for `/Users/` and `token`).
- Zulip and the executor's state are read through `wfe intake status`, `wfe notify status` and `nctl status` before and after step 2.
