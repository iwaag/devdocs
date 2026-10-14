# Agent session p1 — step 6: verification and deployment

Date: 2026-10-14.

## Tests

| Suite | Result |
| --- | --- |
| WFE `npm run check` (types, every test, build) | 306/306 before deploy; the release build re-ran it (`wfe-release.sh` refuses on failure) |
| WFE `test/sessions.test.ts` | policy model/validation/digest, frozen claim, unsupported profile, fresh replacement, resume, configuration change, missing (idempotent), retire, stale attempt, reducer mismatch, observation during wait and resume |
| Browser `checks/sessions.ts` (private service, Chromium) | all passed: Detail control, save/reload, Simple hides it, frozen run, claim/CLI/API agree, run view |
| agautolab (all) | 372 passed, 1 skipped (`test_wfexec.py` 13: run/fresh/refused/missing/changed guide+retire/early crash with restart) |
| pyagag (all) | 1218 passed |
| agobserver (all) | 107 passed (recorded session in evidence and recovery request) |
| Real harness trials | steps 3–5: 3 + 6 + 4 runs, about $17.6 reported in total (step 3 ≈ $1.19, step 4 ≈ $2.3, step 5 $14.06; corrected in ex1 from "~$15", which carried report5's addition error) |

## Deployment

Before activation:

- `nctl status` was healthy and `wfe observe` showed an idle executor.
- The only run was the person's held `jevlab/experiment/run-001`
  (`stopped`). It was not touched, and after deployment it is still stopped
  with policy `fresh` and no session.
- The stores were backed up to
  `.local/agdev/.local/backups/agent-session-p1-*`.

| Component | Version | Activation |
| --- | --- | --- |
| WFE (agdevworld) | `dab1e54` (was `314c98b`) | `wfe-release.sh deploy`: checked, healthy; service and `bin/wfe` agree |
| executor (agautolab) | `c08cb5d` + pyagag `4dde895` | `com.agdev.agautolab-wfexec` kickstarted; it offers `--sessions-json` |
| Observer (pj-agdev `agobserver`) | `d1ce1d5` + lock at pyagag `4dde895` | `com.agdev.agobserver` kickstarted; looking every 60 s |

No schema transition was needed: `sessions[]`, `attempt.session` and
`session.retire` are additive, and exec.sqlite is unchanged. The editor
answers on the LAN (`http://agstudio.local:8093/wfe/`, HTTP 200).

## Deployed multi-attempt check

New disposable workflow `agops/session-check` (`session: run`, talk → do,
root read-only; Gitea `developer/pj-agops`). Run `session-check/run-001`,
requested as "Omni Agent (deployment check)":

- Attempt 1 created session **6cc43778**, recorded before launch and attached
  from the harness. It asked for a word and ended; the slot was released.
- The answer ("lantern (deployment check input)") queued attempt 2. It
  **resumed 6cc43778**: its first request wrote 1.3 k tokens and read 43.2 k.
  It recorded the word and completed.
- Reported cost: $0.136 + $0.106.
- Browser, deployed editor over the LAN: Detail → Workflow settings shows
  *Session policy: run*. The run view shows "Session policy: run — later
  attempts resume the run's session. Current: session 6cc43778, 2 attempts"
  and each attempt's session (new / resumed). Screenshots are in the ignored
  `pj-agdev/.local/agent-session-p1/step6-*.png`.

## Found, not fixed (outside this phase)

Opened over plain LAN HTTP, the editor is not a secure context. Workflow pages
therefore throw `TypeError … reading 'digest'`, because `crypto.subtle` is
unavailable for the approval digests. This happens on the unchanged
`agops/check.yaml` too, so it predates this phase. It has been the case since
the LAN exposure (`783cfcb`). It does not occur on `127.0.0.1`.
