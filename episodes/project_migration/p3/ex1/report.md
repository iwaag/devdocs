# project_migration p3 ex1 — final report

Basis: [brakindump.md](brakindump.md), [plan.md](plan.md). Step reports: [report1](report1.md), [report2](report2.md), [report3](report3.md).

| | State |
| --- | --- |
| agdevworld (editor) | `e1e6be3`, released (previous `75cdae7`) |
| pj-forge | `e9ea373` on `main` (guide facts). The record corrections are on `wfe/forge/freework/run-001`, waiting for the Developer's decision. |
| pj-agdev | agdevworld pointer → `e1e6be3` |
| Nautobot (agforge) | **not retired**: apply refused to the Omni Agent; validated batch kept |
| Agent cost | one forge `freework` run (sonnet) |

## Changes

- **Approval path** (step 1). The header and the focus card read one standing of the result. That standing is the latest decision's, and `run.json`'s integration record outranks the journal. When a result has changed since its acceptance, the header says *Changed since acceptance — accept again*. The focus card lists the changed files and offers *Accept the changed result* beside them.
- **Feedback.** Every form says *Recording…*, then what was recorded or why not, next to its button. The text is kept on failure. A recorded answer reads *Recorded … waiting for <agent>*, which looks different from a failure.
- **Duplicate protection.** The server refuses an acceptance of a result the latest acceptance already binds (`already-accepted`). This covers the direct path and the confirmation-answer path. A changed result is still accepted again. The executor's queue re-reads each row and marks older rows `superseded`; in p3 a stale row overwrote an integrated one. `integrate --status` prints the current standing first.
- **Forge's records** (step 2):
  - Two guide facts were added: demand state follows the outcome, and listening judgments need an attributable record.
  - forge-agent corrected `requests/provide-run-003.md` to `partly-served` (acted speech is still unmet, apart from the original gap), corrected `catalog/speech.md`, and wrote `correction.md`. That file states that no one listened and that format checks are not a verdict. develop-capability/run-001's accepted folder is unchanged.

## Checks

- **Tests:** `test/approvalstate.test.ts` covers plan regressions 1, 3 and 4. `npm run check` passed: 311/311 tests, type check, build.
- **Browser:** `checks/approval.ts` covers regression 2 and the UI side of 1 and 3, on isolated data: 17/17.
- **Release:** the release's own check passed. A read-only LAN smoke check of the deployed run view also passed.
- **Real records:** `integrate --status` was read back on the real forge records. Read back too: the correction diff (only the intended files, no host details in them) and Nautobot's desired state before the attempt.

## What the Developer needs to do

1. **Decide on the correction:** `http://agstudio.local:8093/wfe/#/ws/forge/run/freework/run-001`. Read *Read final report* and `correction.md` in *Reports & files*, enter your name and evidence, and press *Accept result*. The new feedback appears next to the button, and the header moves from *Accepted · integrating* to *Accepted · integrated* within about a minute. If you did listen to the speech lines, saying so in the evidence is the attributable record the catalog lacks.
2. **Finish agforge's retirement:** `cd pj-clusterintent && uv run --project nctl nctl desired apply -f ../pj-agdev/.local/project-migration-p3/retire-agforge.yaml --yes` (3 updates, 0 conflicts in today's preview).

Until (1), `main` still carries the old demand state and the "intelligible" wording. Until (2), old agforge's retirement is incomplete, and that is its only remaining task.

## Notes

- forge-agent's `review` outcome says "Developer confirmed". The Omni Agent answered that question, as the record shows. The review URL also still appears in `run.json` (p3 finding 6, open).
- Deus Ex Machina: the Omni Agent wrote the two guide facts; the record corrections were forge-agent's.
