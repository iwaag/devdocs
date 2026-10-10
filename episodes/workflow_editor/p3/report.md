# workflow_editor p3 — human-led execution trial

## Result

The person and an IDE agent completed one real workflow run and produced a
playable RTS MVP. The run preserves the person's braindump, a fixed workflow
definition, the plan and reports, node transitions, questions and answers, and
the person's acceptance. The browser shows the completed execution and decision.

This is the human-led p3 trial, not the synthetic pre1 rehearsal. The person
reported completion and asked for a review; the reviewer independently inspected
the saved records and Git history, reran the headless checks, and exercised the
game and run view in Chromium. The reviewer did not observe the original session
continuously or measure its live-update latency.

Preparation and its automated evidence are in [pre1/report.md](pre1/report.md).
The implementation remains in `pj-agdev/experiments/workflow_editor/`.

## Trial and durable evidence

| Fact | Recorded value |
| --- | --- |
| Project | `rts-vs-bot` |
| Workflow / run | `build-game/run-001` |
| Input | Human braindump: build a game MVP after a light study of RTS design, with a minimal bot opponent; technology unrestricted |
| Executor | Claude; recorded backend `claude-code / claude-opus-5-5` |
| Final record | Sequence 24, all six nodes completed, result accepted by iwaag |
| Execution duration | Approximately 18 minutes, including human waits |

Project-relative evidence:

```text
devdocs/build-game/runs/run-001/
  braindump.md
  plan.md
  run.json
  definition/build-game.yaml
  spec.md
  report-implement.md
  report-playtest.md
  screenshots/
study/study-rts/notes/01-rts-core.md
study/study-rts/notes/02-rts-bot-ai.md
game/
```

These references describe the trial project; its files are not copied into this
episode repository. Its workspace is in an ignored authoring area.

The workflow was linear:

`study-rts → spec → confirm-spec → implement → playtest → human-check`.

1. **Study and specification.** The agent recorded a lightweight study based on
   general RTS knowledge, then proposed a small game using plain JavaScript and
   Canvas, without build tools or dependencies.
2. **Specification agreement.** `confirm-spec` asked q1 and waited. The person's
   reply, "ok、進めてください。", was recorded by the agent, then taken up before
   the node completed and implementation started.
3. **Implementation and playtest.** The agent implemented simulation, bot,
   rendering/input, a headless test, and play instructions. Changes from the
   agreed specification, including slower mining and a weaker bot, are described
   in the implementation and playtest reports and progress history.
4. **Human decision.** `human-check` asked q2 and waited. The person's reply,
   "MVPとして合格。runは完了としていい。", was recorded and taken up; the node
   completed, followed by an explicit acceptance record.

Both replies were relayed through CLI records. This trial demonstrates the IDE
conversation route; it does not independently demonstrate a human submitting
answers through the browser.

## Review checks

### Run integrity, UI, and Git history

- `wfe run check build-game/run-001` reports the record as usable. Two artifact
  warnings remain, explained below.
- The fixed definition matches the current source definition. The recorded
  transitions follow dependencies and keep asking, answering, taking up, node
  completion, and result acceptance distinct.
- The browser run view shows sequence 24, six completed nodes, no active or
  waiting nodes, and the accepted result. No browser page errors occurred.
- Reading devdocs commit `bfa93c4` reconstructs sequence 7: study and spec are
  complete, `confirm-spec` waits on q1, and implementation has not started.
- Devdocs commit `0da5de0` records completion and acceptance. Project commit
  `82f5471` records that devdocs gitlink; the game and study are also committed.
  The project, devdocs, and study worktrees were clean at review.

This verifies the agreed persistence model on real work: the executed workflow
and execution stage can be recovered at a commit, with timestamped history
retaining the transitions between commits.

### RTS MVP

The implementation has ore collection, construction, worker and soldier
production, movement/combat, an autonomous bot, a building-destruction win
condition, and restart. It opens directly from `game/index.html`.

The reviewer reran `node game/test/headless.js`; all 13 checks passed:

| Match | Winner | Simulated duration |
| --- | --- | --- |
| Passive player vs bot | Bot | 2.89 min |
| Bot vs bot | Red side | 5.31 min |
| Defend-then-attack plan vs bot | Player side | 5.87 min |
| Rush plan vs bot | Bot | 4.84 min |

The command checks also cover construction cost/refund, invalid ownership,
barracks construction, soldier production, and ore delivery.

Independent browser checks used real mouse/keyboard input for worker production
and cancellation, barracks placement, soldier production, attack move, pause,
and restart. Simulation time was advanced through the game's existing test
hook. The bot defeated the test player, the result was rendered, and restart
restored the starting state. No page errors occurred.

The original playtest report records a browser victory using UI automation.
That full browser victory was not rerun by the reviewer; player-side victory
was independently reproduced in the headless match above. The person's actual
acceptance supplies the human usability result.

## Observations and remaining limits

- **Incorrect artifact paths remain visible after correction.** The playtest
  outcome initially used `screenshots/victory.png` and `screenshots/defeat.png`
  as project-relative paths. The actual files are under the run folder. The
  agent attached their correct paths and added a correction note, but the
  earlier references remain as two missing-artifact warnings. The files exist,
  the run is usable, and its completion is unaffected. This is concrete
  evidence for clearer path help or a future explicit artifact-reference
  correction operation; silently rewriting history is unnecessary.
- **The editor process needed a restart after pre1.** An older running API
  lacked the run routes. The old process was stopped before the trial. The
  reviewed browser subsequently served the completed run correctly. Setup
  instructions already explain restarting after an editor update.
- **One linear run is the demonstrated scope.** This trial did not exercise
  delegate/request creation, parallel branches, readonly enforcement, failed
  work recovery, or unattended multi-hour waits. Pre1's synthetic checks cover
  several of those record semantics; they are separate evidence.
- **Game limitations remain documented.** Movement has no pathfinding and can
  jam around buildings. Bot-vs-bot play favors the red side in the recorded
  match, with the cause uninvestigated. Difficulty and game polish remain
  adjustable beyond this accepted MVP.
- **Progress remains reported progress.** The successful IDE trial does not
  establish process-health monitoring, automatic resumption, or Zulip/Observer
  integration. The existing future integration boundary still applies.

## Completion

P3's requested first human-led execution trial is complete for this workflow:
real input produced a playable result through recorded study, work, human
agreement, and acceptance, and the resulting state is visible and durable.

Deus ex machina note: the reviewer inspected records and independently tested
the result; no work belonging to the executing agent was performed for it.
