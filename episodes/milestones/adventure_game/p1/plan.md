# Adventure game p1 — Demonstrate a production cycle guided by human direction and evaluation

Source: [braindump.md](braindump.md).

## Goal and approach

Verify one cycle in the actual environment: the human expresses a concept → identify the necessary research, assets and production methods → build an MVP → the human plays and evaluates it → improve it through the next argue. This plan does not decide the game's content, scale, chapter structure, technology or asset count. Distinguish what p1 demonstrates from fulfilment of the broader long-term development goal.

- The human decides important creative choices, MVP scope, compromises in expression and quality, acceptance of results, whether to continue, change direction or stop, and whether the trial succeeds, with advice from agents. Neither the Omni Agent nor Front may invent or substitute for the human's concept, responses, play evaluation or approval.
- When human judgement is needed, present the artefacts, options and consequences, then wait for a response. Silence is not approval. Research and implementation independent of that decision may continue; routine technical choices and implementation steps do not require repeated confirmation.
- This is a breaking phase in a private experimental environment. Backward compatibility is unnecessary. Implementers choose structure, technology and work order, and may replace code, guides and disposable state as needed. Do not add security hardening or blanket non-destructive or approval procedures. Keep secrets and host-specific facts in ignored `.local/` files under the existing policy.
- Start with Front handling discussion and handoffs, autolab handling research and development, forge producing assets, and cagent inspecting and changing the environment. A fixed workflow engine is not assumed. The Omni Agent observes and repairs infrastructure; record any work it performs on behalf of an in-system agent.

## step1 — Check the current environment and conversation round trips

- Read the prerequisite documents, relevant implementation and latest reports. Inspect service state and observation timestamps through Nautobot or `pj-clusterintent/nctl`. Reuse the existing environment and update or restart only what is needed.
- During execution, tell the human that the trial is starting and where to participate. Verify replies, delegation to other agents and delivery of results through ordinary conversations. Label mechanical test posts as tests; do not treat them as the human's creative input.
- Verify reading, posting and receiving replies in the Arguing Room and Project Room. These checks may coincide with the first real planning exchanges. Fix blocking defects and run tests appropriate to the changes.

Done: the human can participate in discussion and follow requests and replies. Unverified parts are explicit.

Hints: `explicit_reply/p1` updated reply and continuation handling but leaves live posting checks pending. `project_room/p1` also leaves live posting round trips unverified. nctl reporting converged does not prove these exchanges work.

## step2 — Decide the concept and trial scope with the human in argue

- Have the human express their own concept. Front invites autolab, forge, archsage, cagent or others as useful. Do not prefill the presentation format, story, setting or gameplay within the genre.
- Agree with the human on the intended MVP experience, evaluation points, and approximate time and cost budget for this trial. Agents explain options and difficulties and keep important unresolved decisions visible.
- Identify required assets, research questions, and candidate knowledge and workflows. Distinguish verified methods, methods needing trials, and unexplored areas with supporting evidence. Propose small experiments before production where useful.
- Discuss a scope that allows iterative production and continued asset improvement to be observed. If the concept would end with a simple one-shot build, explain the limitation for this trial and leave the scope decision to the human.

Done: the conversation records the human's chosen concept, MVP scope and evaluation criteria, together with unresolved technical questions.

## step3 — Create the project and complete the handoff into production

- Create the project folder, channel and workspace from the agreed direction. If preliminary research is needed, route it to an existing or new study and record its relationship to the game.
- Preserve the reasons behind creative decisions, adopted references, unresolved questions and evaluation criteria where the next worker can read them. The `game` pattern's `main/`, `direction/` and `devlog/` are available as a starting point.
- After workspace preparation, make clear who requests what and which plan leads into execution. For the scope the human has agreed to start, Front hands work into the ordinary workplan/workrun route without seeking the same approval again for each task.
- Verify with real data that Project Room links lead from purpose to plans, execution and results.

Done: work proceeds beyond setup; the agreed research and production begin, and the human can follow progress.

Hints: `agproject open` in `agfront/src/agfront/project.py` creates `pj-<slug>` and its matching channel folder, but performs setup only and does not start production. A `researchplan-` document alone does not execute work either. autolab needs tasks as well as `plan.md`.

## step4 — Research, produce assets, implement and deliver a playable MVP

- In-system agents use ordinary request routes for research, experiments, asset production and integration into the game. When existing methods are insufficient, run small experiments and record success conditions, failures and alternatives.
- Record asset purpose, editable sources, generation and processing methods, adopted versions and in-game uses as needed. Check assets in their actual display, interaction and story context as well as inspecting individual images.
- When important changes to the concept or quality are needed, present comparable prototypes or options to the human. Record agents' technical checks separately from the human's acceptance of expression and experience.
- Deliver the MVP with launch and control instructions, known limitations and the delivered version. Beyond a successful build or completion report, verify that the delivered artefact actually launches and contains the required assets.

Done: the human receives a usable MVP and supporting production, research and asset evidence. If this is not achieved, explain why and let the human decide how to proceed.

Hints: forge's `knowledge list/show/search/path` can access mediagen and `localize/`. `study_import/p1` demonstrated partial asset changes and reuse. Returning findings from generated work into shared knowledge is not automatic; route useful findings back through an autolab workplan or another appropriate route. Do not use Plane or manual runcreate routes from old asset_pipeline reports as current procedures.

## step5 — Receive the human's play evaluation in the next argue and revise

- Have the human actually play, then receive their impressions, what they want to retain or change, and whether they want to continue, in their own words. If their response is pending, leave this step incomplete and wait; do not substitute the Omni Agent's or Front's impressions.
- In the next argue, refer to the same project, the evaluated version, earlier decisions and research findings. Agree with the human on the scope and priority of changes and the criteria for re-evaluation.
- Feed agreed changes back into the plan and carry out necessary additional research, asset revisions and implementation. Make reuse of existing results and reasons for changes traceable. Building a different game from scratch does not complete the continuity trial.
- Return the revised version to the human and have them judge whether it improved. If the human chooses to stop, respect that decision and record that the trial did not reach the full iteration.

Done: the same project completes human evaluation → next argue → updated plan → revised version → human re-evaluation.

## step6 — Review the production cycle with the human and report

- Report the linked conversations, plans, artefact versions and evaluations from concept through revision. Review with the human both the game's success and how far the development cycle worked.
- Briefly record time and cost, stalled handoffs, lost decisions, asset reuse, adoption of research into production, and work performed by the Omni Agent on behalf of others. Full automation is not an acceptance criterion.
- Improve infrastructure and guides based on observed problems, and re-check affected behaviour. Do not add prohibitions or dedicated infrastructure solely for hypothetical problems.
- Write the demonstrated scope and remaining gaps in `report.md`; pending human evaluation remains incomplete. Do not present p1's single cycle as proof of long stories, large asset volumes or sustained long-term development. Commit and push changed repositories and necessary submodule references.

Done: the record includes the human's judgement and concrete findings useful for subsequent production and infrastructure improvements.

## References

- `devdocs/README_DEV.md`, `pj-agdev/.local/devenv.md`, `pj-clusterintent/.local/localenv_memo.md`, `pj-clusterintent/nctl/README.md`.
- Under `devdocs/episodes/`: `argue/p1/report.md`, `argue/p2/ex1/report.md`, `project_room/p1/report.md`, `agentchat/explicit_reply/p1/report.md`.
- `devdocs/episodes/agforge/study_import/p1/report.md`, `pj-agdev/agautolab/agent/project_pattern.md`, `pj-agdev/agautolab/agent/guides/workplan_superdirector/guide.md`.

Treat each report's demonstrated scope as evidence from that point in time, and update it against the code, conversations and environment at execution time.
