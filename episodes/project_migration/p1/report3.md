# project_migration p1 — step 4: the first research run (in progress)

**Run**: agsfx `research/run-001`, requested 2026-10-13 05:27 UTC. **State**: waiting for the person's listening verdict (q3, node `present`).

## How it was requested

- **Request.** The Omni Agent requested it for the Developer through agdevworld's entrance, `wfe request research --author "Omni Agent (for Developer)"`. The subject is the open wedo-sfx escalation `20261007-symbolic-texture`: which route makes wind read as a game cue rather than as realistic wind.
- **Why not Zulip.** The Omni Agent's Zulip account is a bot (user 9, `is_bot`), and the intake ignores bot posts by its first classification rule. The people statement that makes Omni the Developer's proxy (`~/.config/agag/people.toml`) is not read by the intake. A Zulip request from the person works as configured (requester 8); the create run in step 5 uses it.
- **Plan agreement.** The plan was agreed by Omni as proxy. The listening verdict is not Omni's to give: only a listener's verdict counts.

## What the run did (about 13 minutes of agent time)

| Seq | What |
| --- | --- |
| `_plan` | plan.md matching the request. The agent added `present` and `cleanup` nodes from the person's catalog by adjustment. |
| q1 / q2 | Omni agreed on the condition that the ComfyUI host and port leave plan.md. The agent fixed it and asked again, as its contract requires; Omni confirmed. |
| `survey` | `study/study-sfx/10-stylized-game-textures.md` (137 lines, sources linked), README index updated |
| `practice` | 5 renders in `lab/experiments/research-run-001/`: the procedural engine as-is and exaggerated (CPU), Stable Audio 3 Small-SFX with a stylisation prompt (2 seeds) and the plain-prompt failing reference through the ComfyUI service; a standalone API script that reads the service URL from `.local/localenv.md`. MOSS and EzAudio are reported as untried here (GPU venvs), not as failing. |
| `present` | the review video (5 items, 32 s) registered and served as presentation p1, link checked |
| q3 | the listening question, waiting |

The local area bindings worked as intended: survey wrote `study/study-sfx`, practice wrote `lab/experiments/…`, and both were published with the root at each human wait.

## Findings so far

1. **A host address reached a published file.** plan.md named the ComfyUI host and port, copied from `.local/localenv.md`, and was published at the first wait (pj-agsfx `17ed898`). The guide said that host details stay in `.local/`, but the agent still copied the address. It was fixed forward in plan.md; the published history keeps it. The Gitea server is LAN-only and the host name already appeared in older files.
2. **Absolute host paths in a presentation manifest.** `lab/tools/review_video/build.py` writes absolute `source` and `input_manifest` paths into `manifest.json`. That file is part of the presentation and was published (pj-agsfx `cef251d`). This is a tool defect, a natural first `improve-tools` request: record paths relative to the project root.
3. **The question is addressed to whoever requested.** q3 names "Omni Agent (for Developer)" because Omni filed the request, though only the person can answer it. The run view's answer form takes any name, so this does not block, but a listening question should name the listener.
4. **Publication size.** The presentation folder is 5 MB (review video plus sources), and the renders in `lab/` are about 4 MB. Both are published into the root repository. That is acceptable for one run. Many create runs will grow the repository quickly, which is worth watching.

## Waiting on the person

Open the run view, play the 32-second video, and answer q3 with a pass or fail and the reason per item (W1–W5):

    http://localhost:8093/wfe/#/ws/agsfx/run/research/run-001

The agent then records the verdicts, stops the presentation, updates the guide row, the method cards and the escalation, and asks whether the research may finish.
