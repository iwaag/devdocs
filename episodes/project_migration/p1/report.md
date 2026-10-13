# project_migration p1 — final report: pj-agsfx under the workflow editor

**Outcome.** pj-agsfx, a sound-effect project built outside the system, is now a managed project of agdevworld's workflow editor. It has three workflows, and one research run and one create run went from request to an accepted result, both published.

The system gained local areas. It also lost two defects that this migration exposed, both fixed and deployed.

Step reports: [report1](report1.md) (local areas), [report2](report2.md) (migration, workflows, registration), [report3](report3.md) (the research run up to the listening question). Plan: [plan.md](plan.md).

| | State |
| --- | --- |
| agdevworld | `6bdcd3b`, installed (releases `15b75cb` → `8178431` → `6bdcd3b` during the phase) |
| pj-agsfx | `trustedagent/pj-agsfx` `522a85a`, public |
| wedo-sfx | `trustedagent/wedo-sfx` `81bc97c`, public |
| Workspace | `agsfx`, idle, clean, nothing unpublished |

## What was done

1. **Local areas (system).** A project declares folders of its root repository in `project.yaml` (`areas:`), and workflows bind them like submodules. This is how `study/` and `lab/` stay inside pj-agsfx without becoming repositories. Turning an area into a repository later was left out, as the braindump asked.
2. **Migration.**
   - Layout: `study/` became `study/study-sfx` (area), `wedo-sfx/` became `wedo/wedo-sfx` (relative URL), `lab/` became an area, and `CLAUDE.md` became `agent/guide.md` plus one guide per workflow.
   - Added: `project.yaml`, a README, and a devdocs README that explains the old episode links.
   - Kept: history; `devdocs/episodes/` unchanged.
   - wedo-sfx's runbook now says that a request is a create run, and that its record is published.
3. **Workflows.**
   - `create`: produce → screen → present → review → record → cleanup, which is the old eight-step runbook as a graph.
   - `research`: survey → practice → explain.
   - `improve-tools`: implement → verify → present → human-check → cleanup.
   - Registered with Gitea, a workspace on agstudio, and Zulip channels `wf-agsfx.*`.
4. **Two real runs**, both on agstudio only (CPU synthesis here, Stable Audio 3 through the ComfyUI service over HTTP):

| Run | Request | Result | Knowledge added |
| --- | --- | --- | --- |
| `research/run-001` | Omni Agent for the Developer: the open escalation "symbolic texture" (wind as a game cue) | Accepted by the person. The procedural swell passed. Stable Audio 3 with a stylisation prompt failed both seeds. MOSS and EzAudio were reported as untried here, not as failing. | study-sfx `10-stylized-game-textures.md`; guide row split (wind: procedural default); two method cards; the escalation narrowed |
| `create/run-001` | The person, from the browser: a plucked string, procedural against generated | Accepted. Inconclusive by construction: the procedural pluck was heard without an instrument body, the generated one with it. | New `string` family: case with recipe, Karplus-Strong card, guide row and "is a body implied?" question, calibration row, escalation |

Agent time was about 13 and 26 minutes. Most of the wall time was the person's listening.

## Defects found and fixed in the system

| Defect | Effect seen | Fix |
| --- | --- | --- |
| Presentations ignored HTTP byte ranges | Safari showed a struck-out play button on the review video; Chromium, used by the checks, played it | `content/` files are served with `Accept-Ranges`, `206` and `416`. Live test and unit test added; documented in `docs/presentations.md`. |
| The browser API compared a run's agent identity with the execution host | Since p6 a workflow can name its agent (`sfx-agent`). For such runs a browser answer was not queued at once (reconcile picked it up 20–40 s later), and a browser result decision was never published: research/run-001's acceptance stayed only in the working tree | The API compares `hostOf(executor)`. A test drives the HTTP API with a named agent and fails on the old comparison. The research decision was then published through the same code path (`research/run-001: result accepted`). |

**A lost commit, recovered.** create/run-001's final publication failed because another Git process held the workspace's `index.lock` at that second. The checkpoint stayed `failed` and resumable, and `wfe exec publish 38` completed it once the lock was gone. The process that held the lock was not identified. A Git client scanning the folder (an editor's Git integration) is the likely cause; one failure does not justify a retry loop.

## Findings that remain

1. **Host details reached the public repositories.**
   - Both runs wrote the ComfyUI address into `plan.md`, and the create run also into a report, although the guide forbade it.
   - The review-video tool writes absolute paths into the presentation's `manifest.json`.
   - Published history keeps them; Gitea is LAN-only, and the host name already appeared in older files.
   - The guide now carries the rule with these two runs as evidence. Whether that suffices is the next run's test.
   - The tool fix (relative paths) is a ready `improve-tools` request.
2. **The Omni Agent cannot request through Zulip.** Its account is a bot, and the intake ignores bots before it reads the proxy statement. Omni used `wfe request`. Whether the intake should honour `people.toml` is a design question, not done here.
3. **A question names its requester, not its listener.** research q3 was addressed to Omni, which cannot listen. The answer form took the person's name, so nothing blocked.
4. **Repository growth.** Each run publishes its presentation (3–5 MB) and its renders in `lab/` (3–4 MB). That is fine for now; many create runs will make the root repository heavy. Moving delivered media to a Git LFS store or the presentation folder alone would be a later decision.
5. **Plan agreement by proxy.** The Omni Agent agreed research/run-001's plan as the Developer's proxy. Listening verdicts were the person's own in both runs, as the evidence vocabulary requires.

## Checklist for the next migration

- Read the project's own runbook first. Its steps are usually the first workflow.
- Decide each knowledge folder: shared repository (`wfe repo register`, submodule with a relative URL) or local area (`areas:`).
- Replace `CLAUDE.md` with `agent/` guides named by each workflow's `execution.guides`; the harness reads `CLAUDE.md` regardless.
- Validate the converted clone with the installed release's workspace code before pushing.
- Push, then `wfe project register <owner>/<name>` and `wfe project obtain <id>`. Add the project to `zulip.json` intake.
- Write host facts into the workspace's ignored `.local/` and tell the guide how to *refer* to them, not just where they are.
- Check every published file of the first run for host details before calling the migration done.

## Deferred to the agpc episode

GPU stages (learned assessors, MOSS, EzAudio), the CUDA venvs, Drive upload, and a second execution host are deferred. The research run's escalation names the untried routes, so the next episode can pick them up from wedo-sfx.
