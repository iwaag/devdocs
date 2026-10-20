# Argue p4 — ponderer and discussions as workflows

## Objective

Implement [braindump.md](braindump.md): establish `ponderer` as a managed
agent project that conducts discussions through explicit workflow activities,
while Front concentrates on the Developer's intent, project participation
and the work entrusted to those projects.

Make discussion progress inspectable as `announce`, `ask`, `ponder` and
`talk`, with durable connections to speech, questions, decisions and work.
The agent chooses how the discussion proceeds; the system records and
executes those choices without imposing a fixed debate procedure.

Preserve discussion, execution, reports and later analysis so successive
experiments can be compared as models, contexts and coordination practices
change.

This document plans implementation. Creating it does not start development,
deployment or agent trials.

## Evidence and working conventions

Read [p3](../p3/report.md) and [p3 ex1](../p3/ex1/report.md) as the initial
evidence:

- Project-specific knowledge and game facts informed useful agent decisions.
- Front constructed and executed a delegated work graph when its role was
  made explicit.
- Review and preservation needed external reminders in ex1.
- Delivered speech and completed workflow state diverged in several runs.
- A question to the person left an orphaned facilitator wait.
- Incorrect claims about integration survived review and entered a summary.

Follow `localrule.md`:

- Generated files are English; communication with the Developer is Japanese.
- Implementation, testing, deployment and appropriate commits/pushes proceed
  without additional human approval checkpoints. Questions about intent are
  not permission gates.
- Deliver a deployed result accessible from other home-LAN machines, with
  direct review links and minimal setup.
- Obtain current service state through Nautobot/nctl before deployment
  inspection or changes, following `pj-clusterintent/nctl/README.md`.
  Environment notes and episode reports are not live-state evidence.
- Keep machine-specific details, credentials and raw operational evidence
  in ignored files. Preserve unrelated edits and active work.
- Backward compatibility is unnecessary. Account for active deliveries
  during cutover; do not delete historical records to simplify migration.
- Give agents discoverable tools and concise responsibility descriptions.
  Add guidance grounded in named trials rather than hypothetical concerns.
- Record Omni work on behalf of project agents with the handoff note
  required by `devdocs/README_DEV.md`.

## Ownership

| Owner | Responsibility |
| --- | --- |
| Front | Understand the person's purpose, judge alignment, choose participating projects, commission and evaluate specialist work |
| ponderer | Identify current issues, choose questions and speakers within the agreed participation scope, judge information sufficiency, structure and conduct the discussion |
| Participating projects | Explain their domain, propose approaches, execute their specialist work and report evidence and limitations |
| argue | Own discussion archives, source-linked summaries and cross-discussion knowledge |
| WFE | Identity, workflow state, message and node correlation, dispatch, waiting, continuation, recovery and authoritative execution evidence |
| aghangout | Present the conversation, its progress and links to workflow and records |

Ponderer can identify a missing perspective and ask Front to select an
additional project. Front does not need to approve every invitation or
workflow adjustment. Ponderer decides the precise question and its place
in the discussion; specialists decide their technical approach.

Move active facilitation from `argue/facilitate` to ponderer. Do not retain
two competing progress owners. Keep argue's archive and synthesis functions,
and preserve old runs under their original identities.

Create `ponderer` / `pj-ponderer` through the ordinary managed-project path.
Keep its guides and reusable facilitation knowledge there. Discussion
content and cross-project findings remain in `pj-argue`; avoid duplicate
authoritative copies of the same findings.

## Discussion activity contract

The four activities must be recognizable in definitions, run views and
history. Their executable meaning matters more than whether every name
becomes a new primitive node type.

| Activity | Meaning | Completion evidence |
| --- | --- | --- |
| `announce` | Publish an agenda, context, interim synthesis or next-step explanation to the discussion | Recorded message ID and provenance; no implied response wait |
| `ask` | Invite named participants to address a question tied to source messages | Corresponding replies or explicit unavailable, unanswered, withdrawn or failed outcomes; dispatch alone is not completion |
| `ponder` | Read the current discussion and work state, choose the next action and revise the graph when useful | Concise decision, source references, selected action and graph revision or terminal outcome |
| `talk` | Obtain intent or direction from the human or requesting agent | A correlated answer, explicit withdrawal or unresolved outcome with a defined continuation owner |

For `ponder`, retain a short decision explanation sufficient for external
review, not a transcript of internal reasoning. For example: which facts
are settled, which missing fact matters, and why the next question goes to
a particular participant.

These activities do not replace `delegate`, `do` or `study`. Implementation
and substantial specialist work use ordinary project workflows. Ponderer
can continue the discussion around that work without absorbing it into
conversation turns.

### Representation

Inspect the existing user node catalog and run-adjustment machinery first.
`announce` and `ponder` may be templates over `do` if their behavior and
evidence are expressible there. `ask` needs durable question/result
semantics; a label alone is insufficient. `talk` needs both human and
requesting-agent reply paths.

Choose templates, supported node metadata or new primitive types according
to the execution semantics actually required. Document the choice and
update validation, compilation, CLI, API, executor and UI consistently.
Do not build a general plugin/custom-node platform as a prerequisite.

### Adaptive graph

Start with a small workflow. Ponderer may add announcements, questions,
consultations or a conclusion as evidence develops. No fixed rounds,
mandatory consensus or prescribed order of specialists is required.

Current run graphs are acyclic. Represent another round with new nodes
and a recorded revision, retaining prior questions and outcomes. Do not
overwrite a completed node to conceal repetition or introduce graph cycles
merely to model conversational iteration.

A graph dependency is not an approval. Reuse the current autonomy and
revision contracts; graph structure explains the chosen work.

## Speech, waiting and continuation

- Keep the discussion ID independent of workflow runs. A discussion may
  refer to several runs, but every pending question or commissioned result
  has one explicit owner responsible for consuming it.
- Correlate messages and invitations with the originating run, node,
  attempt, recipient, question and source-message horizon.
- Make message submission and node recording recoverable across retries
  and crashes. Replaying an operation must neither post again nor leave a
  delivered announcement permanently running.
- Waiting frees execution capacity. Distinguish waiting for an invited
  contribution, a specialist result and an answer from the requester.
- Route answers to the outstanding `talk` question and its owning work.
  A human answer must not silently create unrelated work while the original
  question remains orphaned. A genuinely new request may start new work.
- Record partial replies, late answers and failed participants explicitly.
  Give ponderer the facts to decide whether to proceed, ask elsewhere or
  consult Front; do not hardcode unanimity as the condition for progress.
- Verify races between contributions, attempt exit and reconciliation.
  Completion, cancellation and restart must not duplicate a continuation.
- Ending the discussion and ending commissioned work are separate acts.
  The final state must account for unresolved questions and outstanding work,
  including who will receive any later result.
- Expose current obligations, such as an agreed review or requested summary,
  through recorded intent and work state. Ponderer should be able to inspect
  what remains without requiring every discussion to follow the same stages.

## Durable records and analysis

Reuse WFE's live store and argue's archive writer. Extend their existing
records where necessary rather than introducing another authoritative
conversation store.

Retain or link:

- Original purpose and source instructions, participants and responsibilities.
- Actual model/profile and relevant guide/context revisions or fingerprints
  already captured by runs; distinguish requested from actual settings.
- Discussion messages, invitations, work graph revisions, node outcomes
  and concise decisions with source references.
- Commissioned work, dependencies, returned reports and artifact revisions.
- Human/Omni interventions and automatic recovery, with their observed effect.
- Source-linked synthesis, unresolved claims and subsequent analysis.

Keep original speech, agent synthesis and external evaluation distinct.
A later correction adds provenance and preserves the earlier statement.
Do not treat several agents repeating one claim as independent evidence.

Provide a clear way to read published project state and integration
evidence. Local `main`, remote published revision and `run.integrate`
records are different observations. A disposition such as "not integrated"
must name the evidence and observation time it rests on.

Use a small episode index in the existing archive/knowledge structure to
find comparable discussions and their analyses. Reuse existing storage;
avoid a new analytics service. Mark unavailable historical context as
unknown rather than reconstructing it as if recorded.

Analysis may compare question selection, repeated discussion, waits,
interventions, completion and evidence quality across runs. It must state
changes in task, model, context and guidance. A few trials do not establish
that one model or workflow is universally superior.

## Step 1 — inspect and settle contracts

1. Inspect current discussion, delegation, catalog, graph-adjustment,
   session and continuation contracts and their implementations.
2. Map each activity to existing semantics and identify concrete gaps.
   Choose its representation and the smallest necessary implementation.
3. Define Front/ponderer/argue responsibilities and message routing.
   Include a requester that is Front and one that is a human.
4. Write event traces for announce, parallel asks, requester clarification,
   delegated work, result review, closing and late replies.
5. Inspect live discussions and waits before choosing the cutover. Specify
   how each outstanding obligation retains an owner.
6. Specify record extensions, published-state reads and the analysis index.

Deliverable: `report1.md` with ownership, activity contracts, event traces,
representation choices, affected components and migration/recovery approach.

## Step 2 — implement workflow activities and reliable continuation

1. Implement the selected node/template representation and documented tools.
2. Add explicit message/question/result-to-node correlation and recoverable
   recording where existing primitives are insufficient.
3. Implement human and requesting-agent `talk` routing, ask outcomes and
   adaptive graph revisions without fixed debate procedures.
4. Expose progress and obligations through normal run/discussion reads.
5. Provide published-revision and integration evidence needed for accurate
   artifact disposition.
6. Verify key transitions, replay, restart, concurrent/late replies,
   cancellation and attempt-end races with focused tests. Cover the orphaned
   wait and delivered-speech/unfinished-node cases observed in p3/ex1.

Deliverable: `report2.md`, updated contracts/help and meaningful tests for
the new execution behavior.

## Step 3 — establish ponderer and update project responsibilities

1. Create the ponderer managed project with intent, guides and a small
   adaptable discussion workflow.
2. Give it tools to inspect source messages, context, outstanding work,
   participant outcomes and current graph revisions.
3. Move active facilitation responsibility from argue to ponderer. Keep
   archive/synthesis ownership explicit and document how ponderer requests
   those services and receives their results.
4. Update Front's guide for intent alignment, participant selection and
   specialist work. Remove unnecessary requests for a facilitator's "go".
5. Keep participant guides focused on their domain and the shared reply
   contract. Ensure substantial work still uses ordinary project workflows.
6. Verify a trivial discussion with additional questions and a requester
   clarification, without rehearsing the substantive experiment.

Deliverable: `report3.md` with project changes, guides and a trace proving
the new division of responsibility.

## Step 4 — present progress and preserve comparable records

1. Extend aghangout and existing WFE views to show the current activity,
   questions and intended respondents, waits, decisions and linked work.
2. Keep ordinary conversation readable. Detailed graph and execution
   information can be linked rather than forced into every speech view.
3. Connect nodes to the messages that fulfill them and expose unresolved
   obligations or failures without ambiguous "done" labels.
4. Extend the existing archive and source-linked summary path for the new
   provenance. Verify retry-safe publication and a fresh checkout.
5. Add an episode index and a lightweight analysis format. Reference p3 and
   ex1 as historical evidence, preserving their recorded limitations.
6. Verify browser behavior, long discussions and direct links to messages,
   workflows, published artifacts and analyses.

Deliverable: `report4.md`, browser checks and inspectable archive/index
examples.

## Step 5 — deploy and exercise the complete path

1. Obtain fresh Nautobot/nctl state; back up affected stores/configuration
   and record a coherent code-and-data recovery path in ignored notes.
2. Deploy WFE, project guides/workflows and aghangout through their ordinary
   release mechanisms. Transfer active facilitation without losing or
   replaying outstanding deliveries.
3. Update service records if placement or deployment ownership changes.
   Verify current client behavior and review links from another LAN machine.
4. Run bounded fixtures for:
   - announcements with no implied wait;
   - multiple asks, partial outcomes and an unavailable participant;
   - a human answer and a requesting-agent answer resuming their own work;
   - a specialist result arriving after an attempt or run has ended;
   - restart and duplicate delivery;
   - a conclusion with explicitly accounted-for outstanding work.
5. Fix demonstrated system defects and rerun affected checks before the
   substantive trial.

Deliverable: `report5.md` with deployed revisions, fixtures, recovery
evidence and LAN verification.

## Step 6 — observe autonomous progression and analyze

Use a bounded substantive discussion comparable to p3/ex1: project-specific
knowledge and facts, decisions, ordinary specialist work and result review.
Record the chosen brief, participant roles, reuse of earlier artifacts and
all other trial conditions before launch.

Give the agents the goal and responsibility definitions at the start.
Do not supply intermediate progression hints. The trial should reveal
whether ponderer selects useful next steps and follows the agreed work
through review and preservation without Omni supplying those transitions.
One execution-and-review cycle is sufficient for this trial; this is not
a global limit on ponderer.

Observe:

| Question | Evidence |
| --- | --- |
| Did ponderer own progression? | Source-linked decisions and graph revisions, distinguished from Front's decisions |
| Did Front retain purpose and participation responsibility? | Alignment judgments, participant choices and work requests |
| Were questions useful? | Missing information identified, replies consumed and resulting changes in direction |
| Did waiting and completion agree with speech? | Node records, message IDs, callback ownership and absence or explanation of leftovers |
| Did execution return to review without a hint? | Delegations, awaited results and the subsequent activities |
| Was the final account accurate? | Published revisions and integration evidence compared with stated dispositions |
| Can the episode be studied later? | Archive, source-linked summary, intervention record and indexed analysis |

A coordination stall or unnecessary loop is a valid observation. Do not
rescue it with an unrecorded hint. Repair genuine system defects with an
intervention record; keep agent behavior and technical faults separate.
Account for trial-owned work when stopping and leave unrelated work alone.

Write `report6.md` with the trial and analysis, and `report.md` with the
delivered behavior, evidence, limitations and direct review links. Compare
with p3/ex1 descriptively, without attributing every difference to ponderer.
Preserve unfavorable observations as well as successful examples.

Update developer documentation, project guides, CLI help and ignored
environment notes. Complete checks appropriate to affected components,
commit/push intended changes and update parent pins where applicable.

## Completion criteria

- Ponderer is an ordinary managed project and the active discussion-progress
  owner; Front and argue have clear, non-duplicated responsibilities.
- `announce`, `ask`, `ponder` and `talk` have visible, documented semantics
  and completion evidence, implemented without unnecessary generic machinery.
- Discussion structure can evolve through agent-selected graph revisions,
  retaining questions, decisions and their sources.
- Human/requester answers and specialist results have durable continuation
  owners; restart/replay do not duplicate speech or strand answered work.
- Ordinary project execution remains separate from discussion turns, with
  inspectable handoffs and results.
- Artifact disposition can be grounded in published state rather than a
  stale local branch.
- Archives, summaries and analysis retain enough provenance to compare
  future episodes while keeping claims and evidence distinct.
- The result is deployed and verified over the LAN with direct review links.
- A substantive trial records how far progression works without hints,
  including any unresolved limitations. A favorable trial outcome is not
  required to report the experiment honestly.
