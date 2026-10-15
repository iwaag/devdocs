# project_migration p4 — step 3: archsage built, references migrated

Written 2026-10-15. `pj-archsage` and the trial targets' changes were built in isolated migration clones under the operating area's `migration/`. They are committed there and not pushed yet; step 4 pushes and registers them. They were validated with the release candidate, agdevworld `1d13ff9`.

`1d13ff9` adds one small system change on top of step 2's `f94f07a`: a top-level `study`/`wedo` folder now has that category in the Project Editor too, as it already does in project access. The contract text was updated to say so.

## pj-archsage (`c995255`, 31 files)

| Path | Content |
| --- | --- |
| `project.yaml` | `ag.project.v2`, id `archsage`, devdocs directory; local areas `index`, `questions` |
| `study/study-rts`, `wedo/wedo-ppt` | shared repositories as submodules (relative URLs): the writing destinations for the trials |
| `index/README.md` | the index format, and **Maintenance**: the `knowledge.yaml` contract (`project` / `supplemented`; absent means unknown, and unknown is not eligible) |
| `index/sources.yaml` | 14 sources with address (`<project>:<path>` or `repo:<owner>/<name>`), repository, form (shared / root-local / external), maintainer, domains, what each applies to, observed revision with selector, refresh outcome. GitHub `study-arxiv-trend` is listed as `inaccessible`, with the note to read `autodev/papers` instead. |
| `index/domains/` | 10 domains (see below) |
| `questions/` | the format, and the 2 imported queue notes, marked *probably answered — verify* with the reason |
| `agent/guide.md` | archsage-agent's head and facts: source authority and citation from provenance; sourced, inferred and unknown kept apart; unreadable sources reported; devdocs evidence levels; gaps become questions; writing only through the run; shared update vs consumers' pins; maintenance declared, not inferred; the index records reading |
| `agent/consult.md`, `study.md`, `backfill-wedo.md` | each task's contract: its result file, and what it says |
| `devdocs/workflows/` | `consult`, `study`, `backfill-wedo`, `freework` |

### Seeded references, with provenance

| Domain | Sources | Origin |
| --- | --- | --- |
| aisvgs, arxiv, growbox, protoprey, worldtrend | `autodev/aisvgs`, `papers` (and the external `study-arxiv-trend`), `growbox`, `protoprey-research`, `worldtrend` | Imported from `autodev/archsage-sages@4612ea6:<name>/guide.md`, with the wording kept. "The tree" now reads "the source", and the sage queue became proposed research questions. aisvgs also records two open items: round 3 is parked, and a personal e-mail address remains in its strand-7 report (autolab's repository, not edited). |
| rts | `study-rts` (pinned `d2734db`), rts-vs-bot's run records (`ba2daa7`) | Written from the projects as read through `wfe projects` |
| presentations | `study-ppt` (`e9c153b`), `wedo-ppt` (`293227c`) | the same |
| sfx | agsfx `study/study-sfx` (root-local, `e8d82d5`), `wedo-sfx` (`81bc97c`) | the same |
| jev | jevlab `study` (root-local, `b3599ae`) | the same |
| media-assets | forge `catalog` (root-local, `f751c14`) | the same |

The old sage store and its trees are not copied. The index points at the sources and records the revision observed.

### Workflows and mutation boundaries

| Workflow | Documentation | Nodes | Editable (besides the run folder) |
| --- | --- | --- | --- |
| `consult` | manual (`advice.md` is the record) | research (study) → answer (talk, `confirms: result`) | nothing |
| `study` | plan (scope and destination) | research → present (`confirms: result`) | `study/study-rts`, `index`, `questions` |
| `backfill-wedo` | plan (eligibility, evidence, proposal) | supplement → confirm (`confirms: result`) | `wedo/wedo-ppt` **guarded** (`integrate: if-unchanged`), `index` |
| `freework` | final report | talk | the project root, `index`, `questions`; every knowledge submodule **readonly** (written explicitly, not placed by `basics`) |

- Every workflow runs as `archsage-agent` on `sonnet` and accepts Zulip requests.
- No workflow generates a document after its `confirms: result` talk.
- Path scopes were checked with the shared access resolver. In `freework`, `study/study-rts` and `wedo/wedo-ppt` are readonly. In `consult`, everything is readonly except the run's own records.

## Trial targets (committed in migration clones, pushed in step 4)

| Target | Change | Why |
| --- | --- | --- |
| rts-vs-bot (`1062e00` on `ba2daa7`) | New local area `wedo/`: `knowledge.yaml` (`supplemented`, maintainer rts-vs-bot, supplementer archsage), `README.md` (layout, citation and evidence strength), `SUPPLEMENTS.md` (an empty ledger). A new workflow `backfill-wedo`, owned by rts-vs-bot. | rts-vs-bot keeps no wedo; its run records (improve-game, the ex3 trial pairs) hold practice. This is the eligible **root-local** target. |
| rts-vs-bot `backfill-wedo` | `execution: {agent: archsage-agent, profile: sonnet, guides: [archsage:agent/guide.md, archsage:agent/backfill-wedo.md]}`, intake requests. `wedo` is editable and guarded; the root and the study are readonly. | Knowledge inside another project's root is written by a run of that project that follows archsage's guides, so one run owns the deliverable. |
| wedo-ppt (`365b7a8` on `293227c`) | `knowledge.yaml`: `maintenance: project`, maintainer ppt (its README already says ppt create runs write it) | The **project-maintained** case: archsage must produce no edit there |

Validation with the release candidate, in a scratch registry:

- pj-archsage: 4 of 4 workflows valid, 0 warnings, structure complete, both submodules initialized, both areas recognized.
- rts-vs-bot with the change: `backfill-wedo`, `freework` and `improve-game` are valid. The `archsage:` guide form is accepted. `wedo` is recognized as a local area.
- rts-vs-bot's `build-game` has 2 errors: it is an `ag.workflow.v1` definition from before p5. This was already there, it is not touched, and it does not block other runs.

## How references are found from a run copy

Nothing archsage needs is an ignored file:

- The guides are read at claim from pj-archsage's committed `main` (step 2).
- The index is read with `wfe projects read archsage index/…`, from any copy of any project. The executor puts the area's `wfe` on `PATH`, and that `wfe` pins the registry.
- The destinations are tracked submodules.

The cross-project guide read is covered by `test/access.test.ts`. The live lookup from a real run copy is step 4's first check, after the project is on Gitea.

## Reproducible migration procedure

1. In the operating area's `migration/`, `git init` pj-archsage, with `origin` = the Gitea `developer/pj-archsage` URL.
2. Add the destinations: `git submodule add ../study-rts.git study/study-rts` and `git submodule add ../wedo-ppt.git wedo/wedo-ppt`, run with the editor's Gitea credentials (a helper that passes `Gitea.gitEnv()` to Git).
3. Write `project.yaml`, `README.md`, `agent/`, `index/`, `questions/` and `devdocs/workflows/`.
   - Import each sage domain from the store with its provenance header.
   - Write the managed projects' sources from `wfe projects tree|read` reads, recording the commit each read returned.
4. Validate: `wfe register <clone>` and `wfe validate` against a scratch registry, plus the access resolver for the path scopes.
5. Clone the targets (rts-vs-bot, wedo-ppt) beside it, commit the maintenance files and the target-owned workflow, and validate the same way.
6. Step 4:
   - push the commits, with the editor's Gitea module (no plain `git push` credentials exist);
   - `wfe project register developer/pj-archsage`, then `wfe project obtain archsage`;
   - register the autolab study repositories as shared `study` repositories (`wfe repo register autodev/<name> --category study`);
   - provision the channels, through the notifier as for forge.
