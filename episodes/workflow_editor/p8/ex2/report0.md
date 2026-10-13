# p8/ex2 step 0 — survey: what changes, the policy format, the activity unit, the boundary gap

Step 0 of [plan.md](plan.md). Nothing was changed. Production data was only read.

## Checkouts and the deployed release

| What | State |
| --- | --- |
| agdevworld | `main` = `origin/main` = `6bdcd3b`, clean |
| pj-agdev | `main` = `origin/main` = `fa96822` (pointer to `6bdcd3b`, deployed) |
| agautolab | `main` = `origin/main` = `f1c466d` (the wfrun guide, read by wfexec at each claim) |
| Installed editor release | `6bdcd3b` (`service/previous` = `8178431`); no open attempt |
| Baseline tests | `npm run check` in `workflow_editor`: tsc clean, **217/217** tests pass, Vite build OK |

The catalog has three projects (`rts-vs-bot`, `ppt`, `agsfx`), and none has a `freework` workflow. Their workflow files:
`rts-vs-bot`: `build-game` (v1, unsupported) and `improve-game`. `ppt`: `create` and `study`. `agsfx`: `create`,
`improve-tools` and `research`.

## Branches on the policy name that must change

These places branch on the policy name today:

| Where | What it does |
| --- | --- |
| `shared/compile.ts` | `manual` returns the authored graph; anything else expands to plan + reports + final |
| `shared/validate.ts` | `documentation-policy` accepts only `default` / `manual` |
| `shared/canonical.ts` | the definition digest projects `{policy}` |
| `shared/run.ts` `run.adjust` | refuses `next.policy !== cur.policy` |
| `shared/run.ts` `notableEvents` | `reporting = policy === 'default'` decides whether a node completion is news |
| `server/runs.ts` (bundle check) | `eff.policy !== root.documentation.policy` |
| `server/runs.ts` `adjustRun` | `documentation` must be unchanged (canonical JSON of the model) |
| `server/exec.ts` `documentationBrief` | three fixed texts (none / manual / default) |
| `server/workspace.ts` `createWorkflow`, `server/api.ts`, `cli/wfe.ts` | `--policy default\|manual` |
| `src/runView.ts` (6 places), `src/workflowView.ts` (policy chip, preview), `src/inspector.ts` (policy select), `src/canvas.ts` (step tooltip) | `policy === 'default'` decides whether generated steps are drawn and counted apart |
| `cli/run.ts` | the `show` line about the policy |
| `server/yamlDoc.ts` | `documentation: record({policy})` — the only key the editor owns |

## Decision: the policy storage format

Here is the new form. `default` and `manual` keep their meaning and their digests:

```yaml
documentation:
  policy: custom          # default | manual | custom
  plan: false             # the plan talk (_plan) before the entry nodes
  reports: false          # a report step after every authored node
  final: true             # the final report (_final-report)
```

- `custom` requires all three keys, each `true` or `false`. A missing key is the error `documentation-steps`.
- `default` and `manual` must not carry any of the three keys. If they do, that is `documentation-steps`: a
  contradiction is refused, never reconciled.
- A non-boolean value (`plan: yes-please`, `plan: "true"`) is a shape issue. The file reads as unusable, as other
  wrongly shaped fields do. Nothing is coerced.
- The normalized setting `{plan, reports, final}` is what the compiler, validation, the brief and the views use:
  `default` = all true, `manual` = all false.
- The definition digest gains the three keys **only for `custom`**, so every existing approval stays valid.
- The compiler becomes `ag.wfe-compile.v3`. Its graphs carry `documentation: {plan, reports, final}`. For
  `default` and `manual` the nodes and edges, step instructions included, stay byte-identical to v2. Old runs
  can therefore be adjusted under v3 without touching a frozen generated step. A v2 graph's setting is derived
  from its `policy`.
- The final report's instructions change only when node reports are off. The v2 text ("All authored work and
  its reports are complete") would be false then.

## Decision: the activity unit

The attempt logs (`.local/agdev/.local/exec/logs/<attempt>.jsonl`, written by wfexec from the agent's stdout) are
Claude Code `stream-json`. I counted the 12 most recent attempts, all `claude_code / claude-sonnet-5`. One model
response arrives as several `assistant` events that share one `message.id`. For example, `amv6bxdfm` has 94
`assistant` events, 61 distinct message ids, 60 tool uses and 60 tool results.

- **Unit: a turn = one distinct assistant `message.id`** in the attempt's log. It is stable (a re-read counts the
  same file the same way), monotone while the attempt runs, and means something to the person.
- A log that is not stream-json (another harness) gets **no count**: the view says only that the log was
  written. "Updates" is not invented for it.
- **Scope: the current attempt** ("this session"). A new attempt (after an answer, a resume) starts at 0, and Detail
  lists the count per attempt. A run-wide total would mix sessions of different nodes.
- **Last activity = the log file's modification time.** Only the agent process writes that file. Executor
  heartbeats, SSE reconnects and the view's 3 s polling never touch it, so none of them counts as activity.
- The count is computed on the service, incrementally (byte offset and the ids seen, per log, only up to the
  last complete line), and returned in the existing `GET …/exec` view as a small summary. No log text reaches
  the browser.
- Process alive, last activity and node completion stay three separate facts. Logs exist only on the
  execution host: on another host the summary is "not observed here", never "idle".

## The boundary gap

I checked the reducer (`shared/run.ts`) for the four boundary cases of plan §2:

| Case | Existing API |
| --- | --- |
| talk completes, final report present: add work before `_final-report` starts | **Works.** `_final-report` is pending, so its incoming edges may change; the recompiled graph wires it after the new terminal nodes |
| add work again after the new work, with a new talk | Works (new ids; completed talks are untouched) |
| node reports on | Works: `talk → _report.talk → new` keeps `_report.talk`'s incoming edge |
| **no final report: completing the terminal talk** | **Gap.** `node.complete` of the last node completes the execution, and `run.adjust` then refuses `run-completed`. Adjusting first is `mid-node` |

The minimal general extension is a combined operation: "complete this node and change the future work". The
service applies it as two history entries in one locked write:

1. `run.adjust` with `completing: <node>`. The `mid-node` check exempts exactly that node, provided it is the
   only running node. All other adjustment rules apply (it must stay unchanged, the boundary, authorization).
2. `node.complete` of that node, with its usual checks (documents, presentations and the last-completion
   boundary when nothing is left).

`replay` (and so `wfe run check`) refuses a `completing` adjustment that is not followed directly by that
node's completion. A record can therefore never show an adjustment made in the middle of a node. A completed
run is never revived, and nothing depends on the workflow's id. The CLI form is
`wfe run complete … --adjust <file> --reason … --source …`.

## Placing freework: settings resolved per project

A workflow cannot change its bindings or execution in an adjustment. So `freework` gets them at placement,
resolved from the project:

- **Bindings:** the union of the bindings of the project's valid workflows. Each path gets the most permissive
  access some existing workflow already grants it, so nothing becomes editable that no workflow made editable.
  A project without valid workflows (a new project) gets its root editable, and every submodule and local area
  readonly.
- **Execution:** an `agent` / `profile` that all valid workflows agree on is kept. `guides` keeps the guides all
  of them list. Otherwise the field is left out (the host's default).
- **Intake:** none. Zulip intake stays an explicit opt-in.
- Side effect to report in step 7: the notify thread provisions a Zulip channel for **every** workflow
  (`server/notify.ts`, `ensureTarget` per workflow summary), so placing freework in the three projects creates
  `wf-<project>.freework` channels.

## Where the work goes

- `shared/model.ts`, `validate.ts`, `canonical.ts`, `compile.ts`, `run.ts`; `server/yamlDoc.ts`, `runs.ts`,
  `exec.ts`, `workspace.ts`, `create.ts`, `api.ts`; `cli/wfe.ts`, `run.ts`.
- `src/projectView.ts` (creation entry), `runView.ts` and `runSide.ts` (activity), `inspector.ts` and
  `workflowView.ts` (policy), `canvas.ts`.
- `agautolab/agent/guides/wfrun/guide.md`: growing the run's plan from the conversation, and the combined
  completion.
- `docs/contract.md`, `docs/adjustments.md`, `docs/runs.md`, README.

## Open checks

- `checks/setup.ts` has been stale since p4 (memory of p8/ex1). It is not one of the checks this episode relies on.
- Native macOS IME is still unverified from p8 / p8/ex1. This episode adds forms, so the same caveat applies.
