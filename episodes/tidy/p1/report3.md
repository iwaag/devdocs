# tidy p1 — Step 3 report: cross-project tidying through ponderer

Date: 2026-10-27 (JST). Trial conversation `tidy-p1-trial` (category conversation, author "Omni Agent (for
Developer)"; the Developer said nothing in it). Machine-specific details are in the ignored
`pj-agdev/.local/tidy-p1/notes.md`.

## How it was commissioned

The room was created with no members and `wfe conversation start` asked ponderer to lead. The brief (#447)
said it was an Omni request on the Developer's behalf. It explained the original incident and listed
each project's findings as of 2026-10-26 (to be re-verified), the new `tidy` template and the devpolicy
standard. It asked for three independent requests, individual assessment and a final summary. It did
not prescribe edits.

## What happened

| Time (UTC) | Event |
| --- | --- |
| 20:07 | ponderer `conduct/run-048` started; one ponder node added delegate nodes `tidy-protoprey` (d1), `tidy-gimmickimg` (d2), `tidy-lowerpoly` (d3), each to the project's `freework` |
| 20:09 | #448 progress post; children protoprey `freework/run-006`, gimmickimg `freework/run-016`, lowerpoly `freework/run-004` created, each with `tidy` r1 in its frozen catalog |
| 20:12–20:17 | each child added a `tidy` node from the template (run revision r2, reason "request asks for a tidy node"), worked in it, wrote its report and completed |
| 20:14–20:17 | ponderer judged each result satisfied 12–23 s after taking it up; the last completion put run-048 to rest. **No summary was posted.** Request 226 stayed unanswered |
| 20:33 | Intervention 1 (#449): Omni noted the summary was missing |
| 20:33 | #450 summary: "all three completed, nothing major unresolved" |
| 20:34 | Intervention 2 (#451): Omni named two facts that contradicted the summary |
| 20:35 | #452: ponderer agreed and returned d1 and d3 to the same children (`delegate revise`) |
| 20:37–20:38 | corrections came back; both judged satisfied; #453 reported d1 only |

## Results per project (published main after the trial)

**protoprey** `dcfc792937` → `99762ff527` (run-006: `f3e1b56`, `1ecdcc4`, `dfb4f5c`).

- README: new title, and a "Current state" table that keeps **built / adopted / installed / verified by
  running** apart. It names the adopted deliveries (gimmickimg `mushroom_waterfall` r2, playprey `wolf` r1,
  agsfx `location_sfx` r3) and says their bytes are in the ignored `external/`, "so searching the sources
  does not show the wolf mouth scene".
- "Run on macOS only" became a statement that `VERIFY.md` was done on macOS. The child found no Linux
  verification in the repository and did not claim one; the project_coop p1 Linux check lives in devdocs,
  not in protoprey.
- Esc in the mouth scene is stated.
- Root `report.md` moved with `git mv` to `devdocs/runs/freework/run-003/report.md`. The earlier Task 4
  report was restored from `243a95d` to `devdocs/reports/task-4-report.md`.
- `agent/guide.md` now points at README and `devdocs/runs/`.
- The dangling `direction/REFERENCES.md` references in README, GOAL and VERIFY were replaced or removed.
- Authored data, tuning, tests and VERIFY results were untouched.
- **Defect, corrected:** the first result had no `devdocs/runs/freework/run-006/report.md`. The command
  `sed -i '…' README.md && cat > report.md <<E` failed at BSD `sed`, so the report was never written. The
  agent still completed `_final-report` ("Report written") and told ponderer the audit record was there.
  ponderer judged it satisfied 14 s later. After revise c1 the report exists (`dfb4f5c`, sha256-checked by
  ponderer).

**gimmickimg** `feec1955fc` → `84af344030` (run-016: `0430cb2`).

- 148 `.godot/` files of three Godot projects untracked with `git rm --cached`; `.gitignore` covers
  `.godot/` everywhere.
- Before that, each project was copied to `/tmp`, its `.godot/` removed and re-imported headlessly. The same
  file counts (30/108/10) came back, exit 0.
- `sheet-review.md` moved to `devdocs/runs/freework/run-009/review-summary.md`. Root `sheet/` was removed
  after the child found it byte-identical to run-009's frozen presentation `p1/content` (it stays in
  history). Nothing referenced either.
- Sources, `.import` files, experiments and deliveries were untouched. Not checked: that the scenes
  render the same after the re-import (Step 4).
- Accepted without revision. An attempt was interrupted midway (`b31f8c7`), recovered and continued
  without intervention.

**lowerpoly** `fe4fa5d744` → `8d1bf5278d` (run-004: `128a973`, `cd20d77`).

- New root README:
  - the `lod-chain` offer, its input (a single-mesh static GLB on the share) and its output (LOD GLBs,
    `lods.json`, evaluation, optional `MANIFEST.json`);
  - how to ask, and an evidence table of the three `lod-chain` runs;
  - "not established": no human visual judgement, Godot verified only headless, rig data lost,
    **no consumer delivery format yet**, the manifest is a draft;
  - build cost, folders and requirements.
- **Defect, corrected:** the first README ended with a "Host-specific notes" section: the original host,
  its home path, the NFS backing path and a LAN IP URL. That copied one machine's facts into the portable
  entry, against the standard. ponderer had not noticed it. After revise c1 the section is "Inputs and
  environment" with portable statements only. LAN URLs remain in `wedo/` findings as dated evidence,
  which the standard allows.
- The run report (`devdocs/runs/freework/run-004/report.md`) still describes the first version of the README.
  The correction was not added to it.

## Interventions and what they changed

| # | What Omni supplied | Owned mechanism improved | Repeated |
| --- | --- | --- | --- |
| 1 | the summary #447 asked for was missing | WFE `2cc9cd9`: reminder `conversation-unanswered` when a standing conductor's completion would put it to rest with the person's latest words having no reply, result, question or announcement from it. p18 assumed a resting conductor leaves nothing unsaid; run-048 shows that assumption fails | ponderer's next turn posted the summary (#450). The reminder was deployed after run-048's first rest, so it did not act in that turn |
| 2 | protoprey's report did not exist; lowerpoly's README carried host facts | WFE `964e80b`: reminder `document-missing` at the completion of a documentation step whose document is absent, and the same check `document-missing` in the delegated-result assessment (`wfe delegate show`, take-up, the `delegated-document-missing` reminder at `dispose satisfied`). Deployed at 20:21, before the intervention; d1's assessment then read `MISSING document-missing` | d1 and d3 were revised to the same children and corrected. The host-facts case has no mechanical check: it is judgement against the standard |

Both reminders are tested: `test/reminders.test.ts`, `test/assessment.test.ts`, `test/activities.test.ts`
(625 tests pass). agpc pins wfe `2cc9cd9`.

These corrections were **not independent successes**. Without the interventions the published state would
have had a falsely reported audit record and host facts in lowerpoly's entry, and the room would have
had no summary.

## Assessment of ponderer's coordination

- **Worked unprompted:** three separate delegations with project-specific requests, run in parallel,
  failures isolated. The children found and used the `tidy` template. ponderer never edited another
  project's files. Revisions went back to the same children.
- **Weak:** each acceptance took 2–23 s, from the child's outcome line, without reading the evidence.
  Unprompted, it caught neither defect. Even after the assessment showed `MISSING`, its summary turn
  re-read published main rather than `wfe delegate show`. After the corrections it reported d1 but not
  d3 (#453), and `conversation-unanswered` did not fire because #451 already had a reply (#452).
- The children worked fast (protoprey 91 s, lowerpoly about 2 min, gimmickimg about 5 min). None ran the
  game or a load check; protoprey rests its new README on reading data files. Step 4 checks this.

## Next

Step 4: fresh readers, a gimmickimg fresh-copy load check, and a follow-on protoprey run that should keep
its report in its run folder.
