# Workflow editor p2 — authoring trial report

## Outcome

The Developer reported creating a real-time strategy project and asking the IDE
agent to edit one workflow. Read-only inspection confirmed the resulting project,
workflow, and committed edit. No implementation changes were made during this
review; this report records the evidence and its limits.

The authoring preparation is documented in [pre1/report.md](pre1/report.md).
Unlike pre1's scripted rehearsal, this review concerns the project created during
the Developer's own use. The IDE conversation itself was not inspected.

## Project and workflow

The project is `rts-vs-bot`, displayed as “シンプルRTS（対ボット）”. Its intent is
to build a minimal playable single-player RTS against a bot, using recorded RTS
knowledge to inform design. Goals cover resource gathering, construction, unit
production, combat, autonomous bot economy and attacks, and a decided match outcome.

The root repository is the intended location for game implementation. `devdocs`
and `study/study-rts` are initialized submodules for specifications/workflows and
reusable RTS knowledge respectively. This inspection verified the authoring
artifacts, not an implemented or playable game.

`devdocs/workflows/build-game.yaml` defines one workflow, “Build the RTS”:

1. `study-rts` — study RTS fundamentals and record the findings.
2. `spec` — write the minimum game specification using those findings.
3. `confirm-spec` — agree on scope and technology with the person.
4. `implement` — implement the game loop, entities, input, rendering, and bot.
5. `playtest` — verify a match against the bot and record results and issues.
6. `human-check` — have the person play, report problems or requested changes,
   and decide whether to accept the result.

Repository bindings are editable for the study repository, devdocs, and project
root. The graph is a straight sequence. Its scope agreement before implementation
and human review after testing are consistent with the project's stated intent.

## Confirmed edit and checks

The latest devdocs commit, `b66d484` (`build-game: add human-check talk node`),
adds the final `human-check` talk node, its edge from `playtest`, and its layout
position. The previous commit, `728470a`, introduced the workflow.

The project-root commit `1ad073e` (`Update devdocs: build-game human-check node`)
records the updated devdocs gitlink. Thus the new node is committed both in the
workflow repository and in the parent project's reference to that repository.

Read-only checks used `wfe list --json`, `wfe status --workspace rts-vs-bot --json`,
`wfe validate --workspace rts-vs-bot --json`, the definition files, and Git status,
logs, and the committed diff.

At inspection:

- Project structure diagnostics were empty.
- Workflow validation reported zero errors and zero warnings.
- The root and both submodules had no uncommitted changes.
- Both submodule HEADs matched their recorded parent gitlinks.
- Intent and definition approvals were both `unapproved`; no approval was added
  during the inspection.
- The CLI reported the editor running with the same registry as the inspected
  workspace. This confirms service availability, not what the browser displayed.

## Limits of this result

The evidence confirms project creation, workflow authoring, and a subsequent
committed workflow change following the Developer's reported instruction.

The following were not independently observed in this review:

- The exact IDE prompt, agent tool use, or any difficulties during authoring.
- The Developer making a browser adjustment and saving it.
- The agent reading and continuing from that browser adjustment.
- Live browser reflection of the agent's edit or its latency during this trial.

Consequently, this report does not claim that the entire file → browser edit →
agent continuation round trip was demonstrated in p2. Pre1's automated measurements
remain preparation evidence, not measurements of this human-led trial.

The existing MVP assumptions remain: writers take turns; definition approvals are
separate from acceptance of game results; and the editor does not execute the
workflow. No project files, approvals, or implementation were changed by the
reviewer.
