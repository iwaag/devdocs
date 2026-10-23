# Challenge p3 — Step 3: migrating existing records

Script: agdevworld `d71e6f6`/`a252289`,
`workflow_editor/scripts/challenges-p3-layout.ts`. I rehearsed it on a copy
of the live repository and journal. The live run belongs to the cutover in
step 6: the WFE and aghangout releases that read the new layout go live at
the same moment. If only the data moved, the deployed WFE `21a094e` would
find no challenges, and the deployed aghangout would not understand the new
API shape. The outcome of the live run is in [report6](report6.md).

## What exists (inspected)

| Path at `6b72a13` | Meaning | Destination |
| --- | --- | --- |
| `challenges/ch-0001.md` | the gimmickimg edge-remnant statement | `challenges/ch-0001/summary.md` |
| `evidence/ch-0001/input-green.png`, `wan-c-contact.png`, `wan-composite.mp4`, `run-006-report.md` | output and review of gimmickimg `freework/run-006`, the work that **exposed** the problem; none of it is an attempt at the extraction problem | `challenges/ch-0001/evidence/…`, at the challenge's level (problem examples and source material) |
| `challenges/ch-0002.md` | p1 deployed-check fixture (resolved), no files | `challenges/ch-0002/summary.md` |
| `README.md` | describes the old layout | rewritten for the directory layout |
| `challenges/.gitkeep`, `evidence/.gitkeep` | placeholders | removed (the directories now have files or are gone) |

No historical trial was created. The "Ideas so far (from run-006)" section
stays in the body as written. It is not a recorded attempt, and turning it
into a trial would invent one.

## The move

The migration is one ordinary Git commit, authored by `Omni Agent
(challenge p3 migration)`, with `Challenge-Request:
migration:challenge-p3-layout` and `Challenge-Via: migration`:

- `git mv` keeps every byte.
- Relative Markdown links are rewritten to the same targets: the four
  `../evidence/ch-0001/<f>` links in ch-0001 become `evidence/<f>`. There
  are no other links.
- Ids, titles, states, projects and the free body are unchanged.
- File names inside each challenge stay the same (`evidence/<f>`), so a name
  read at an old commit is still valid.
- Running the script a second time does nothing.

## Rehearsal

Paths are in `pj-agdev/.local/challenge-p3/rehearsal/`: `plan.json`,
`verify.json`, `verify.ts`.

- **Index.** The new store reads the moved copy. Counts are open 1 and
  resolved 1; custom and unspecified are 0.
  - `ch-0001` still shows its creator (Omni Agent, `8176fd6`) and its four
    files.
  - `ch-0002` keeps its p1 check author.
  - The last change of each is the migration commit.
- **History.** `ch-0001` has two entries across the move: the migration and
  `8176fd6`.
- **Existing attachments.** I checked every attachment in the live
  conversation store (4: conversations `…chp1fg` #312, `…vv689m` #314 and
  `…chp2ck` #316/#320, all `ch-0001@8176fd622497`). Each re-read at its
  pinned commit:
  - reads as `layout: legacy`;
  - has a **sha256 equal** to the one stored with the message, so runs that
    were given it still verify;
  - resolves its four files from `evidence/ch-0001/<f>`.
- **Current read.** The moved copy is `layout: directory`; the body links
  resolve to `challenges/ch-0001/evidence/<f>`.
- **Journal.** On a copy of the live `challenges.sqlite` (3 published
  operations, none open), schema v1 → v2 was applied. Old rows derive their
  `paths` in the new layout. This is display only: they are published, and
  nothing re-checks them.

## Cutover procedure (executed in step 6)

1. Back up `<area>/.local/challenges/challenges.sqlite`. The repository
   itself is backed up by the agpc mirror and by Gitea.
2. Activate WFE `≥ a252289` (it reads both layouts; the journal becomes v2).
3. Clone the repository and run the script with `--readme` and `--commit`,
   then push to Gitea (fast-forward only).
4. `wfe challenge retry` and `status`: reindex, then mirror catch-up to the
   migration commit.
5. Re-run `verify.ts` against the live repository and conversation store.

Rollback: `git revert` of the migration commit restores the old layout (no
force push). The previous WFE also needs the v1 journal backup, because it
refuses v2. These details are recorded in step 6 and in the ignored notes.
