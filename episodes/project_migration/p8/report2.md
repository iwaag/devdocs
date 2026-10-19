# p8 Step 2 — common operator capabilities

Source: agdevworld `9d71240`, `d5f467e` (WFE). Not deployed in this step; the
release follows in Step 5 together with the conversation facility.

## What changed

| Gap (report1) | Change |
| --- | --- |
| G1 definition writes had no agent verb and no server-side stale check | `wfe workflow show <wf>` / `wfe project show` print the file with its revision; `wfe workflow save <wf> --file … --base <rev>` / `wfe project save …` write it through the editor's own save. The editor's PUT routes take the same `base`; the views send the revision their draft was made from. A file that moved on is refused with `definition-conflict` and its current revision, and nothing is overwritten. |
| G2 no queued agent-authored request | `wfe request <wf> --request <file> [--on-behalf-of <person>] [--original <file>]`. From an executing attempt: the requester is that run's agent; it is entrusted by that run (`entrustedBy: {kind: run, project, workflow, run}`); the receipt is derived from the attempt and the text, so a retried command finds the same run. `ExecStore.request` takes the request input. |
| G3 acting on another run from inside an attempt | The CLI recognizes an open attempt of a different run as an **operator**. It records as that run's agent (`by`), never takes the target's fencing, and names itself in the entry (`operator: {run, attempt}`). Covers instruct, answer, stop, resume, cancel and decide. `--by` with another name is refused; an ended attempt records nothing. |
| G4 an instruction never woke a waiting run | `ExecStore.afterInstruction` queues a continuation (`instruction:<ws>/<wf>/<run>/<id>`) for a run with no open attempt, no hold, not completed and not cancelled. Called from `wfe run instruct`, the run view's ops route and the Zulip intake. A completed run keeps an instruction for an attempt that reopens work (an existing tested behavior), but nothing continues it; the CLI says to start a successor. The contract's claim that a completed run refuses instructions was corrected to the actual behavior. |
| G5 delegate node added by adjustment untested | A test covers the whole path: adjustment with `--completing`, dispatch, the child's question, the parent's answer, the result confirmation, the child's decision and completion, the parent's disposition, and the grown run's completion. It passed without a code change: dynamic delegation already worked. |
| G6 raw ops route skipped domain checks | `delegation.*`, `run.assign` and `session.retire` are no longer offered on `POST …/ops`. They are made by `wfe delegate` and `wfe run assign|session`. |

Retries and identities:
- run requests dedupe on their receipt (person: the browser's or conversation's receipt; agent: attempt + text);
- run operations dedupe on `--receipt`;
- delegation dispatch dedupes on its derived receipt;
- `wfe create` refuses an existing non-empty folder and continues only with `--resume`, so a retried creation cannot make a second project.

Live versus published state is unchanged from p4/p7: `wfe projects run` shows the live record beside the published snapshot, also for runs on another worker.

## Operation matrix (completed)

| Area | Browser | Agent (CLI) | Same domain operation |
| --- | --- | --- | --- |
| Discovery and reads | Home, project, workflow, run, context views | `wfe projects …`, `wfe run show/state/io/events`, `wfe delegate show`, `wfe context …`, `wfe observe`, `wfe exec workers`, `wfe boast status` | yes |
| Project creation, registration, workspaces, shared repositories, submodules | Home, project view | `wfe create`, `register`, `project register/obtain`, `repo …`, `add-repo` | yes |
| Project configuration (intent, goals, guides, execution) | project settings (Save) | `wfe project show/save --base` | yes, with the same base check |
| Workflow creation | Create workflow | `wfe workflow new`, `workflow basics` | yes |
| Workflow editing (nodes, edges, execution settings) | workflow editor (Save) | `wfe workflow show/save --base`, `arrange`, `nodes instantiate` | yes, with the same base check |
| Approval records | workflow editor | `wfe approve` | yes |
| Requests | project view (a person's words) | `wfe request --braindump` (a person's words), `--request` (an agent's) | yes |
| Instructions, answers, questions | run card, side panel | `wfe run instruct/answer/ask/take-up/withdraw/ack` | yes; the instruction continuation is shared |
| Run revision | Adjust future work | `wfe run adjust [--completing] [--propose]` | yes |
| Lifecycle | stop, cancel, resume | `wfe run stop/cancel/resume/recover/assign/session` | yes |
| Delegation | read-only view | `wfe delegate request/answer/instruct/dispose/cancel` | agent-only by design |
| Context | Context sections | `wfe context set/remove/check/… --base` | yes |
| Execution settings | workflow editor (Execution) | workflow save; `wfe run resume --profile`; `wfe run assign` | yes |
| Result review and decisions | side panel accept/reject | `wfe run decide` | yes |
| Integration, publication | run view | `wfe run integrate`, `wfe exec publish` | yes |
| Presentations, shares | run view | `wfe present …`, `wfe run input/publish/inputs/outputs --verify`, `wfe share …` | yes |

No browser operation lacks an agent tool. Committing a saved definition is
Git work for both: runs read definitions from the workspace.

## Tests

- New `test/operator.test.ts`, 5 tests:
  - the dynamic delegate lifecycle;
  - instruction continuation (once; none for held and completed runs);
  - operator provenance through the real CLI (instruct, answer, stop, `--by` refused, ended attempt refused);
  - an agent request through the real CLI, retried (one run; requester, person served, original words);
  - a stale workflow save, a stale project PUT, and the raw ops refusals.
- `test/intakestore.test.ts`: one expectation updated. An earlier post's instruction now continues the waiting run, and the answer joins that one job.
- WFE `npm run check`: 408/408, type check and build pass.
