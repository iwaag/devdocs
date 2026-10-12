# Workflow editor p7 — step 6: the RTS workflow and its review screen

Commits: `developer/pj-rts-vs-bot` `72b69cf` (pushed to Gitea), agautolab `d31f654`.

## Changes

- **`improve-game`** (`devdocs/workflows/improve-game.yaml`, still `ag.workflow.v2`, default documentation policy).
  - Graph: `implement → verify → present → human-check → cleanup`.
    - `present` is a `do` node with appearance `present`/`teal`.
    - `human-check` is the talk node with `confirms: result`.
    - `cleanup` is a `do` node with appearance `cleanup`/`slate`.
  - The intent now says that the review is one link to the game and its request-specific explanation on one screen, and that the run ends after the presentation server is stopped and verified.
  - Compiled run graph: `_plan → implement → _report.implement → verify → _report.verify → present → _report.present → human-check → _report.human-check → cleanup → _report.cleanup → _final-report`. Cleanup precedes the final report, so the report can state the shutdown outcome. Planning, per-node reports and the final report are unchanged in kind.
  - `execution.guides: [agent/review.md]`; agent and profile stay the host's defaults (autolab, its default profile).
  - `wfe validate improve-game`: valid.
- **Project guide** `agent/review.md` (rts-vs-bot). It holds the reusable delivery behavior, so a braindump only says what to change:
  - what the person reviews: the frozen `game/`, and an explanation in the language of the braindump (what changed in terms of play, how to try it, what to check, and what was and was not verified — keeping the `improve-game/run-002` fact that the executor had no browser);
  - the facts of serving: register from the running `present` node, start, the review question `--presentation`, the decision from the confirming answer, a correction as an adjustment, cleanup's verified stop;
  - what the final report states: the delivered revision, verification, the decision, and the shutdown outcome.
- **Common contract** `agautolab/agent/guides/wfrun/guide.md`. A new *Presentations and changes to the run's future work* section states, as facts:
  - a presentation survives the session;
  - review answers bind a revision;
  - completion waits for a verified stop;
  - cancel stops presentations and hold does not;
  - adjustments change only future work, need the person's records, and never reopen finished work.

  The guide is read at every claim, so wfexec needs no restart; each attempt records its hash.
- **Review screen.** It is the generic one from step 3: the game in a frame and the explanation beside it, with a link back to the answer location. For RTS runs, which are requested in agdevworld, that location is the run view (`present.json` `answerBase`, set on this host in step 7). Feedback and acceptance stay in the run's existing question; there is no other decision endpoint.
- **Other RTS definitions.** `build-game` is still `ag.workflow.v1`. It has been unsupported since p5 and has not been run since p4. It was left as it is and not revived.

## Verification

- `wfe validate improve-game` (installed release `0c2c072`): valid. The old release ignores `appearance`, as designed.
- The behavior the workflow relies on — presentation, review question, decision, cleanup, completion after a verified stop, correction by adjustment — was verified in isolation in steps 3–5 (`test/present.test.ts`, `test/adjust.test.ts`, `checks/review.ts`). Step 7 deploys it and runs a live smoke with the real game.

## Limitations

- The game takes keyboard input only after the person clicks into the frame. The explanation says so; "open alone ↗" opens the game in its own tab.
- Until the release in step 7, a run requested now would get the new workflow but an editor without `wfe present`. No run was requested in between: the catalog had no open run, and step 7 followed directly.

## Next

Step 7: tests, deployment and the release record.
