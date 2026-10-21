# Argue p4 ex1 — ponderer owns discussion practice and learning

## Objective

Implement [braindump.md](braindump.md) and the agreed direction: make
ponderer responsible for conducting discussions, preserving their records,
evaluating their outcomes and improving its practice. Organize external
research in `study-argue`, reusable discussion knowledge in `wedo-argue`,
and project-specific records and workflows in `pj-ponderer`.

Retire argue as an active project agent and dependency without leaving
compatibility aliases, duplicate knowledge authorities or dual archive
writers. Preserve original records, authorship and citation provenance.

This plan covers implementation, migration, deployment and verification.
Writing this plan does not itself start those operations.

## Evidence and conventions

The inspection preceding this plan found:

- Published ponderer `eb87712` holds conductor workflows, run records and
  three findings in `knowledge/`. Its local checkout lagged published main;
  inspect published revisions when planning migration.
- Published argue `4fd89fd` holds discussion archives, summaries, episode
  analyses and subject-matter knowledge. Its guides distinguish discussion
  practice from the subjects discussed.
- Ponderer's `knowledge/owed-work.md` still says its practice is to be
  checked against p4. Argue's `episodes/argue-p4.md` already records the
  omitted review and stale final account. Evaluation and revision of
  practice therefore have separate owners today.
- The configured facilitator is `ponderer/conduct`; `argue/synthesize` is
  a participant and argue is the archive destination.
- `study-argue` and `wedo-argue` were not visible through the configured
  Gitea principal and were not found in the inspected registration. Check
  ownership and existing repositories before creating them.
- Three discussions were open, with no open invitations in the list view.
  This is not proof that all associated work has ended.
- WFE's archive writer resolves a configured workspace, but its publication
  records retain commit IDs and horizons without a destination identity.
  Changing the workspace setting alone can leave misleading publication
  status and links.

These are planning observations, not a live cutover inventory. Follow
`devpolicy/styles.md`, `devpolicy/terms.md`, `devdocs/README_DEV.md`,
`pj-agdev/.local/devenv.md` and `localrule.md` during execution:

- Generated files are English; communication with the Developer is Japanese.
- Execute authorized implementation, deployment and appropriate commits and
  pushes without approval checkpoints. Clarify genuinely missing intent;
  do not turn review, acceptance or integration records into permissions.
- Backward compatibility is unnecessary. Preserve evidence and ongoing work
  without maintaining obsolete operational paths.
- Inspect local services through Nautobot/nctl before service work, using
  `pj-clusterintent/nctl/README.md`. Report observation ages and refresh
  relevant observations when needed.
- Keep machine addresses, credentials, backups and raw operational evidence
  in ignored local files. Preserve unrelated edits.
- Finish with a deployed result reachable from other LAN machines and
  direct review links requiring little setup.
- Give agents discoverable tools and concise responsibility descriptions.
  Ground guidance changes in named trials, rather than hypothetical failures.
- When Omni performs an in-system agent's work, leave the required one-line
  episode note: "did X for agent Y — handoff candidate".

## Ownership and destinations

| Owner / destination | Responsibility |
| --- | --- |
| ponderer / `pj-ponderer` | Discussion progression, workflows and guides, individual discussion records and summaries, experiment conditions, evaluations and the decision to revise practice |
| `study-argue` | Literature research: questions, sources, methods, findings and limits; distinguish external evidence from local observations |
| `wedo-argue` | Reusable knowledge about discussion design, facilitation and evaluation, with evidence, applicability, revisions and unresolved questions |
| Participating projects | Their subject knowledge, implementation, adopted decisions and domain-specific knowledge maintenance |
| Front | The Developer's purpose, alignment and choice of participating projects |
| WFE | Authoritative live conversation and execution state, identities, delivery, continuation and archive publication |
| aghangout | Presentation of discussion, progress and records |

Ponderer owns the learning agenda and adoption of findings. It may commission
literature research from archsage through existing study workflows. Declare
repository maintenance and write ownership using the current knowledge
contracts; do not introduce a second study execution system.

Bind the study and wedo repositories through the normal managed-project
submodule mechanism. Record the revision actually used by a run. A study
writer publishing a change does not silently update ponderer's pin; adoption
and the corresponding project revision must be visible.

Within `pj-ponderer`, retain a clear division:

- `agent/` and `devdocs/workflows/`: responsibilities and executable work.
- `devdocs/runs/`: the existing versioned run records.
- `discussions/<id>/`: archive manifests and transcripts, plus separately
  authored summaries. WFE remains the sole writer of archive files.
- `episodes/`: individual evaluations, conditions and comparison index.
- Ignored runtime storage: existing WFE stores, transient execution data and
  large/raw evidence as appropriate. Do not duplicate WFE's live database
  inside the project or commit secrets and machine-specific runtime state.

An individual summary or evaluation is evidence, not automatically a general
finding. Promote knowledge deliberately and keep contrary evidence visible.
Research, evaluation and promotion are available work, not mandatory stages
of every discussion. Agreed reviews and summaries remain obligations until
fulfilled or explicitly dropped.

Subject-matter findings do not all belong in `wedo-argue`. For example,
argue's asset-provenance finding belongs with the relevant domain owners;
ponderer retains the discussion and its conclusion. Distinguish preserving
a conclusion from commissioning implementation: an earlier request for no
handoff must not become a new implementation request during migration.

## Step 1 — inventory and settle the migration map

1. Refresh published repository references without overwriting working
   changes. Inspect applicable project instructions and existing knowledge
   repository conventions.
2. Inventory argue and ponderer files, branches, run records, tools, guides,
   workflows, registrations, study bindings and knowledge maintenance rules.
3. Classify every retained artifact as original record, individual summary,
   evaluation, external research, reusable discussion knowledge or domain
   knowledge. Assign one current owner and destination.
4. Trace active references: Front and participant guides, archsage's index,
   WFE configuration and CLI help, archive status and URLs, aghangout links,
   schedules, outstanding requests and execution callbacks.
5. Inspect open discussions, conductor runs, invitations, delegations,
   questions and pending archive writes. Record how each outstanding result
   retains a receiver through cutover. Do not close discussions merely to
   simplify migration.
6. Select a history-preserving import strategy. Retain access to referenced
   commits and paths in the new repository, or explicitly map citations in
   derived documents. Never rewrite verbatim speech or historical run
   identities to pretend ponderer performed argue's earlier work.

Deliverable: `report1.md` with the artifact map, affected components,
outstanding-work inventory, citation strategy and coherent recovery plan.

## Step 2 — establish ponderer's knowledge and workflow structure

1. Establish or reuse `study-argue` and `wedo-argue` after checking their
   existence and ownership. Add maintenance declarations and submodule
   bindings through normal managed-project mechanisms.
2. Move ponderer's reusable facilitation findings to `wedo-argue`, preserving
   named trial sources and revision history. Remove the competing current
   copies in `knowledge/` and update guides to the new locations.
3. Transfer argue's summaries, episode analyses, archive-reading tools and
   synthesis capability to ponderer. Separate workflows such as `conduct`,
   `synthesize` and evaluation may serve different work without requiring
   another agent identity. Choose the smallest useful workflow set.
4. Make workflow selection and continuation work for synthesis requested
   from an active conductor. Verify whether existing participant routing
   assumes one workflow per workspace; use ordinary delegation where it
   already provides the needed semantics. Do not retain argue as a routing
   alias or let a synthesis request accidentally start another conductor.
5. Give ponderer discoverable access to its archives, evaluations, study and
   wedo. Update Front, participants and archsage's index to the new owners.
6. Transfer domain knowledge to its appropriate maintainers, keeping source
   links and the distinction between adopted conclusions and proposals.
   Record a specific owner for cross-project findings rather than copying
   the same authoritative text into multiple repositories.

Deliverable: `report2.md` with repository structure, responsibility changes,
workflow routes and knowledge ownership.

## Step 3 — migrate archives and remove destination assumptions

1. Import historical archives, summaries, evaluations and relevant Git
   history into ponderer according to the migration map. Preserve message
   IDs, authors, timestamps, run references, annotations and archive horizons.
2. Update archive publication to handle the destination change explicitly.
   Inspect stored publication state, dedicated writer clone, origin, locks,
   retry state and cached URLs. Prevent old commit/horizon records from
   reporting the new destination as current before it has been verified.
3. Keep one writer. Coordinate final catch-up and cutover so concurrent
   publications and run integrations cannot lose or duplicate records.
   Rebuild disposable writer state from published truth where appropriate.
4. Update current citations and links to resolve through the new destination.
   Preserve literal historical citations as evidence, with an explicit
   provenance map where needed; this is documentation, not a runtime alias.
5. Remove argue-specific assumptions from current code, CLI help, contracts
   and UI text. Reuse the configurable archive mechanism rather than adding
   a parallel publisher or archive service.
6. Add focused tests for destination switching, closed and open archives,
   publication interruption, replay, concurrent integration and citation
   resolution. Verify migrated manifests against their transcripts and
   ensure unchanged records remain unchanged.

Deliverable: `report3.md` with migration evidence, publication behavior,
citation checks and focused test results.

## Step 4 — deploy, cut over and retire argue

1. Obtain current Nautobot/nctl state. Back up affected repositories, stores
   and configuration in ignored storage and record mutually compatible code
   and data recovery steps. Backup retention is not a compatibility path.
2. Publish project changes and deploy affected WFE/UI components through
   their existing release mechanisms. Switch the archive destination and
   current synthesis/evaluation routes to ponderer.
3. Reconcile every migrated archive and check published commits, horizons,
   contents and browser links. Verify an existing open discussion can
   continue, and pending results still reach their owning work.
4. Remove argue from active participant, workflow, schedule and project
   discovery routes after its outstanding work is accounted for. Preserve
   historical run references using supported historical records, not an
   executable compatibility project. Extend retirement handling only if a
   concrete history-resolution gap requires it.
5. Once history and citations are independently accessible at their new
   destinations, retire the old repository from operational use and remove
   obsolete working copies and configuration. Do not keep a second live
   authority, redirect shim or forwarding workflow. Do not erase evidence
   merely to achieve a clean reference search.
6. Update service records if deployment ownership or placement changes.
   Update current developer documentation and ignored environment notes;
   preserve historical episode reports as historical evidence.
7. Verify from another LAN machine that discussion progress, archives,
   summaries and knowledge links work. Supply direct review URLs.

Deliverable: `report4.md` with deployed revisions, retired paths, live
continuation checks, LAN evidence and recovery notes.

## Step 5 — prove the learning cycle

Use the p4 omitted review and stale final account as a bounded case. Moving
files alone does not demonstrate that ponderer can improve its practice.

1. Give ponderer a request to evaluate the p4 evidence and decide whether
   existing discussion knowledge should change. Record the brief, contexts,
   model/profile, repository pins and any Omni assistance.
2. Have its ordinary workflow inspect the archive, run records and existing
   analysis; preserve the distinction between what the agent said and what
   execution evidence showed. A single trial does not establish causation.
3. Exercise one bounded literature-research request through the established
   study mechanism. Preserve actual sources and their limits in
   `study-argue`; if the research cannot be completed, report that limitation
   rather than presenting repository setup as completed research.
4. Let ponderer revise or qualify `wedo-argue` from the local and external
   evidence, retaining provenance and contrary observations. Correct the
   obsolete "to be checked against p4" status. Integrate the change and
   explicitly adopt its revision in ponderer.
5. Run a small subsequent discussion with an agreed result review and
   summary. Confirm that its context includes the adopted knowledge revision
   and that ponderer can access and use it. Observe whether review and
   preservation occur without intermediate progression hints.
6. Check that summaries, evaluation and any reusable finding reach their
   intended destinations, with no active dependency on argue. Record an
   omitted review or failed adoption honestly; do not force a favorable
   behavioral result by hardcoding a universal discussion sequence.

Deliverable: `report5.md` with the research, evaluation, knowledge revision,
adoption and subsequent discussion trace. Record system defects separately
from agent judgment and trial assistance.

## Completion criteria

- Ponderer owns discussion operation, preservation, evaluation and learning;
  Front and participants retain their existing domain responsibilities.
- `study-argue` contains sourced external research; `wedo-argue` contains
  reusable discussion knowledge; individual records and domain findings
  have explicit, appropriate owners.
- Historical speech, authorship, run identities and citations remain
  inspectable. Open work retains its receivers through migration.
- WFE remains the live-state authority and sole archive publisher. Published
  status and links refer to verified content in the new destination.
- No active argue agent, compatibility alias, duplicate writer or old
  repository dependency remains. Historical names and provenance are not
  mistaken for obsolete runtime dependencies.
- A demonstrated evaluation → research → knowledge revision → explicit
  adoption → subsequent discussion path is inspectable. Behavioral
  limitations are reported, not concealed.
- Relevant tests and LAN checks pass; deployed review links require little
  setup. Intended changes are committed/pushed and parent pins updated where
  applicable.
- `report.md` summarizes the delivered structure, evidence, limitations and
  review links. Include required Omni handoff notes and updated current
  developer documentation.
