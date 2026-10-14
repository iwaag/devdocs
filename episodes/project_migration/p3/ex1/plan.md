# project_migration p3 ex1 — clear final approval and close remaining work

## Goal and scope

Basis: [brakindump.md](brakindump.md), [../report.md](../report.md), [../report4.md](../report4.md), and the p3 review.

Make final approval understandable and responsive, prevent duplicate acceptance/integration, correct forge's misleading records, and finish the old forge's Nautobot retirement. Keep runtime testing minimal. Delegate, shared cluster context, Front routing and unrelated forge improvements remain outside this episode.

## Step 1 — correct approval state and interaction

- Inspect the current implementation and p3 evidence before changing it. Start with WFE's `src/runView.ts`, `src/runSide.ts`, `src/integrationState.ts`, and the corresponding decision/integration operations. Coordinate with any concurrent editor changes; preserve unrelated work.
- Derive the header and action area from the same current state. Distinguish an unanswered confirmation, an answer awaiting agent processing, accepted work awaiting integration, a changed result needing renewed acceptance, and integrated work. When renewed acceptance is needed, show that as the primary status rather than an unqualified `Accepted`.
- Put the appropriate answer or renewed-acceptance control next to that status. Show the result/evidence being accepted; do not require discovery through *All records*. Preserve the distinction between answering a question and accepting a changed result.
- Trace the reported unresponsive input through submission, API response and refresh. Show pending, success and error feedback; retain entered text on failure. A recorded answer waiting for the agent must be visibly different from a failed submission.
- Disable repeated submission while a request is in flight and remove obsolete acceptance controls. Enforce duplicate protection at the authoritative operation as well, including confirmation-answer and direct-acceptance paths: the same result must not create another acceptance or integration. Changed results must still permit legitimate renewed acceptance. Do not rewrite historical decisions.
- Make integration status use the relevant latest decision/attempt, including `integrate --status`, so stale rows cannot contradict a successful integration.

Deliverable: one coherent approval path with explicit feedback and correct current status.

## Step 2 — correct forge's records and guidance

- Correct `requests/provide-run-003.md`: the outcome is partial, and acted speech remains an unmet demand. Clearly distinguish the original lack of any speech tool from the current remaining limitation. Preserve origin and follow-up links.
- Add an explicit correction to develop-capability/run-001's verification evidence: automated format/padding checks do not establish a listening verdict. Cite a person's listening assessment only if an actual record supports it. Review the speech catalog's proof for the same unsupported claim.
- Add the corresponding brief guide facts: demand state follows the actual outcome; listening judgments require attributable evidence. Do not turn this into a broad guide rewrite.
- Keep accepted run history and decisions intact. Publish corrections as new, reviewable changes through the supported project workflow/integration path; do not alter old acceptance hashes or pretend the corrections were part of the original result.

Deliverable: consistent demand and verification records, with the correction provenance visible.

## Step 3 — finish retirement and release

- Check current desired/actual state through Nautobot or `nctl`. Re-read and preview the prepared retirement batch; it should retire only old agforge's service/agent and disable its placement. Apply only still-needed changes and verify the resulting desired state. Preserve the checkout/venv used by comfynotify, historical records and stored assets.
- The p3 report records a previous approval rejection. Do not bypass it. If the operation is still denied, retain the validated concrete batch and report the exact rejected action and reason; leave retirement explicitly incomplete rather than claiming success.
- Deploy the editor fix using the existing release procedure. Prepare a directly accessible affected run or isolated example before asking the person to inspect the UI. Services remain reachable from the LAN per `localrule.md`.
- Update p3's completion/remaining-work notes and write a short ex1 `report.md` with changes, checks, release, record corrections and retirement status. Commit and push the changed repositories and any required superproject pointers.

## Minimal verification

- Run focused regression checks for: (1) renewed acceptance shown with its action; (2) pending/failed/successful answer feedback; (3) duplicate acceptance creates no second decision/integration while a changed result can be reaccepted; (4) the latest successful integration supersedes an older failure in displayed/CLI status. Reuse existing fixtures and tests where possible.
- Run required type/build/release checks once. Broaden testing only for a failure or a changed dependency; do not repeat the media-generation trials or start paid agent runs merely to test UI state.
- Perform one short browser smoke check of the deployed status/action/feedback path using isolated data when a write is needed. Do not reaccept a real completed result just to exercise the control.
- Read back the corrected demand/evidence and Nautobot retirement state. Inspect the intended diff for host details and secrets before publishing.

Completion: the person can see whether and where approval is needed, receives visible submission feedback, cannot accidentally accept the same result twice, and sees the latest integration outcome. Forge's records no longer claim complete provision or unsupported listening evidence. Old forge's retirement is verified, or an actual approval block is clearly recorded as the sole remaining task.
