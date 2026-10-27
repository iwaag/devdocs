# tidy p2 — final report

Date: 2026-10-27 (JST). Steps:
- `report1.md`: boundaries and baseline;
- `report2.md`: implementation and deployment;
- `report3.md`: trials;
- `report4.md`: verification and remaining gaps.

Machine-specific notes, logs and rollback commands are in the ignored `pj-agdev/.local/tidy-p2/notes.md`
and in `pj-agdev/.local/devenv.md` *tidy p2 on this host*.

## What exists now

- **Receiver evidence.** Where a delegated result is received (the brief, `wfe delegate take-up|show`, the
  HTTP view), the assessment shows, in order:
  - the findings first (`FINDINGS (n)`, or "none in what WFE checks; the content is not checked");
  - the correction the result answers ("an earlier judgement does not cover it");
  - the child's **claim** (outcome, report) beside its **records** (files changed and removed, outputs,
    report paths not in its copy);
  - the checks;
  - `to judge:` questions for what WFE cannot observe.

  The questions are chosen from the child's work: documentation, delivery, removals and moves, tests. A
  path or a `tidy` node chooses a question, never a finding.
- **Producer advice.** Printed once, with `wfe run start` of the final-report step or with the completion
  that leaves only reporting. It asks:
  - what changed and at which commit, which checks ran with what results, and what is left;
  - whether the entry, guide and setup still match. A tidy run is asked for evidence instead of another
    pass.

  The questions that fit the run's own changes follow. It refuses and records nothing. A delegated
  child's brief says how it will be assessed.
- **Multi-child account.**
  - `conversation reply` lists the commissioned results with news since the run's previous post.
  - New reminder `results-unreported` at a standing conductor's rest. It catches what
    `conversation-unanswered` misses once a reply exists. It clears when the run speaks.
- **Rest checks in HTTP.** `discussion-open`, `conversation-unanswered` and `results-unreported` refuse
  `POST …/ops` as they do the CLI.
- **One authority for the meaning.** devpolicy `styles.md` *Reporting and accepting results* (`434657a`).
  Catalog `tidy` r2 names the acquisition check of adopted deliveries.
- **Unchanged:**
  - `document-missing`, the `delegated-*` reminders and `--yes`;
  - optional documentation;
  - frozen catalog snapshots;
  - the wfrun guide.

  No new state, schema, gate or approval.

## Revisions (published)

| Repository | Revisions |
| --- | --- |
| agdevworld | `828fd47`, `538e96d`, `115872b`, `17c52b3` (pre-p2 `2cc9cd9`); studio and agpc run `17c52b3` |
| devpolicy | `434657a` (Gitea mirror synced) |
| pj-agops | `0a0e08c`, `c1d84dc` (fixture workflow `evidence-fixture`) |
| agsfx (by its run) | `89000eb0a6` |
| protoprey (by its run) | `c4db96ffb5` |
| devdocs | this episode, README_DEV *Reporting and assessing results*, p1 dispositions P2 and P4 resolved |

Tests: new `test/resultEvidence.test.ts`, five new tests in `test/assessment.test.ts` and an HTTP case in
`test/activities.test.ts`. The release suite passed 635/635 on each of the four deploys.

## Trials

| Trial | What happened | Interventions |
| --- | --- | --- |
| A, real (`tidy-p2-trial`, conduct/run-049) | d1: agsfx published r3 and described its offer. d2 (dispatched after d1): protoprey re-adopted it by share reference and verified a fresh clone. Both were judged with recorded checks. The summary #458 named both and their limits. Omni then installed from a fresh clone with no wfe and ran 8 of 8 tests. **p1's open finding is resolved** | none |
| B, fixture (`tidy-p2-fixture`, run-050) | Host paths, "4 claimed / 3 ran" and false portability claims were found unprompted by running the tests; the sound result was accepted. ponderer offered a correction and asked first | none. The fixture opted out of delegation (my error), so ponderer used `wfe request` |
| C, fixture, fresh context (`tidy-p2-fixture2`, run-051) | The same gaps were found unprompted (#463). After a go-ahead ponderer sent a precise correction (d1 → run-006). The **partial** correction was judged satisfied in 19 s from the report ("Tests not re-run"), an **independent failure**. Within a minute ponderer re-read the test, found it unchanged and returned it to the same child (`revise`). It verified the final correction. Summary #466 named both | #464: a go-ahead with no defect information |

## Assessment against the completion criteria

- **Boundaries deliver concise, contextual guidance through normal paths.** Met. Seen live: the producer
  advice in run-004 to run-006 and the receiver evidence at every take-up.
- **Facts are distinguished from judgement questions; optional documentation and `--yes` are intact.**
  Met.
- **An unprompted receiver detects a gap, returns it to its owner and verifies the correction, without
  rejecting everything.** Partly met:
  - detection was independent three times, and sound results were accepted four times;
  - one correction was returned to the same child unprompted and its follow-up was verified;
  - the first correction was sent only after a go-ahead;
  - one premature acceptance happened, though ponderer reversed it itself.
- **Multi-project summaries name each outcome and its limits.** Met in A and B. C's last summary dropped
  one limit and the intermediate failed correction.
- **The trial output is usable and verified outside the producer's copy.** Met (fresh clone, share
  reference, hashes).
- **Tests and release checks pass, deployments are updated, no tidy loop.** Met.

The behavioural objective is **not claimed as achieved in general**:
- one real trial and two fixture trials;
- one intervention;
- one independent premature acceptance.

## Remaining limits

1. ponderer asked before sending corrections (B, C). It may read "evaluate and summarize" as scope.
2. The premature acceptance answered the tests question from the report's own output. `17c52b3` now says
   the output is the child's claim and asks for the receiver's own run or an explicit "unverified". This
   has **not been re-tried**.
3. Requests made with `wfe request` + `ask` (copied from a previous run as precedent) get no evidence
   prompts and no `revise`.
4. Reopening a judged result must cite the person's words. Correction 1 was ponderer's own finding but is
   recorded "from Omni Agent (for Developer)", and `review-unrecorded` then reads ok.
5. A hand-operated run with a final-report step produces two result events. Real runs freeze facts at the
   attempt's end.
6. Out of scope and still open from p1: agops' and cagent's root records, projects without README,
   lowerpoly's toolchain-host constraint.

## Recovery

No store or schema change. `wfe-release.sh activate .local/agdev 2cc9cd9…`; agpc batch back to `2cc9cd9…`
(`nctl desired apply`, `render production`, `setup_wfe_worker.yml -l agpc`). Catalog: `wfe nodes update tidy
--expect-revision 2` with r1's text. The fixture workflow can stay. Its runs were never integrated.

## For the Developer to look at

- Rooms:
  - `http://agstudio.local:8093/hangout/?conv=tidy-p2-trial`: the real case, no intervention;
  - `?conv=tidy-p2-fixture2`: the fixture with its correction cycle; #464 is the one intervention.
- A receiver message: `wfe delegate show conduct/run-051 d1 --workspace ponderer` (findings, the
  correction line, claim and records, `to judge:`). In the run view it is under the delegation of
  ponderer/conduct/run-051.
- A producer message, as run-005 got it at `start _final-report`:

  ```
  Before the result goes back (advice; nothing is refused): ponderer-agent compares what you report with your files and records.
    · Say which files and records changed and at which commit, which checks actually ran and with what results, and what is unverified, deliberately unchanged or left open.
    · The report (devdocs/runs/evidence-fixture/run-005/report.md) and every path it names are read where you say they are.
    · Do the project's entry (README), guide and setup or acquisition instructions still match this change? …
    · Documentation: … are one machine's facts (hosts, home paths, LAN addresses) out of portable text, …?
    · Tests: did the new and changed tests actually run (listed by the runner, not only present in the source), and do the counts you report include them?
    (chosen from this run's changes: …/README.md, …/test_linecount.py; a path chooses a question, it shows no defect)
  ```
- protoprey with the portable effects: main `c4db96f`. `PROTOPREY_SHARE_ROOTS=agshare=<share>` then
  `tools/deliveries.py install`.

## Dispositions of the remaining limits (added by tidy p3, 2026-10-27)

The limits above are kept as they were written. What became of each, with the evidence:

1. Asking before returning: **not seen again.** In seven tidy p3 trials with p2's implicit wording, ponderer returned
   every deficient result without asking (`../p3/report5.md`).
2. The sharpened tests question: **re-tried in a fresh context** (`../p3/report1.md`); no premature acceptance there
   or in the trials. Not proof of reliability.
3. `wfe request` + `ask`: **resolved in code and tried once** (`../p3/report3.md`, trial T1 in `../p3/report5.md`).
4. Reopening needs the person's words: **resolved in code** (`--reopen --evidence`, `../p3/report2.md`); correction 1
   of run-051 d1 is attributed to ponderer-agent and its review status reads UNKNOWN. An agent reopening its own
   mistaken acceptance was **not observed** behaviourally.
5. Two result events from a hand-operated run: **fixture changed** (agops `2a1638d`).
6. Open p1 items: unchanged (out of scope).
