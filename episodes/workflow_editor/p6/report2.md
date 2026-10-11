# p6 step 2 — pj-ppt and its two workflows

Plan: [plan.md](plan.md) §Step 2.

## What was created

Everything was created through the existing mechanisms:

- `wfe create` with the area's Gitea setting;
- `wfe repo create`;
- `wfe repo add`.

devdocs is in directory mode, the default; nothing called for a separate repository.

| Resource | Gitea | In the workspace | Commit |
| --- | --- | --- | --- |
| Project `ppt` ("PowerPoint agent"), workspace `ppt` | `developer/pj-ppt` (registered `gitea:133`) | `.local/agdev/pj-ppt` | `2f45462` create, `5d3b662` workflows and guides |
| `study-ppt` (category study) | `developer/study-ppt` (`gitea:134`) | `study/study-ppt` (submodule, `../study-ppt.git`) | `815b8a1` README with layout |
| `wedo-ppt` (category wedo) | `developer/wedo-ppt` (`gitea:135`) | `wedo/wedo-ppt` (submodule, `../wedo-ppt.git`) | `e888747` README with layout |
| Runs | — | `devdocs/runs/<workflow>/<run>/` | created by requests |

`wfe status --workspace ppt` reports structure complete and both workflows valid (0 errors, 0 warnings). `wfe dashboard` lists `ppt` and both shared repositories, each used by `ppt`. The intent and definition approvals are unrecorded; they are statements, not gates (docs/runs.md).

The running notifier picked up the newly registered project on its next pass and provisioned the Zulip places, as designed in p5:

- folder **#30 `wf-ppt`**;
- channel **#275 `wf-ppt.create`**;
- channel **#276 `wf-ppt.study`**.

These channels are where requests will be made; no run exists yet.

## Workflows

Both declare `documentation.policy: default`, so planning talk, per-node reports and the final report are the generated steps. None is authored twice.

| Workflow | Authored nodes | Read / write scopes |
| --- | --- | --- |
| `study` | `research` (study) → `explain` (talk: explain findings, ask permission to finish) | root readonly · `study/study-ppt` **editable** · `wedo/wedo-ppt` readonly |
| `create` | `create-slides` (do) → `verify` (do: content and rendered appearance) → `present` (talk, `confirms: result`) | root readonly · `study/study-ppt` readonly · `wedo/wedo-ppt` **editable** |

Each workflow also carries the settings that step 3 makes effective:

```yaml
execution:
  agent: ppt-agent
  profile: sonnet
  guides: [agent/guide.md, agent/<study|create>.md]
intake:
  zulip: requests
```

The `present` node carries `confirms: result`, the designated result-confirmation talk for step 6. Until step 3 models these fields, the deployed editor reads past them; they are preserved on save.

### Agent guides (in the project root, read-only to runs)

| File | Content |
| --- | --- |
| `agent/guide.md` | Who ppt-agent is, what it can see (both shared repositories, host tools), and these facts: shared knowledge holds only what outlives a run and names its source; read knowledge and exercised knowledge are kept apart; renderer evidence is not PowerPoint evidence; deliverables live in the run folder because a machine path is not a delivery |
| `agent/study.md` | Findings go to `study-ppt`, one topic per file. A generation study ends with one recommended generation approach and one rendering approach for this host. Permission to finish is the answer to `explain` |
| `agent/create.md` | The plan names audience, purpose, scope, approach and verification. Deliverables (PPTX, preview, source) go in the run folder. Verification is slide by slide on the rendered preview. Procedures and lessons go to `wedo-ppt`. `present` is the result confirmation |

The shared repositories' READMEs define their layout:

- `study-ppt`: `topics/`, `recommendations.md`;
- `wedo-ppt`: `examples/<name>/`, `procedures/`, `lessons.md`.

Each entry names the run it came from.

## Artifact locations (declared)

| Artifact | Location | Identifiable by |
| --- | --- | --- |
| Original request, plan, dialogue evidence, node reports, final report | `devdocs/runs/<workflow>/<run>/` (`braindump.md`, `plan.md`, `run.json`, `report-*.md`, `report.md`) | run folder |
| PPTX, preview (PDF/PNG), generating source | the create run's folder | run folder; published by the execution host's checkpoints, linked by commit |
| Reusable study knowledge | `study/study-ppt` | each change names `ppt study/run-NNN` |
| Exercised procedures, examples, lessons | `wedo/wedo-ppt` | each change names `ppt create/run-NNN` |

## Host facts that matter for the real runs

- **Present:** `uv`/`uvx`. `uvx --from python-pptx` gives python-pptx 1.0.2 without installing anything. Also present: `node`/`npx` and `pdftoppm`.
- **Absent:** LibreOffice (`soffice`), PowerPoint, Keynote and ImageMagick. Rendering a PPTX to a preview therefore needs something the host lacks today, such as LibreOffice natively or in a container. The plan leaves this choice to the study run, which says so in its guide. Installing a renderer is a host change; if the study recommends one, I will ask the person before installing it (step 8).

## Pushes

All four repositories are pushed: the root and both shared repositories, through the user's Git credential helper. The root records the shared repositories at their pushed commits.

## Next entry point

Step 3: model `execution`, `intake` and `confirms` in the workflow contract, and compose the execution instructions from the common contract, the project guides and the brief. Then record the effective settings with each attempt.
