# project_migration p5 — Step 4: workflows, guides, registration

Written 2026-10-16. `pj-cagent` is on Gitea and registered with WFE. Its five workflows validate, and their request channels are provisioned. **Definition approval is the Developer's and has not been recorded.** It is requested together with the Step 5 trial sheet. Host facts and channel ids are in the ignored notes (`pj-agdev/.local/project-migration-p5/step1.md`, *step 4*).

## The project

| Item | State |
| --- | --- |
| Repository | Gitea `developer/pj-cagent`, **private** (report1 §5: its records name hosts). `main` at `c033533`. |
| Registration | `wfe project register developer/pj-cagent`, then `wfe project obtain cagent`, give workspace `cagent`. Structure complete. The local areas `records`, `desired`, `agent` and `tools` are recognized. |
| Workflows | `consult`, `report`, `scan`, `change`, `freework`: 0 errors, 0 warnings, in a scratch registry and in the operating registry |
| Channels | Folder `wf-cagent`, with `wf-cagent.{change,consult,freework,report,scan}`, provisioned by the notifier |
| Intake | The Developer is a requester for every workflow. Front is an agent requester for `consult` and `change` (D13: Front's own guides are its episode's follow-up). Delegation needs no entry: the intake binds a delegated post by the delivery's message id. |
| Redaction | `io.secretFiles` now also lists nctl's token file, the Ansible vault password file and the Gitea token file: raw-value files, read as one secret once agdevworld `38ebdff` is deployed (Step 5) |
| Ignored configuration | The workspace `.local/cagent.toml` (approver, self-dependency host, pj-clusterintent path) and `localenv.md` (host roles). Both are copied into every run copy. |

## Workflows as built (design §4)

| Workflow | Nodes (effective graph) | Editable bindings | Notes |
| --- | --- | --- | --- |
| `consult` | `answer` (talk, `confirms: result`) | none | answers with sources and ages in `answer.md` |
| `report` | `record` → `verify` → `confirm` | `records` | verification by reads only |
| `scan` | `review` → `confirm` | `records` | `observe` + `snapshot` + scan record; never repairs |
| `change` | `_plan` → `execute` → `verify` → `confirm` | `records`, `desired` | plan step = registration and the agreement point |
| `freework` | `talk` → `_final-report` | project root only | `records` and `desired` explicitly readonly |

Every workflow uses `documentation: custom`, runs as `cagent-agent` on `sonnet` with `session: fresh`, and has the guides `agent/guide.md` and `agent/<workflow>.md`. The confirming talk is last, so no generated document follows the confirmation. That is the integration trap design §4 names.

**One amendment to design §4.** The compiler puts a generated `_plan` before every authored entry node, so a `register` node cannot precede it. Registration is therefore the plan step's own work:

- the change record, with request, targets, kind and the drift at registration;
- the batch documents and their previews;
- `plan.md` with its `cagent-plan` block.

The authority point is unchanged: nothing executes before the Developer agrees to that `plan.md`. This does not alter what the Developer agreed in D1–D14.

## Guides (role facts; usage in `--help`)

- **`agent/guide.md`** opens with the head: who cagent-agent is, whom it speaks with, what it can see (`cagentctl`, `records/`, `.local/localenv.md`, `wfe projects`). Its facts:
  - reported, observed and desired are three statements;
  - every observed fact has an age, and an unreachable host is unknown, not empty;
  - observing is not converging, and `maintain` is not autonomous convergence;
  - a change rests on the Developer's agreement to the exact plan, and a delegation, definition approval or acceptance is not that agreement;
  - destroy-class and self-dependency work is a person's, recorded only on their word and then verified;
  - the ledger, unknown outcomes, no repetition;
  - prerequisites are re-read before execution;
  - integration publishes, it does not authorize or undo;
  - recovery ownership; host facts and credentials.
- **The per-workflow guides** are each workflow's output contract:
  - `consult.md`: `answer.md`'s content, reads only, follow-ups;
  - `report.md`: provenance, duplicates and supersession, the verification vocabulary, "becoming managed is a change";
  - `scan.md`: per-host outcomes, drift before and after, report verification, and what `complete: false` means;
  - `change.md`: registration in the plan step, the Developer agreeing whoever asked, refusals as answers, long effects and the external wait, verification by observation, one-time vs maintain, cancellation, reversal.
- **Trial references.** Facts written after a seen failure keep their reference. `consult.md` cites pre1 report1 §7.3, the old cagent's wrong "reachable"/"converged" explanations.

**One tool addition.** The plan step must preview before any agreement exists. `cagentctl preview batch|reconcile` was added: dry, no ledger, and its test shows nothing is dispatched (27 tests pass).

## Access from a run copy

A disposable worktree at the run-copy location was used, with the workspace's `.local` files copied in as WFE does and the executor's launchd `PATH` with an otherwise empty environment. From there:

- `cagentctl read status` reached nctl and printed the nctl and pj-clusterintent revisions;
- `preview reconcile` produced a dry plan;
- `records check` read the records;
- `wfe` resolved the workspace. `observe` was refused `run-unreadable`, correctly, since there was no such run.

The tool's `.venv` is created in the copy and is ignored. The worktree was removed.

## Definition approval (pending)

The five definitions are presented for the Developer's approval in the editor, linked from the Step 5 request. That approval is a statement about the definitions only. The tool does not read it, and it does not stand for plan agreement or for agreeing to any live change.

## Acceptance

| Criterion | State |
| --- | --- |
| All five definitions validate | met |
| The registered project and its request channels resolve | met |
| Run copies have the intended records, tool and configuration access | met |
| Definition approval recorded as itself | outstanding: the Developer's |
