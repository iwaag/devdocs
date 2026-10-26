# tidy p1 — Step 2 report: the tidy activity and the fix for root reports

Date: 2026-10-27 (JST). Machine-specific details (Gitea ids, logs, rollback commands) are in the ignored
`pj-agdev/.local/tidy-p1/notes.md`.

## Shared guidance in-system runs can read

- **devpolicy is now a registered repository.** Gitea `developer/devpolicy` is a pull mirror of the
  devpolicy repository (interval 10 minutes; same history), registered with
  `wfe repo register developer/devpolicy --category other`. Any run reads it with
  `wfe projects read repo:developer/devpolicy styles.md`; verified at `078ec243d4`. The repository stays the
  one authority: it is edited where it always was, and Gitea only follows it.
- **`styles.md` *Keeping a project tidy*** (devpolicy `078ec24`): six principles with the incident that
  motivated them:
  - one maintained current entry;
  - records stay in their run or episode folder;
  - capability, installed bytes and verification are separate facts;
  - generated caches are ignored;
  - one machine's facts stay out of portable text;
  - domain folders are legitimate, and tidying never rewrites history.

  The layout list now names `devdocs/runs/<workflow>/<run>/`, so WFE run records no longer look like a
  deviation from the example.

## The tidy template

`wfe nodes create tidy` (catalog template `tidy` r1, type `do`, icon cleanup, green; by Omni Agent
(tidy p1)).

- **Description** (copied into the node): declare the target, the revision and the scope. Compare the
  maintained entry with the implementation, the deliveries and the evidence, keeping capability,
  installation, runtime verification and human judgement apart. Look at current state, misplaced records,
  references, caches and machine facts. Correct within the run's editable bindings while preserving
  history. Return another project's defects to its owner. Verify by what changed, look for the cause of a
  recurrence, and record all of it in the run folder. It names devpolicy as the shared standard and the
  project's guide for its exceptions.
- **Expectation** (for designers): a tidy node after meaningful implementation or integration, before the
  result is handed over, or when a check or tidy is asked for. A small edit needs no full audit, and no
  layout is required.

There is no new node type, no automatic insertion and no gate. A run created from now on receives the
template in its frozen catalog (`wfe run nodes`). Earlier runs keep their snapshots.

## The fix for root reports, at WFE

Step 1 traced the cause: the brief never gave the run folder as a path. A step's document was named
only once the step was ready, and `manual` runs got none.

- agdevworld `a6564d3`, `server/exec.ts` `runFolderLine`, `documentationBrief`:
  - every bootstrap brief, whatever the policy, says `Run folder: devdocs/runs/<wf>/<run>/ (relative to
    your working directory, the copy's root)`, what goes there, and why a root file of the same name is
    wrong, with the two observed cases;
  - documentation steps not reached yet are listed with their document paths.
- Tests: `test/docsteps.test.ts` (the default and manual briefs). The whole suite passed (623) locally and
  in the release check.
- Contract: `workflow_editor/docs/runs.md` *Documentation steps*, "Where documents go (tidy p1)".
- Checked on the deployed release: `documentationBrief` rendered for protoprey `freework/run-003` and agops
  `recover/run-037` (manual) now names `devdocs/runs/freework/run-003/report.md` and
  `devdocs/runs/recover/run-037/`.

The wfrun guide was not changed. It already says that documents are files of the run folder, and the
missing piece was a path, which now comes with each run.

## The original wrong inference, at the tool that gave it

ponderer concluded "no 3D scene" from a tree search of protoprey. `wfe help projects` (agdevworld
`46fe968`) now says what the tool cannot see: ignored folders such as installed deliveries, shares and
hosts. It says that finding nothing is no evidence of absence, and points to the project's entry and
adoption records, citing the incident. The fact lives with the tool's usage, not in a guide.

## Not changed, and why

- **No reminder for a run that completes with a documentation step still ready.** run-003 ended that way.
  Documentation steps are optional by design (docs/runs.md), and the reminders already present were passed
  with `--yes`. With the path now given, the cheaper fix is in place. Step 4's follow-on run shows whether it
  is enough.
- **protoprey's guide** ("read … `report.md` first") is the project's own text. It is corrected by
  protoprey's run in Step 3, not here.
- ponderer's guide: the catalog brief already shows every run the template. Commissioning is its existing
  delegate path.

## Deployment

| Component | Before | After |
| --- | --- | --- |
| WFE service and launcher (studio) | `9fd7a3f` | `a6564d3`, then `46fe968` (no store change; rollback is the release alone) |
| agpc worker's wfe | `9fd7a3f` | `46fe968` through nctl (placement `agautolab-agpc`) and the worker playbook; drift converged |
| executor / wfrun guide (agautolab) | `eeddf95` | unchanged |
| node catalog | 6 templates | + `tidy` r1 |
| devpolicy | `69db240` | `078ec24` (GitHub and the Gitea mirror) |

The first release attempt of `a6564d3` failed two `watch.test.ts` file-watcher tests during the release
check. They passed in the local run just before, and again on the retry with nothing changed. They are
timing-dependent and unrelated to this change.

## Next

Step 3: a ponderer-led request on behalf of the Developer, delegating tidy work to protoprey,
gimmickimg and lowerpoly.
