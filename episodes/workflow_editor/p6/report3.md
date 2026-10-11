# p6 step 3 — project-based execution settings

Plan: [plan.md](plan.md) §Step 3.

## What changed

Commits: agdevworld `a3fed5d` (`workflow_editor/`) and agautolab `efc0712`.

### Declarative settings in the workflow (editor)

| Field | Meaning | Where it acts |
| --- | --- | --- |
| `execution.agent` | the identity runs are performed as | run creation: `executor: {name: <agent>, host: <execution host>}` |
| `execution.profile` | one of the execution host's profiles | claim: picks the backend; launch: the harness profile |
| `execution.guides` | project-relative guides added after the common contract | claim: read from the workspace, hashed and handed to the host |
| `intake.zulip: requests` | the workflow's channel accepts request topics | step 4 |
| node `confirms: result` | the result-confirmation talk | step 6; kept in the frozen graph |

- **Model and validation** (`shared/model.ts`, `shared/validate.ts`, `server/yamlDoc.ts`):
  - the fields are read, round-tripped and validated with errors `execution-agent`, `execution-profile`, `execution-guide-path`, `intake-mode`, `confirms-kind`, `confirms-type` and `confirms-duplicate`;
  - a workflow that declares none of them writes none.
- **Approval** (`shared/canonical.ts`): the definition digest covers the fields **only when they are declared**. Existing workflows such as `rts-vs-bot` keep their digests, and changing a profile makes the approval stale.
- **Compiler** `ag.wfe-compile.v2`: authored nodes keep `confirms`. Runs compiled by v1 are followed as frozen.
- **Credentials stay in host configuration.** The workflow names a profile; the host's `agents.toml` maps it to a harness and model.

### Identity and ownership

`Executor` gains `host`. A run requested through the execution host is performed as the definition's agent (`ppt-agent`) and records `host: autolab`.

The host's ownership checks now use the host, not the agent name:

- the workspace holder;
- `afterAnswer`;
- `resume`;
- reconcile.

`RunSummary` shows both. A workflow without `execution.agent` is performed as the host itself, as before. No adapter for the old hardcoded configuration was added.

### The attempt records what it was launched with

`wfe exec claim` now takes what the host offers:

- `--profiles-json {name: backend}`;
- `--contract <name>=<sha256>`.

`ExecStore.execution()` resolves the run's settings from its **frozen** definition (the bundle, not the current file) and reads the guides from the workspace.

`attempt.begin` records the settings in the history entry and on `control.attempt.settings`:

```json
{"agent": "ppt-agent", "profile": "sonnet", "backend": "claude_code / anthropic/claude-sonnet-5",
 "contract": {"name": "wfrun/guide.md", "sha256": "…"},
 "guides": [{"path": "agent/guide.md", "sha256": "…"}, {"path": "agent/create.md", "sha256": "…"}]}
```

Two cases **drop the job with the reason** instead of launching something else:

- a profile the host does not offer;
- a guide that cannot be read as a regular file of the workspace.

The claim returns the guide texts, which are not recorded, so the prompt is composed from exactly what was hashed.

### The execution host (autolab)

`wfexec`:

- offers every profile in `agents.toml` that resolves (8 today) and the SHA-256 of the contract text it will use;
- launches with the claim's `execution.profile`;
- composes the prompt as **common contract → `# Project guide: <path>` sections → `# This attempt` brief**.

The rest of the loop is reused unchanged: the queue, fencing, process groups, stops, publication and the notifier thread.

The common contract `agent/guides/wfrun/guide.md` is now **agent-neutral**. It no longer says "You are autolab"; the brief names the agent and the host that launched it. It states the facts the plan lists:

- the braindump and later input are separate records;
- questions and answers are workflow records, and the interface (Zulip or agdevworld) only delivers them;
- every node boundary, and the last completion, checks pending instructions;
- receiving, taking up and acting are distinct;
- plan agreement, permission to finish, execution completion and result acceptance are four different records;
- the host publishes and posts, the agent does neither.

The guide already names the commands that steps 5 and 6 add (`wfe run instructions`, `wfe run ack`, `ask --subject`, `decide --question`). They are deployed together in step 7.

## Checks

- **editor:**
  - `node --test 'test/**/*.test.ts'`: **154/154** pass, including the new `test/settings.test.ts` (6 tests):
    - fields read, written back and validated;
    - digest stable without the fields and stale on change;
    - compiler v2 keeps `confirms`;
    - a run performed as `ppt-agent` and still owned by the host;
    - the claim records profile → backend, contract and guides by hash, hands back the texts, and writes them into the brief;
    - an unknown profile or a missing guide drops the job and begins nothing.
  - `tsc --noEmit`: clean.
- **autolab:** `uv run pytest -q tests/test_wfexec.py tests/test_wfnotify.py` gives 24 passed before the new test, then `test_workflow_settings_choose_profile_guides_and_identity` passed. That test runs the real CLI with a stub agent and checks:
  - the harness profile was the workflow's (`stub`);
  - the prompt starts with the contract;
  - the project guide comes before the brief;
  - the brief names `guide-agent` and the host;
  - `run.json` records the agent, host, profile, backend and the hashes of the contract and the guide.
- **Real data, read-only with the source CLI:**
  - `rts-vs-bot` `improve-game/run-001` and `run-002`: `wfe run check` says usable (frozen v1 graphs);
  - `ppt` `create`: valid with the modelled fields.

## Not yet deployed

Nothing was released. The installed editor is still `78caec2` and the running `wfexec` daemon still runs its old code (step 7 deploys both).

One interim effect: the daemon reads `wfrun/guide.md` from disk at every launch. An attempt launched before step 7 would get the new contract text, which names commands the installed editor does not have yet. No run is queued or active (`Execution host: available — autolab; idle`), and the RTS project has no pending work. I left it so; step 7 releases both sides together.

## Next entry point

Step 4: the durable Zulip intake.

- **Editor:** inbox store, routing from the notifier's channel targets and the workflow's `intake`, application via run operations with source receipts. Request creation becomes idempotent by its `run.create` receipt.
- **autolab:** the transport (cursor, history reconciliation, `topicFirst`, bot flags).
