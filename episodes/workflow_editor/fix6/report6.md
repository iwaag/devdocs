# fix6 step 6 — deployment and behavioural trial

Plan: [plan.md](plan.md) step 6.

## State, backup, releases

- State first: `nctl status` ok; `nctl drift --host agpc` converged; the
  desired placement `agautolab-agpc` pinned WFE `506a116` and the executor
  `8f341ed`; the WFE service ran `506a116`, no attempt open, both workers
  idle.
- Backup of every host store and config before the first release (ignored
  notes). No store schema changed; run records gained optional fields and
  four operations, so a pre-fix6 release cannot replay a record that uses
  them (recovery steps in the ignored notes).
- WFE: `09596f6` (steps 1–5), then `ab342dd` (the announce change below),
  each with `wfe-release.sh deploy` (release check 571/571, healthy).
- agautolab `15258d7` (wfrun guide); the local executor reads the guide from
  its checkout.
- agpc worker: pins updated through Nautobot (`nctl desired apply`, `render
  production`, `setup_wfe_worker.yml -l agpc`) to WFE `ab342dd` and executor
  `15258d7`; the worker is active, heartbeating, and its `wfe delegate help
  revise` is the new one.

## Trial A — real facilitated conversation, unprompted

Conversation `fix6-trial-a` (category check), check actor `fix6-check`:
"pj-miscで、トールセル（tall cell）による2D格子分割がどう見えるかを示す図を
1枚作ってほしいです。…" — no word about presentation or cleanup.

- ponderer/conduct/run-037 added a delegate node to misc/freework itself —
  with `accepts: parent` — and dispatched. Its dispatch output gave the
  relay sentence ("you relay its questions and presentations to them").
- misc/freework/run-007 (brief: "A report of file paths to ponderer-agent
  is not a presentation to fix6-check") **presented on its first attempt**:
  a `draw-figure` node, then `present` (p1 served, link in its question),
  `cleanup`. Successful delivery to the parent.
- ponderer answered the child's review question **itself, 23 s after taking
  it up** ("Checked the PNG: acceptable as is … Please stop the page"),
  without asking fix6-check. The child stopped the page and completed.
- ponderer took the result up, read the assessment ("Assessment is all ok"
  in its attempt), judged it satisfied and reported with `wfe conversation
  announce` (#366): "It was presented for review and the presentation has
  been stopped". **The page never reached the person.**
- Why nothing warned: the parent chose `accepts: parent`, so the plan's
  review check (scoped to `accepts: person`) did not apply; the presentation
  existed and was stopped, so the other checks were `ok`; and `announce` was
  not an assessed report boundary. The last point was a gap in step 2's
  coverage (the motivating case reported with `reply --kind result`), so
  `ab342dd` assesses a conductor's announcement for its delegated results
  (unit-tested; fix5's own report checks are not added there). It would not
  have changed this trial (all checks were `ok`), so the trial was not
  repeated. Whether a page reviewed only by the parent's agent under
  `accepts: parent` should be flagged is a scope question left to the
  Developer (report.md).

Attempts: conduct/run-037 `amvmblz8b-f1fc30`, `amvmbr6wt-14dadd`,
`amvmbu8zo-20381a`; misc/freework/run-007 `amvmboxet-cbb42b`,
`amvmbsyte-e53417`.

## Controlled fixture — the deployed correction path

agops `correction-check/run-001`, operated by hand as `fix6-check` (no
execution host), delegating to misc/freework with `accepts: person`; its
request ended "only write the file … Do not prepare a presentation" to make
the omission certain.

1. misc/freework/run-006 wrote `squares.svg` and completed without a
   presentation ("no presentation, as requested").
2. Take-up printed the assessment: `MISSING presentation-missing` (work
   `draw-svg`, viewable candidate `…/squares.svg`, no presentation),
   `UNKNOWN review-unrecorded`, the earlier integration as a historical fact.
3. `dispose satisfied` was refused; its last line: `In short:
   delegated-presentation-missing: d1 (freework/run-006) completed draw-svg
   with wedo/wedo-misc/trials/fix6-check/squares.svg and presented nothing →
   wfe delegate revise … | delegated-review-unrecorded: …`.
4. `wfe delegate revise` returned it. The same child received correction
   `c1`, was continued once, added `present-svg`, `confirm-svg` and
   `cleanup-svg` after its completed work, served p1 and asked fix6-check
   (relayed through the parent) with the link.
5. The page answered from agpc (another LAN host); fix6-check answered as
   itself. The child stopped p1 (verified gone) and completed again.
6. The new result (event 5) assessed all `ok`; `dispose satisfied` passed
   without reminders; the run completed. The child's earlier integration and
   the corrected result's new one are both listed.

Attempts: misc/freework/run-006 `amvmbo0yh-8d7d93`, `amvmbrhsm-15b628`,
`amvmbufzu-4e8392`.

## Verified from another LAN host

From agpc: the fixture's review page (p1, `:8301`) while it served, and
`GET /wfe/api/workspaces/agops/runs/correction-check/run-001/delegations`
with the assessment and the correction. The run view
([screenshot](run-view-correction.png), Detail → Delegations) shows the
returned result, what was asked, and each check of the corrected result.

## Pages

Both review pages (run-006 p1, run-007 p1) are stopped and verified gone.
No page was kept on request; no disposable page is left serving.

## Deus ex machina

None: the Omni Agent presented nothing for a child and answered no question
for the Developer; the fixture's parent was operated by the identified check
actor by design.
