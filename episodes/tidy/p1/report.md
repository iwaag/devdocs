# tidy p1 — final report

Date: 2026-10-27 (JST). Steps: `report1.md` (evidence, cause), `report2.md` (activity, fix), `report3.md`
(ponderer-led trial), `report4.md` (verification). Dispositions of every finding: `problem.md`
*Disposition after tidy p1*. Machine-specific notes and rollback commands: the ignored
`pj-agdev/.local/tidy-p1/notes.md` and `pj-agdev/.local/devenv.md` *tidy p1 on this host*.

## What exists now

- **A tidy activity**: catalog template `tidy` r1 (`do`). It is offered to every new run and visible at
  `http://<host>:8093/wfe/` → User nodes, and through `wfe nodes show tidy`. It uses existing execution
  semantics: no new type, no gate, no automatic insertion.
- **One shared standard readable in-system**: devpolicy `styles.md` *Keeping a project tidy* (`078ec24`),
  read by runs as `wfe projects read repo:developer/devpolicy styles.md`. Gitea `developer/devpolicy`
  is a pull mirror of the one devpolicy repository.
- **The cause of root reports fixed where it was**: the WFE brief now names the run folder and every
  documentation step's document as paths (agdevworld `a6564d3`). Before, "this run's folder" was read
  against the copy root about half the time (agops recover 11 of 21, cagent consult 2 of 7,
  protoprey run-003).
- **The tool behind the original wrong answer** says what it cannot see (`wfe help projects`,
  `46fe968`): finding nothing in tracked content is no evidence of absence.
- **Two gaps the trial exposed, now reminders** (`964e80b`, `2cc9cd9`):
  - `document-missing` at a documentation step's completion and in the delegated-result assessment;
  - `conversation-unanswered` when a standing conductor would rest with the person's latest words
    unanswered.

  Releases on studio and agpc are `2cc9cd9`. The suite passes (625). The store is unchanged, so
  rollback is the release alone.

## Project results (published main)

| Project | Before → after | Outcome |
| --- | --- | --- |
| protoprey | `dcfc792937` → `2ee7d8f28d` | README is the current entry (built / adopted / installed / verified); run-003's report is in its run folder; the Task 4 report is under `devdocs/reports/`; the guide is fixed; dangling references are gone. `deliveries.py status` warns about non-share sources (run-007, test made effective in run-008) |
| gimmickimg | `feec1955fc` → `84af344030` | 148 generated `.godot/` files of three projects untracked and ignored, reproducible from a fresh clone; the review record moved into run-009 |
| lowerpoly | `fe4fa5d744` → `8d1bf5278d` | root README for consumers: the `lod-chain` offer, input/output, request route, evidence, limits; no host facts |

## Interventions (not independent success)

1. ponderer rested without the promised summary. Omni asked for it (#449). → `conversation-unanswered`.
2. ponderer accepted a protoprey result whose report was never written (a failed `sed && cat` chain)
   and a lowerpoly README with host addresses, each within seconds. Omni named both (#451). → the
   `document-missing` reminder and assessment check. ponderer then revised both children correctly.
3. The follow-on protoprey run-007 added a test that never ran. Omni found it. → corrected by successor
   run-008. No mechanism was added: it is ordinary verification.

## Verification

- A fresh reader (`tidy-p1-reader`) answered the incident's question correctly: implemented and adopted,
  how a fresh copy installs it, what is unverified. It stated unprompted that source-search absence is not
  evidence. It also found lowerpoly's offer and limits from the README alone.
- Fresh-clone Godot import and run of gimmickimg's three projects: same cache files, no errors.
- Every run created after the brief change kept its report in its run folder: protoprey 006–008,
  gimmickimg 016, lowerpoly 004.
- Concurrent work kept: protoprey run-005's agsfx waterfall launcher stays in the README.

## Limits and what remains

- One trial, one reader, two follow-on runs. This is evidence of improvement, not proof that misplaced
  reports cannot recur.
- **ponderer's assessment is the weak point.** It accepts results from their outcome line in seconds
  and reads assessments only when told. The reminders put the facts in front of it, but judging them
  is still its behaviour. A guide change was not made, because the reminders were deployed mid-trial and
  have not yet been seen working unprompted.
- Runs can still complete with `_final-report` unrecorded (run-003, run-007). Documentation steps are
  optional by design, so no reminder was added for this.
- **Open findings** (`problem.md`):
  - protoprey's agsfx `location_sfx` adoption points at a path in agsfx's run copy on one host (agsfx
    must publish it);
  - agops' root `report.md`/`review.md`/`page`/`presentation*` and cagent's root `answer.md` are not
    moved;
  - agsfx's README does not describe its delivery;
  - rts-vs-bot, ppt, jevlab, agops, front and misc have no README;
  - lowerpoly's README dropped the "needs the toolchain host" constraint.

## Recommended next scope

A second, small batch through ponderer, with the same template, after one more unprompted trial of the
new reminders:

1. **agsfx**: publish `protoprey_locations` r3 to the share and describe the delivery in its README; then
   protoprey re-adopts it by reference.
2. **agops and cagent**: move the root records into their runs, preserving cagent's dated evidence.
3. **README entries** for the projects without one, starting with those other projects ask: agops,
   front.

A dedicated tidy workflow or a "tenderer" project is not warranted yet. `freework` plus the template
served all three projects.

## For the Developer to look at

- Room `http://agstudio.local:8093/hangout/?conv=tidy-p1-trial` (the trial, interventions included) and
  `?conv=tidy-p1-reader` (the fresh reader's answer).
- protoprey README on Gitea main (`2ee7d8f`), lowerpoly README (`8d1bf52`), and the catalog's `tidy` at
  `http://agstudio.local:8093/wfe/` → User nodes.
