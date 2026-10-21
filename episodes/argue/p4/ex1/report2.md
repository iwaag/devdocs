# argue p4 ex1 — Step 2: ponderer's knowledge and workflow structure

## Repositories

| Repository | Revision | Content |
| --- | --- | --- |
| developer/wedo-argue (new, Gitea, registered `wedo`) | `56cd52a` | `knowledge.yaml` (`maintenance: project`, maintainer `ponderer`); README (layout, what belongs here); `index.md`; `findings/{asking-the-requester,owed-work,artifact-claims}.md`, moved unchanged from `ponderer@a1fdab5:knowledge/`, each with a *Revisions* line saying so |
| developer/study-argue (new, registered `study`) | `f11bc1d` | `knowledge.yaml` (maintainer `ponderer`; written by archsage `study` runs that ponderer commissions); README (layout `topics/<topic>.md`, rules: every claim sourced, external evidence apart from local observation, limits are results). No topic yet |
| developer/pj-ponderer | `2b0be53` | see below |
| developer/pj-archsage | `92ad65a` | study destination and discussions domain |
| developer/pj-forge | `c4091ee` | `knowledge/asset-provenance.md` |

Both new repositories were checked for existing names and owners
beforehand (Step 1: absent everywhere), then created with `wfe repo create`
under the configured owner.

## pj-ponderer

- **History import** (`bbbe548`): pj-argue `main` (`4fd89fd`, 140 commits)
  was merged with `--allow-unrelated-histories`. The tree takes argue's
  `discussions/`, `episodes/` and `tools/` byte for byte
  (`git diff argue/main HEAD -- discussions episodes tools` is empty).
  - Old citations resolve in pj-ponderer under their original commit ids.
    Checked: `e4937184de19` and `bece165a2d87` (cited in the provenance
    finding) are commits here.
  - argue's 17 run branches are tags `history/argue/wfe/argue/<wf>/run-NNN`
    on Gitea.
  - argue's run records and agent files stay in history only.
- **Structure** (`2b0be53`):
  - `study/study-argue`, `wedo/wedo-argue`: submodules with relative URLs,
    as archsage and ppt bind theirs. The pins are the adopted revisions.
  - `knowledge/`: removed. Its findings are in wedo-argue, so no competing
    copy remains.
  - `tools/argue` → `tools/discussions`:
    - `cite` prints `ponderer@<commit>`;
    - `knowledge` became `wedo`, which shows the pinned wedo-argue index,
      each finding's status and whether the checkout equals the pin;
    - `validate` passes for all 11 discussions, and `episodes validate`
      for all 3 episodes.
  - `HISTORY.md`: where every argue path went, how `argue@<commit>`
    resolves, what was deliberately not changed (speech, authors, run
    identities, the format names `ag.argue.discussion.v1` and
    `ag.argue.episodes.v1`).
  - `project.yaml`: intent and goals now cover conducting, records and
    learning. Areas: `discussions`, `episodes`, `tools`. `README.md`,
    `discussions/README.md` and `episodes/README.md` updated.

### Workflows (smallest useful set)

| Workflow | Lane | Who starts it | Does |
| --- | --- | --- | --- |
| `conduct` (unchanged definition) | conversation | an untargeted message, a later outcome, a request | progression as activities |
| `synthesize` (new) | default | a delegate node of the conductor run, or a request | `discussions/<id>/summary.md`, then `contribute --kind synthesis` |
| `evaluate` (new) | default | a request (person, Front) | `episodes/`; commissions `archsage/study` → study-argue; revises wedo-argue (bound editable); adopts by moving pins |
| `freework` | default | a request | — |

All four are valid (`wfe validate`). Notify provisioned
`wf-ponderer.synthesize` and `wf-ponderer.evaluate` by itself, so a
delegation can reach them.

### Routing for synthesis

- WFE identifies a discussion participant by workspace. `contribute` maps
  any run of the facilitator's workspace to the participant `ponderer`,
  and an invitation to `ponderer` is an invitation to the facilitator. A
  second ponderer entry in `participants` would be refused by the config
  check, and it would collide with the conductor anyway.
- Ordinary delegation already has the needed semantics:
  - only the same workflow is refused (`delegate-self`);
  - `conduct → synthesize` is a delegate node;
  - the conductor waits without holding the slot;
  - the child's result continues the conductor;
  - the child's `--kind synthesis` message is accepted, because its
    participant is the facilitator's workspace.

  No routing change, no alias, and a synthesis cannot start a conductor.
- `agent/conduct.md` states this as a fact: an invitation to `ponderer`
  reaches the conductor itself, not `synthesize`.
- Live proof is in Step 5's discussion.

### Guides

- `agent/guide.md` (all workflows):
  - the head names the tools (`wfe discussion`, `wfe run`,
    `wfe delegate`, `wfe projects`, `tools/discussions`) and the
    repository areas;
  - the facts now include records kept apart (transcript, summary,
    annotation, evaluation; statement, proposal, inference), learning as
    deliberate promotion with adoption by pin, and subject findings as
    their owners'.

  The "Preserving is argue's" fact is gone. Trial references are kept
  (p3, p3 ex1, run-009).
- `agent/conduct.md`: where earlier episodes and the pinned wedo are, and
  the synthesize-by-delegation fact.
- `agent/synthesize.md`: argue's synthesize guide, rewritten for ponderer.
  It keeps the p3 ex1 and p16 step 4 trial references and adds the
  p16-s4-provenance #145 "no hand-off" case.
- `agent/evaluate.md`:
  - evidence kinds (said / executed / earlier readings);
  - the episode form;
  - wedo revisions with contrary observations;
  - commissioning `archsage/study` for one bounded question, with an
    incomplete study reported as such;
  - adoption as a pin move;
  - practice changes as facts, not sequences.

## Responsibility changes elsewhere

- **archsage** (`92ad65a`):
  - `study/study-argue` submodule, bound `editable` in `study` and in its
    `research` node, so ponderer can commission literature research
    through the existing study workflow. No second study system.
  - `index/sources.yaml` gains `study-argue`, `wedo-argue` and
    `ponderer-discussions`.
  - New `index/domains/discussions.md`.
  - `agent/discuss.md` points to ponderer's records instead of "the argue
    project's knowledge/".
- **forge** (`c4091ee`):
  - `knowledge/asset-provenance.md`: argue's provenance conclusion, text
    unchanged, plus an ownership paragraph and a revision line. It says
    this is the adopted conclusion of p16-s4-provenance, kept as the
    person asked (#145), and that no implementation is requested.
  - A `knowledge` area in `project.yaml`, `knowledge/index.md`, and one
    line in forge's guide.
  - The discussion and its record stay ponderer's.
- **Front**: no change needed. Its guides already name ponderer as the
  conductor; its argue mentions are trial references.
- **agautolab wfrun guide**: no change needed (no argue dependency).
- Pending for Step 3/4: WFE configuration (archive destination,
  `argue/synthesize` participant), WFE code and help, and aghangout
  characters, fixture and README.

## Knowledge ownership after this step

| Knowledge | Owner, place |
| --- | --- |
| Conducting, preserving and evaluating discussions (reusable) | ponderer, `wedo-argue` (adopted at ponderer's pin) |
| Literature on discussions | ponderer (agenda), written by archsage, `study-argue` |
| One discussion's record, summary, evaluation | ponderer, `discussions/`, `episodes/` |
| Asset provenance (subject of p16-s3/s4) | forge, `knowledge/asset-provenance.md` |
| Where knowledge lives | archsage index (points, does not copy) |

## Omni handoff notes

- did migrate the three facilitation findings into wedo-argue for agent ponderer-agent — handoff candidate
- did record the provenance conclusion as forge knowledge for agent forge-agent — handoff candidate
- did add the discussions domain and the study-argue destination to archsage's index and study workflow for agent archsage-agent — handoff candidate
