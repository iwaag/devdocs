# Workflow editor fix6 — report

Plan: [plan.md](plan.md); pre-research: [preresearch.md](preresearch.md).
Step reports:

- [report1](report1.md): evidence model and shared assessment
- [report2](report2.md): the assessment where the parent receives and reports
- [report3](report3.md): same-child correction
- [report4](report4.md): dispatch context and reminder wording
- [report5](report5.md): behaviour and recovery checks
- [report6](report6.md): deployment and trial

## What changed

| Plan item | Now |
| --- | --- |
| Shared assessment | `shared/assessment.ts` reads the child's record **as of the result**: its own history replayed to the end of the completing attempt, with its frozen catalog and the files its copy changed. It sets these beside the catalog's expectations (prose, never compiled) and reports what is unknown. Three checks: presentation missing; the person's review not traceable (`accepts: person` with a deliverable; unknown, not a violation); a page still serving (distinguishing open review, kept on request and forgotten cleanup). Each assessment is bound to the event, manifest digest, child seq, revision and catalog. Current state and integrations are listed apart |
| Coverage beyond `work` | an untemplated do/study node the run added with files beyond notes counts as work, for the child's own reminders too. Files are read from every repository the run owns. Coordination, reporting and documentation nodes stay out |
| Parent's receiving and reporting points | the assessment appears at take-up (once), in the brief and in `wfe delegate show`. Its reminders (`delegated-*`, merged with the parent's own) come before anything is recorded at `dispose satisfied`, a `reply --kind result|synthesis` (with or without `--completing`), `announce`, and the completion of the parent run. Satisfied delegations are assessed again at report time; a corrected result is assessed anew. HTTP `…/dispose`, `…/revise` and `…/assessment` take `yes`; remote workers forward the CLI |
| Same-child correction | `wfe delegate revise` checks the result's identity under the parent's lock. The request is delivered as a delegation post, and the child's intake records `correction.receive` and reopens the completed child. The child stays in progress until work is added or `wfe run correction --no-change`. It is continued once and re-queued by reconcile; an attempt that leaves it unaddressed is held `missed-correction`; the next completion is a new result. Stale, returned, closed and competing requests are explicit errors |
| Dispatch and brief | the dispatch shows `accepts`, the relay role and the target catalog (as information). The child's brief names the person, says the request is unchanged, and says a file-path report to the agent is not a presentation to the person |
| Wording | the refusal names its codes first and ends with `In short: <code>: <omission> → <repair> | …`. The "records that need no viewing" escape is gone. The structured `summary` is kept for clients |
| Requested availability | `wfe present keep` (`presentation.retain`) records the person's request to keep a page and who stops it later |

Releases: agdevworld `4888ea3` (steps 1–3), `09596f6` (tests), `ab342dd`
(announce, after the trial). agautolab `15258d7` (wfrun guide). The
ponderer and Front guides were read and needed no change.

## Checks

- WFE `npm run check`: 571/571, tsc, both builds (558 before; 13 new tests
  in `test/correction.test.ts` and `test/assessment.test.ts`). Each release
  ran it again (571/571).
- agautolab pytest: 47 passed, 1 skipped.
- All 351 run records of the registered projects replay under the new
  reducer (read-only), so no migration was needed.
- A read-only assessment of every delegation on the host flags exactly the
  known omissions: misc run-001…004, gimmickimg run-001 and run-010. It
  flags no study, consultation or presented result.

## Deployed verification and trial

- **Controlled fixture** (agops correction-check/run-001 → misc/freework/run-006,
  check actor `fix6-check`), on the deployed release:
  1. the omission was assessed;
  2. `dispose satisfied` was refused with the concrete repair in its last
     line;
  3. `revise` reopened the same child, which added present, confirm and
     cleanup nodes, served the page and asked through the parent;
  4. the page answered from agpc, and the child took fix6-check's answer
     and stopped the page;
  5. the new result was assessed clean and judged satisfied.

  **The correction path works end to end.** This fixture forced the
  omission by its request; it shows the mechanism, not agent behaviour.
- **Unprompted real trial** (conversation `fix6-trial-a`):
  - the child misc/freework/run-007 **presented on its first attempt**,
    which counts as delivery to the parent;
  - the presentation **did not reach the person**: ponderer created its
    delegate node with `accepts: parent`, answered the child's review
    question itself 23 s after taking it up, and told it to stop the page;
  - ponderer then reported "presented and stopped" with `announce`;
  - its own attempt log shows it read the assessment, which was all `ok`;
  - no correction was requested, so this trial does not test the
    correction hypothesis.
- Verified from another LAN host (agpc): the fixture's review page while
  it served, and the delegations API with the assessment. Run view:
  [screenshot](run-view-correction.png); entry point
  `http://agstudio.local:8093/wfe/#/ws/agops/run/correction-check/run-001`
  (Detail → Delegations).
- Both review pages are stopped and verified gone.

## The hypothesis

The hypothesis was that evidence and a concrete correction at the parent's
receiving point improve delivery. One real trial cannot confirm it:

- the child presented without being asked, so the child-side changes (the
  brief's relay sentence, the wider coverage) are consistent with an
  improvement;
- the parent read the assessment, but the omission that happened was
  outside the bounded checks;
- the fixture shows that the parent has a working repair path and that a
  child follows a correction. It does not show that a real parent will
  choose that repair.

## Remaining limitations and decisions for the Developer

- **A parent can stand in for the person.** Under `accepts: parent`, the
  plan's review check does not apply, so a page reviewed only by the
  parent's agent and then stopped raises nothing. This is what happened in
  trial A. Two ways forward, which the plan left out of scope:
  - flag "reviewed only by an agent, stopped before the person saw it"
    whatever `accepts` says;
  - or have ponderer default its delegate nodes to `accepts: person`.
- `review-unrecorded` stays `unknown` for most `accepts: person` results
  relayed today (no `because`). It may teach parents `--yes`; fix5's
  accepted limitation applies.
- The signals are heuristics: viewable means a matching extension, and
  files are attributed by mtime. A remote child's files appear on the
  coordinator only after publication. Expectations stay prose.
- `announce` is assessed for delegated results only; fix5's own report
  checks were not added there.
- A pre-fix6 release cannot replay `delegation.revise`,
  `correction.receive`, `correction.take-up` or `presentation.retain`.

## Recovery references

The ignored `pj-agdev/.local/fix6/notes.md` holds the state before the
release, the store backup, the activate command for `506a116` with how to
find records that use the new operations, the agpc pin rollback and the
trial attempt ids. `pj-agdev/.local/devenv.md` has a fix6 section,
`pj-clusterintent/nctl/.local/fix6/` has the batch and playbook logs, and
`devdocs/README_DEV.md` has the fix6 entry.

Historical runs were not rewritten.
