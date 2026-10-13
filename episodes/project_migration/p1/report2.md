# project_migration p1 — steps 2 and 3: pj-agsfx joins the workflow editor

**Repositories** (Gitea, public, history kept): `trustedagent/pj-agsfx` at `affceb5`, `trustedagent/wedo-sfx` at `f816d68`.
**Registry**: project `agsfx`, shared repository `trustedagent/wedo-sfx` (wedo), workspace `agsfx` at `pj-agdev/.local/agdev/pj-agsfx`.

## The project before

- `study/` held 9 surveys as a plain folder. `lab/` held experiments and tools. `devdocs/episodes/` held six episodes in AG style.
- `wedo-sfx/` was a root-level submodule with an absolute URL.
- `CLAUDE.md` was the only agent guide, and `todo.md` the purpose line.
- There was no `project.yaml` and no workflow.
- The runbook in `wedo-sfx/README.md` already described the request workflow in eight steps.
- Raw requests and listener comments were kept unpublished in `.local/requests/`, because wedo-sfx is public.

## The migration commit

One commit on `main`, made in a clone under the operating area's ignored `migration/` folder, as the RTS migration was.

| Before | After |
| --- | --- |
| `study/` | `study/study-sfx/`, a local area |
| `wedo-sfx/` (absolute URL) | `wedo/wedo-sfx/`, URL `../wedo-sfx.git` |
| `lab/` | unchanged, a local area |
| `CLAUDE.md` | `agent/guide.md` plus one guide per workflow |
| `todo.md`, empty `.vscode/settings.json` | removed; the purpose is in `project.yaml` |
| — | `project.yaml` (`ag.project.v2`, `devdocs: directory`, `areas`), `README.md`, `devdocs/README.md`, `devdocs/workflows/*.yaml` |
| `devdocs/episodes/` | unchanged; `devdocs/README.md` says that their links use the old paths |

- **Links.** Markdown links in `lab/` to `study/` and `wedo-sfx/` were rewritten to the new paths. Links inside the episodes and the `pj-agsfx/<path>@<hash>` links in wedo-sfx stay as written; they were right at their commit.
- **CLAUDE.md removed.** The harness reads a `CLAUDE.md` whatever guides a run is given, so keeping it would have handed the agent two guides.
- **wedo-sfx `f816d68`.** The runbook now says that a request is a `create` run of pj-agsfx. The run keeps the request, renders and answers, and is published. The promotion rule links the run instead of a local request id. The rule against committing request text is gone, by the Developer's decision that everything stays public.

## Workflows and guides (step 3)

Every workflow has `documentation.policy: default`, Zulip intake, and `execution.agent: sfx-agent` with the host's default profile.

| Workflow | Nodes | Editable |
| --- | --- | --- |
| `create` | produce → screen → present → review (`confirms: result`) → record → cleanup | wedo, lab |
| `research` | survey (study) → practice → explain (talk) | study, wedo, lab |
| `improve-tools` | implement → verify → present → human-check (`confirms: result`) → cleanup | lab, wedo |

- **create** is the wedo-sfx runbook as a graph. The pre-review screen and the prediction come before the listener hears anything. `record` does the case promotion, calibration and escalation.
- **research** is the shape that `auto_prereview` and `flexible_standardize` repeated: sources into study-sfx, practice in `lab/`, reusable results into wedo-sfx. A listening check is added by adjustment when the question needs one.
- **improve-tools** follows RTS `improve-game`, for `lab/tools` and the pre-review runner.
- **Not a workflow.** Escalation: an escalation file that a create run writes becomes the next research run's request.
- **Guides.** `agent/guide.md` states the host facts: a Mac without CUDA, so CPU synthesis runs here and generative models are reached through the ComfyUI service by HTTP. It also says that the GPU venvs, learned assessors and Drive upload are not available, that verdicts come only from a listener, and that host details stay in `.local/`.
- Definition approvals were not recorded; they are the person's to give.

## Registration and host setup

```sh
wfe repo register trustedagent/wedo-sfx --category wedo
wfe project register trustedagent/pj-agsfx
wfe project obtain agsfx
```

- **Status.** `wfe status --workspace agsfx` reports structure complete, all three workflows with 0 errors and 0 warnings, and both areas and the submodule present.
- **Dashboard.** It lists the project and `trustedagent/wedo-sfx` "used by agsfx".
- **Zulip.** `zulip.json` intake gained `agsfx → workspace agsfx`. The notifier provisioned folder `wf-agsfx` and channels `wf-agsfx.create`, `.research` and `.improve-tools`.
- **Host notes.** `pj-agsfx/.local/localenv.md` (ignored) records the ComfyUI service address, the Stable Audio 3 checkpoints it has (checked 2026-10-13), a Japanese-capable font for the review video tool, and what is not on this host.
- **Push credentials.** The `developer` token had push rights on the `trustedagent` repositories, so no other credential was needed.

## Checks

- **Before the push.** The migration clone was validated with the release's own workspace code: no project issues, and 0 errors and 0 warnings in all three workflows.
- **Host details.** No added line contains a host path, a host name, an address or a token.
- **Identity.** A local wedo-sfx commit was first made with an explicit author email. It was amended to the host's configured Git identity before the push.
