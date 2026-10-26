# tidy p1 — Step 4 report: readability, behaviour and recurrence

Date: 2026-10-27 (JST). Published revisions checked: protoprey `99762ff527` (then run-007 and run-008 on
top), gimmickimg `84af344030`, lowerpoly `8d1bf5278d`.

## Results

| Check | Evidence | Result |
| --- | --- | --- |
| Current-state understanding | A fresh conversation `tidy-p1-reader` (ponderer `converse/run-001`, no part in the cleanup), asked only the questions (#454), answered #455 from published main at the named revisions | **Passed.** Implemented (the `mouth` step's interactive choice, adopted playprey `wolf` r1 with source and producer commit). Fresh copy: `tools/deliveries.py install`, the `--from` / `PROTOPREY_SHARE_ROOTS` alternatives, import twice, `--jump`. Unverified: playing inside the game, Linux only as an environment note, nobody listened, tests not run by the reader |
| No false absence inference | the same answer | **Passed.** Unprompted, it said the bytes are not in Git, that a source search does not find the scene, and that finding nothing "is not evidence of absence". It cited both the README and the `wfe projects` help |
| Producer discovery | #455 part 2 | **Passed.** It read lowerpoly's README: the offer (`lod-chain`), input and output, how to ask, limits (static single-mesh input, no visual judgement, headless-only Godot check, no `delivery.json` yet, licence, build cost) and evidence. It used no private workspace knowledge and noted that the local workspace lags. The worker restriction ("runs only on `studio`") is no longer stated in the README; see *Limits* |
| References and history | `git archive` of protoprey main; every backticked path in README, guide, GOAL, VERIFY and cooperation.md checked | **Passed.** Moved reports exist (`devdocs/runs/freework/run-003/report.md`, `devdocs/reports/task-4-report.md` restored from `243a95d`). `direction/REFERENCES.md` is mentioned only as absent. The remaining unresolved names are bare filenames under `tests/`, examples inside `devdocs/runs/` or other projects' files. gimmickimg's moved review record is in run-009's folder; `sheet/` stays in history |
| Cache reproducibility | fresh clone of gimmickimg at `84af344030` (no `.godot/` anywhere): `godot --headless --import`, then `--quit-after 60`, for each project | **Passed** for the three projects whose caches were untracked: `godot/` 30 files, `experiments/window-loop/godot/` 10, `experiments/waterfall-loop/godot/` 108, import exit 0, no errors at run. `player/` (never tracked) reports `Couldn't open directory res://gimmick/scenes` in the fresh clone and in the workspace alike: it needs a built delivery, which predates this episode |
| Follow-on work | protoprey `freework/run-007`: a small real task (Omni for the Developer). `deliveries.py status` should flag a delivery whose source is not a share reference | **Record placement passed:** `devdocs/runs/freework/run-007/report.md`, no root file. **The work had a defect:** the new test was appended after `if __name__ == "__main__":`, so it never ran, and the record said "test added; 7 tests pass". Found by Omni (`unittest -v` ran 7 of 8 `test_` definitions). An instruction to a completed run executes nothing, so a successor `freework/run-008` (`--predecessor run-007`) moved the test into the class (8 tests pass) and recorded the correction, again in its run folder. `_final-report` of run-007 stayed pending although the file exists ("written, not yet recorded") |
| Concurrent changes | protoprey README after the tidy | **Passed.** run-005's agsfx waterfall launcher (`tools/play_waterfall.sh`, merged after the problem record) is kept in the README and its current-state table |
| Batch coordination | Step 3 | **Passed with interventions.** Separate children, separate results, revisions to the same children, no edit of other checkouts by ponderer. Two Omni interventions were needed (report3) |

## A finding the tidy run did not make

The follow-on surfaced what protoprey's tidy missed. The agsfx `location_sfx` r3 adoption records as its
source an absolute path inside **agsfx's run copy on this host**
(`…/pj-agsfx/.local/wfe/runs/create/run-005/tree/…`, producer tree `modified`). It is not a share
reference. It is not installed in the protoprey workspace either. A fresh copy elsewhere cannot obtain it.

This contradicts the cooperation standard and the new README's "Installed … placed by
`deliveries.py install`" for this delivery. The tidy agent read `data/deliveries.json` but did not run
`status`. The repair belongs to agsfx: publish the pack with `wfe run publish`, then protoprey re-adopts it
by reference. That is recorded in run-007's report, and `status` now warns about it. It remains open
(see `problem.md`).

## Limits

- One reader conversation, about two projects. One follow-on task, which itself needed a correction.
  This is evidence of improvement, not proof that misplaced reports cannot recur.
- **Record placement improved on every run created after the change.** run-006, run-007, run-008,
  gimmickimg run-016 and lowerpoly run-004 all wrote their reports in their run folders. Two runs still
  ended with `_final-report` unrecorded (run-003 before the change; run-007 with the file present). The
  `document-missing` reminder covers only a step that is completed without its file.
- The children verified little themselves: protoprey's tidy ran nothing, and gimmickimg checked import
  only. The fresh-copy runs here were Omni's.
- lowerpoly's corrected README no longer states that its runs need the toolchain host (worker
  `studio`). The reader reported "the toolchain must be prepared under `.local/`", which is true but
  incomplete for a requester choosing a node.
