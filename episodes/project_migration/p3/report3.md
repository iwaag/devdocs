# project_migration p3 — step 3: registered and prepared

Written 2026-10-14. forge is a registered project with a prepared workspace. No system change was needed, so the installed release (agdevworld `75cdae7`) is used as it is.

## Done

| Item | State |
| --- | --- |
| Gitea | `developer/pj-forge` (id 139), created through the editor's Gitea module. Pushed `b7a79dc` with the module's credentials. |
| Registry | `wfe project register developer/pj-forge`, then `wfe project obtain forge`, giving workspace `forge` (`.local/agdev/pj-forge`) at `b7a79dc`, clean. A copy of the registry from before is in the ignored step notes. |
| Workflows | 4 valid, 0 warnings. Intent and definition approvals are left unrecorded: an approval is the person's statement and is not a gate for runs, so the Omni Agent does not write it in their name. |
| Zulip | The notify thread provisioned `wf-forge.develop-capability` (#295), `wf-forge.freework` (#296), `wf-forge.provide` (#297) and `wf-forge.refresh-catalog` (#298). `wfe intake due` lists all four with no problems. |
| Local context (ignored) | `pj-forge/.local/forge.env` (0600, copied file to file from the step 2 clone, values never shown), `sources.toml` (5 sources), `comfy/` (3 workflow exports), and `localenv.md`, which records the service notes, host tools, source caveats and how presentations are reached |
| Readable from a run | A real Git worktree at the run-copy location (`.local/wfe/runs/_probe/run-000/tree`, removed afterwards), with the executor's launchd `PATH` and an empty environment otherwise: `tools/forge local` resolved the workspace's `.local/`, found all seven settings, `sources.toml`, the three presets and `localenv.md`, and `tools/forge sources` read the revisions. `tools/.venv` is ignored, so a run's tool environment is not saved to its branch. |
| Result viewing | Presentations bind every interface on ports 8300–8399 with LAN links (`present.json`). The editor answers at `http://agstudio.local:8093/wfe/` (HTTP 200, version `75cdae7`). The forge run views are under `#/ws/forge/…`. |

## How requests reach forge in this phase

- **The person:** a `request-…` topic in `wf-forge.provide` (or another `wf-forge.*` channel), or *Request a run* in the editor's project view.
- **The Omni Agent, for the person:** `bin/wfe request <workflow> --braindump <file> --author "Omni Agent (for Developer)" --workspace forge`. Omni's Zulip account is a bot and the intake ignores bots (p1).
- Front does not route to forge in this phase. Old forge's channel keeps running until step 5.

## What the person sees

Each run asks its questions (plan agreement, then result confirmation) in its request topic, or in the run view for a CLI request. Each `provide` run serves a review page through a presentation, and its link is in the question.
