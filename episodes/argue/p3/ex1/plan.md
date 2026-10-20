# Argue p3 ex1 — guided discussion and structured execution

## Objective

Implement the experiment in [braindump.md](braindump.md): the Omni Agent
acts for the Developer, giving instructions and hints during an argue so
the project agents can develop a structured path from discussion to work.

The intended pattern is that jevlab contributes relevant existing
literature and knowledge, rts-vs-bot explains the game and its integration
surface, and Front uses that shared information to decide whether more
discussion is needed, whose proposals would help, and how to organize
execution as workflows.

Observe how much of that reasoning and organization the agents perform
themselves and which Omni interventions help. Do not equate a successful
assisted result with autonomous coordination.

This document plans the experiment. Creating it does not start the trial
or send messages to agents.

## Starting point and scope

Use [p3's final report](../report.md) as evidence, not as a recipe to give
the participants:

- The unassisted trial reached a result review in one shared discussion.
- It chose a small solution compatible with the deliberately loose goal.
  That choice is not itself a defect.
- Work happened inside discussion turns, without project-workflow
  delegation, so the new awaited-result path was not exercised by the
  actual experiment.
- Two RTS turns delivered speech without completing their run records.
- The review corrected claims, but no separate synthesis or knowledge
  document was produced and implementation results were not integrated.

Keep the same RTS/Jev subject so the change in interaction can be examined.
The exact task goal comes from [the parent braindump](../braindump.md):
one AI-versus-AI RTS experiment with some Jev involvement and no demanding
quality requirement. ex1 adds process guidance; it does not silently require
TypeSafe's service, a particular model, in-play inference, a benchmark suite
or a higher-quality game.

Start a new discussion, preserving the earlier one. Existing project
knowledge and p3 artifacts remain available through ordinary discovery.
Record whether participants use them; this is not a clean-room comparison.

Retain the parent's single discussion → execution → report review cycle
as the initial scope. ex1 replaces the no-intervention condition with active
Omni guidance. Review examines existing reports and artifacts; it does not
launch another experiment or a product-improvement cycle.

## Working conventions

- Follow `localrule.md`: generated files are English and communication with
  the Developer is Japanese. Development, checks, deployment and appropriate
  commits/pushes do not require additional human approval checkpoints.
- Before deployment inspection or changes, obtain current service state
  through Nautobot/nctl, following `pj-clusterintent/nctl/README.md`.
  Previous reports do not establish current availability.
- Prepare direct LAN-accessible links and perform verification the agent
  can perform itself. Keep machine-specific details, credentials and raw
  operational evidence in ignored files.
- Preserve unrelated edits and active work. Backward compatibility is not
  required, but active delivery must remain accounted for.
- Reuse the ordinary discussion, delegation, wait, instruction and recording
  tools. Keep shared tool facts in their owning help/guides.
- A decision to proceed is the agents' working judgment, not an approval
  request to the Developer. Do not turn this experiment's observed stages
  into mandatory global argue gates.

## Participants and responsibility

Verify the current workspace identities before launch:

| Participant | Responsibility in this experiment |
| --- | --- |
| Omni Agent, explicitly acting for the Developer | State the goal and process expectation; offer adaptive instructions and hints; record assistance and evaluate the outcome |
| `jevlab` | Bring relevant existing knowledge with sources and limitations; propose and execute its own specialist work |
| `rts-vs-bot` | Explain the actual game, control interfaces and constraints; propose and execute its own specialist work |
| `front` | Judge information sufficiency, decide further inquiries or proposal requests, organize work as workflows and account for results |
| `argue` | Facilitate the shared conversation and preserve its continuity, attribution and records |

Front is responsible for substantive coordination in this trial. The
existing argue facilitator still owns invitation mechanics where the system
requires it. Front can state whom it wants to hear from and why, and the
facilitator can issue that invitation on Front's behalf with provenance.
Do not silently let the facilitator make every decision while crediting
Front, or let Omni become the actual workflow planner.

Inspect whether this division can operate through current tools. Provide
missing transport/capability support if necessary, but do not build a new
general orchestration system solely to enforce the expected pattern.

## Guidance and attribution

The opening message contains:

- The Developer's task goal, quoted exactly from its source.
- The ex1 expectation: relevant knowledge and game facts should inform
  Front's decisions about discussion and workflow execution.
- The trial-local protocol: Omni may guide, participants retain technical
  choices, and the trial ends after one execution-and-review cycle.

Label quotes from the Developer separately from Omni's interpretation.
Post subsequent hints as **Omni Agent (for Developer)**, not as new
Developer-authored requirements.

Give the agents a reasonable opportunity to act after each instruction.
Observe active runs, pending invitations and waits before interpreting
silence. Avoid issuing redundant hints while a response is already queued.

Choose assistance based on the actual conversation. Examples of useful
prompts are asking which existing source supports a proposal, what the
game actually permits, what Front still needs to decide, or where the
work and its result will be recorded. These are possibilities, not a
script or mandatory sequence.

Begin with questions or concise process hints when sufficient. If those
do not help, a direct instruction is allowed; record that greater degree
of assistance. Do not repeatedly rephrase an ineffective hint indefinitely.
If further progress would require Omni to provide the technical solution,
task graph or implementation, stop and report the observed limit.

Do not preselect the Jev mechanism, invent the game interface, supply
ready-made parameter values or perform specialist implementation for the
participants. If an accidental intervention supplies substantive solution
content, record it and narrow any claim about independent agent reasoning.

For every intervention record:

- The message ID, time and recipient.
- The observed issue and available evidence.
- Whether it was a question, process hint, direct instruction, technical
  assistance or system repair.
- The response and any subsequent behavioral change.
- Which decisions and artifacts remained agent-produced.

## Step 1 — prepare the trial and verify readiness

1. Read current discussion contracts, project guides, participant
   registration and the p3 reports. Inspect nctl/WFE state and outstanding
   work without modifying unrelated runs.
2. Confirm all participants can speak, read each other's original messages,
   access their project knowledge and use ordinary work workflows.
3. Establish how Front's coordination decisions reach the facilitator's
   invitation mechanism and how results return to the same discussion.
   Check the existing `discussion await` and delegation path before adding
   anything.
4. Fix demonstrated capability or permission gaps needed for the experiment.
   Keep technical support separate from coaching. Record revisions and
   verify changed behavior with a trivial unrelated fixture where needed.
5. Avoid task-specific rehearsals or global guide tuning to prearrange the
   desired RTS/Jev answer. Record any generic guide changes as part of the
   experimental conditions.
6. Set up passive observation and an intervention ledger. Account for
   automatic recovery agents as another possible source of assistance.

Deliverable: `report1.md` with current readiness, changes, the coordination
boundary, deployed revisions and any focused verification.

## Step 2 — build shared understanding through guided discussion

1. Open one new discussion with the task and trial protocol, recording
   source revisions and explicit authorship.
2. Ask jevlab and RTS to contribute the existing knowledge and concrete game
   facts relevant to the goal. Let them select their sources and level of
   detail; do not require an exhaustive survey.
3. Ask Front to use those contributions to judge whether to proceed, ask
   further questions or request proposals from particular participants.
   Let Front explain its reasoning and choose the route.
4. Observe whether participants respond to each other's evidence, expose
   assumptions and revise proposals. Give hints only where the actual
   exchange warrants them.
5. When Front judges a direction sufficient, let it formulate the work,
   ownership, dependencies and expected result evidence using WFE. Omni
   checks whether coordination is happening rather than writing the graph.

Deliverable: `report2.md` with the discussion's decisions, cited knowledge
and game facts, rejected or revised options and the guidance that mattered.

## Step 3 — follow structured execution and return

1. Let Front and the project agents commission ordinary workflow work,
   rather than carrying the implementation inside discussion turns.
   Remind them of this experiment's structural objective if needed, and
   record the reminder.
2. Link commissioned runs and dependencies to the discussion. Verify that
   wait ownership is explicit, waits release slots, and the responsible
   agent receives questions and results.
3. Let agents choose implementation and verification details. Observe
   actual cross-project handoffs, source revisions and delivered artifacts.
4. Use adaptive hints if coordination goes wrong. Repair demonstrated
   system defects separately; record changes to capabilities during the
   trial. Do not confuse a project needing to develop its solution with a
   system defect for Omni to repair.
5. Track results until they return to the same discussion, without treating
   a dispatch acknowledgment or early facilitator completion as success.
   Observe correct run completion as well as message delivery.

Deliverable: `report3.md` with the work graph, execution/results, callback
evidence, interventions and any point where assisted coordination stopped.

## Step 4 — guide report examination and conclude

1. Have the participants examine the returned reports in the original
   discussion. Front should account for the original goal, each work item
   and the evidence, while specialists state their limits.
2. Guide attention to unsupported claims or missing provenance when needed.
   Review may read existing files and clarify reports; it does not rerun
   the match, add experiments or repair the product.
3. Separate what the system actually did from claims about Jev's benefit or
   game quality. Report uncertainty without forcing a favorable conclusion.
4. Ask the responsible agents to preserve a source-linked summary and
   explain the disposition of their artifacts: run-only, integrated, or
   intentionally left unintegrated. Do not integrate blindly merely to
   improve the experiment's completion statistics.
5. End after review. Also end with an honest limitation if sustained hints
   fail to produce progress or if proceeding would require Omni to supply
   the solution. Distinguish that from the parent's unassisted stall.
6. Account for outstanding trial work, close the discussion and verify
   archival publication. Record any operational cleanup performed by Omni.

Deliverable: `report4.md` with review, artifact disposition, archival links,
final work state and the reason for termination.

## Evaluation and final report

Evaluate observed behavior and assistance together:

| Question | Evidence to retain |
| --- | --- |
| Did existing knowledge inform the discussion? | Source-linked contributions and later decisions that used, challenged or rejected them |
| Did game facts constrain the proposal? | Actual interfaces/constraints cited by RTS and their effect on the chosen approach |
| Did Front coordinate? | Front's decisions about questions, proposal requests, work ownership and workflow structure, distinguished from argue/Omni decisions |
| Was work structurally separated from discussion? | Ordinary project runs, dependencies, waits, result callbacks and run completion records |
| Did the same discussion review the results? | Returned reports, questions, corrections, unknowns and final synthesis |
| How much help was required? | Intervention ledger, responses, ineffective hints and direct instructions |
| What persisted? | Archive horizon, summary, artifact revisions and integration or run-only disposition |

Write `report.md` with the task, protocol, participants, revisions, timeline,
findings, assistance and review links. Compare with p3 descriptively:
ex1 changes the instructions and includes active coaching, and agents may
reuse previous artifacts. It cannot establish a controlled causal result
or demonstrate autonomous behavior merely because the outcome improved.

Keep future improvement ideas separate from this trial's work. Update
affected documentation and ignored environment notes, complete checks
appropriate to actual changes, and commit/push intended changes and pins
without disturbing unrelated work.

## Completion criteria

- One new trial used the RTS/Jev goal with clearly attributed process
  guidance, without pretending that Omni hints came verbatim from the
  Developer.
- Specialists had the opportunity to supply relevant existing knowledge
  and game facts; Front's subsequent judgments and actual role are recorded.
- Structured workflow execution and return to review were attempted, with
  the achieved path or observed limit supported by evidence.
- All coaching, system repairs and automatic recovery were distinguished
  from independent participant behavior.
- The trial ended after one execution/review cycle or an explicitly
  documented limit, with outstanding work accounted for.
- Records and review links make both the result and the required assistance
  inspectable. Success does not require a strong game result or proving that
  guided coordination worked.
