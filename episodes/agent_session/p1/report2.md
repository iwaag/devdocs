# Agent session p1 — step 2: the two session policies

Date: 2026-10-14. Source: agdevworld `e7fbe73`, agautolab `600448a`,
pyagag `4dde895` (not deployed yet; deployment is step 6).

## The setting

A workflow declares its session policy beside its other execution settings:

```yaml
execution:
  profile: sonnet          # unchanged
  guides: [agent/g.md]     # unchanged
  session: run             # fresh | run; not declared = fresh
```

| Policy | Behaviour |
| --- | --- |
| `fresh` | Every attempt starts a new harness session from the run's records and files (today's behaviour). On a session-capable profile the new session still gets an explicit id, so the record names it. |
| `run` | Later attempts resume the run's retained session by its id. A changed configuration, a retirement or a missing conversation starts a new one, with the reason recorded (step 3). |

- **Not declared = `fresh`.** This keeps the comparison baseline; it is not a
  compatibility layer. The final default is chosen from step 5's evidence.
- **Validation.** Any other value is an error (`execution-session`), so a run
  of that workflow is refused at creation.
- **Approval.** The policy is part of the definition digest when declared;
  workflows that declare none keep their digest.
- **Kept apart.** The policy is separate from the profile, the node types and
  the documentation policy. No per-node heuristics.

## Where it travels

| Boundary | What happens |
| --- | --- |
| Editor (Detail) | Workflow settings has a new **Execution** section. It shows the agent, profile and guides as declared, plus a *Session policy* select (`fresh (not declared)`, `fresh`, `run`) with a one-line explanation. Simple does not show it: the settings panel is Detail-only. |
| File | `execution.session` is written on save and removed when set back to "not declared"; comments and other fields are kept. |
| Run creation | The definition is frozen byte for byte into the run's bundle, so the policy is frozen with it. A later edit reaches future runs only. |
| Claim (`wfe exec claim`) | The policy is read from the frozen definition. The host says which profiles keep explicit sessions (`--sessions-json {"sonnet": "explicit", "": "explicit"}`; `''` is the host default). The resolved session (`policy`, `mode`, `id`, `replaces`) is recorded in `attempt.begin` `settings.session` and returned to the executor. |
| Executor | `agautolab.wfexec` offers the capability from `agag.harness.SESSION_HARNESSES`. Only `claude_code` is in it. |
| CLI / API | `wfe run session <run>` prints the frozen policy and the sessions. The API's run response carries the frozen bundle (with `execution.session`) and `record.sessions`. |
| Run view | The execution section states the effective policy and the current session (short id, number of attempts), and why earlier ones were replaced or retired. Each attempt shows its session (new or resumed). |

## Unsupported profiles

When a run with `run` reaches a profile whose harness keeps no resumable
conversation, the claim **drops the job** with the reason, as it does for a
profile the host does not offer. The reason reads: *the workflow's session
policy is "run", but profile X (backend) keeps no conversation this host can
resume; choose fresh, or a profile that supports sessions.* Nothing begins.
The run is never executed fresh in place of `run`.

`fresh` on such a profile runs as before, with no session id.

## Evidence

- **WFE `npm run check`: 306/306** tests pass, including the new
  `test/sessions.test.ts`; type check and build pass. The tests cover: parse
  and write-back, save/remove in an existing file, validation, the digest, the
  frozen policy at claim, refusal on an incapable profile, fresh replacement,
  and resume.
- **agautolab `tests/test_wfexec.py`: 13/13** with the real WFE CLI and a
  session-keeping stub. The new tests cover run, fresh, refused, missing,
  changed guide plus retire, and early crash with restart.
- **pyagag: 1218/1218** pass, including the new seam tests.
- **Browser check `checks/sessions.ts`** (private project, registry and
  service; Chromium) — all passed:
  - Detail shows the Execution section and its facts. An undeclared policy
    reads "fresh (not declared)".
  - Choosing `run` and saving writes `session: run` into the file, and the
    choice survives a reload.
  - Simple does not show the control.
  - A run created under `run` stays `run` after the workflow is edited back
    to `fresh`. Its claim carries policy `run`, mode `new` and a minted id.
  - `attempt.begin`, `wfe run session --json` and the API run response name
    the same session.
  - The run view states "Session policy: run … Current: session 7e74268e,
    1 attempt".
  - No console errors.

  Screenshots are in the ignored `agdevworld/.local/wfe-checks/screenshots/`
  (`sessions-detail-settings.png`, `sessions-run-view.png`).
