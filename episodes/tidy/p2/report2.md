# tidy p2 — Step 2 report: guidance implemented, tested and deployed

Date: 2026-10-27 (JST). Releases: agdevworld `828fd47`, then `538e96d` (a report-path filter fix). Before:
`2cc9cd9`. devpolicy `434657a`. Catalog `tidy` r2. Machine details and rollback commands are in the ignored
`pj-agdev/.local/tidy-p2/notes.md`.

## What changed

| Boundary | Change | Code |
| --- | --- | --- |
| Receiver: brief, `delegate take-up`, `delegate show`, HTTP `…/delegations` | the assessment starts with a findings headline (`FINDINGS (n)` with non-ok checks listed first, or `findings: none in what WFE checks …; the content is not checked`). Then: the correction a result answers; `claim —` (outcome, report path, hash, commit); `records —` (files changed and removed, artifacts, published outputs, report paths not in the child's copy); the checks; and `to judge:` questions | `shared/assessment.ts`, `shared/resultEvidence.ts`, `server/delegate.ts` (`namedPaths`) |
| Producer: `wfe run start <final-report step>`, or the completion after which only reporting is left | advice printed with the ordinary output, once; refuses and records nothing | `server/resultEvidence.ts`, `cli/run.ts` |
| Producer: a delegated child's brief | one paragraph: the result is assessed beside the child's files where its requester receives it | `server/exec.ts` |
| Conductor: `conversation reply` (any kind except progress) from a run with delegations | the output lists the commissioned results with news (result, judgement, correction) since the run's previous post | `cli/conversation.ts` |
| Conductor: rest of a standing conductor | new reminder `results-unreported`: a delegation's latest result fact is later than the run's last post in the room. It clears when the run speaks | `server/restChecks.ts` |
| Rest checks in HTTP | `discussion-open`, `conversation-unanswered` and `results-unreported` now also refuse `POST …/ops` (before, the CLI only) | `server/api.ts` |
| Shared principle | *Reporting and accepting results* (claim and evidence; check before accepting; corrected results checked anew; summaries account for each; a reminder is no proof) | devpolicy `styles.md` |
| Tidy template r2 | step 3 adds "whether a fresh copy can obtain each adopted delivery (run the acquisition or status check: the adoption record alone does not show it)". Step 5 asks for files and commit, and checks with results | live catalog |

The contextual questions (documentation, delivery, removals and moves, tests) are chosen from template
provenance (`tidy`), changed paths (entry or guide files, tests, delivery or adoption manifests,
`.gitignore`), removed paths and outputs. A path chooses a question, never a finding. Each list says what
chose it. With nothing chosen, only the general questions appear. No classifier, no structured form, no
new state, no schema change.

**Not changed:**
- `document-missing` and the `delegated-*` reminders, with `--yes` per command;
- optional documentation;
- frozen catalog snapshots (runs created before r2 keep r1);
- the wfrun guide. Its delegation bullet stays true, the new output carries the rest, and no executor
  release was needed.

## Tests

New `test/resultEvidence.test.ts` and five new tests in `test/assessment.test.ts`, plus an HTTP case in
`test/activities.test.ts`. They cover:

- **Boundary delivery:**
  - advice at final-report start;
  - advice at the completion that leaves only reporting;
  - no advice while work remains, at a reporting node itself, or once the run completes.
- **Known missing facts:** `FINDINGS (1): MISSING document-missing` comes before `ok` lines.
- **Uncertain applicability and unrelated small work:** a `src/main.py` change gets only the general
  questions, and no reminder.
- **Revised results:** the correction line appears on event 2, not on the returned event 1.
- **Optional documentation:** the producer advice never refuses, and the questions add no reminder.
- **No loops:**
  - `results-unreported` clears after a reply;
  - a progress note does not clear it;
  - another run's post does not clear it;
  - it is silent while not at rest or with no delegations.
- **run-048 replayed:** at the state before #453 only `results-unreported` fires
  (`conversation-unanswered` cannot: #452 replied); at the first rest both fire.
- **Override and paths:** HTTP `POST …/ops` refuses with `conversation-unanswered` exactly as the CLI
  does; `--yes` still passes.

The release suite passed: 635/635 in both deploys (`npm run check` inside `wfe-release.sh`).

## Deployment

- studio: `bin/wfe version` names `538e96d`; the service is healthy.
- agpc: placement `agautolab-agpc` pins wfe `538e96d` (executor `eeddf95` unchanged). The playbook ran
  ok=22, changed=3. `~/wfe-worker/src/agdevworld` is at `538e96d` and `wfe-worker` is active.

## The rendered messages on live records

`wfe delegate show conduct/run-048 d3` (lowerpoly's corrected tidy) now prints:

```
findings: none in what WFE checks (presentation, the person's review, a serving page, documents); the content is not checked
This result follows correction request 1 (it returned event 1): "Rewrite the last README section …". The judgement of event 1 (satisfied) and any check of event 1 does not cover it: check the correction itself.
claim — outcome: "README's last section rewritten portable (…)"
claim — report: devdocs/runs/freework/run-004/report.md (856ca371d981, 1445 bytes, in 8d1bf5278d)
records — changed outside its run folder (1): README.md
records — published outputs: none
records — the report names … paths that are not in the child's copy: … (moved on purpose, elsewhere, or missing: look)
ok presentation-missing … / ok presentation-serving … / ok document-missing …
to judge: WFE checked records, not content. Compare the claim with the files before judging (wfe delegate file conduct/run-048 d3 <path>).
  · Documentation: does the entry agree with the implementation …? … Are one machine's facts (hosts, home paths, LAN addresses) out of portable text, and the portable requirement kept?
  · (chosen from the child's changes: a tidy node, README.md; a path chooses a question, it shows no defect)
  · What is missing goes back to the same child (wfe delegate revise …), and its next result is checked again; …
```

The first deploy's rendering listed `__pycache__` and `run-001..003/report.md` as missing report paths.
`538e96d` skips folder names written with a trailing slash and ranges.

What protoprey run-006 would have seen at `start _final-report` (its real changed paths):

```
Before the result goes back (advice; nothing is refused): ponderer-agent compares what you report with your files and records.
  · Say which files and records changed and at which commit, which checks actually ran and with what results, and what is unverified, deliberately unchanged or left open.
  · The report (devdocs/runs/freework/run-006/report.md) and every path it names are read where you say they are.
  · This run tidied: report its evidence rather than tidy again.
  · Documentation: …
  · Removals and moves: can what you removed be regenerated, are original assets and evidence kept, and are references and consumers updated?
  (chosen from this run's changes: a tidy node, agent/guide.md, GOAL.md, report.md (removed); …)
```

run-006 never changed `data/deliveries.json`, so the delivery question would not have been chosen for it.
That gap is why tidy r2's step 3 names the acquisition check.

## Recovery

There is no store or schema change. Rollback is `wfe-release.sh activate .local/agdev 2cc9cd9…` plus the
agpc batch back to `2cc9cd9…` (the same three commands). Catalog: `wfe nodes update tidy
--expect-revision 2` with r1's text.

## Next

Step 3: one unprompted ponderer-led trial on the real agsfx → protoprey case.
