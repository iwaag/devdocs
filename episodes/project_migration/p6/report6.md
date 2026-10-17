# project_migration p6 — Step 6 report: visible, adaptable workflows

Status: complete (2026-10-17), as authored and rehearsed workflows. Real agents run them in Step 7. playprey
`6de0b0df` holds the workflows and guides. Raw evidence, including the rehearsal logs, is in the ignored
`pj-agdev/.local/project-migration-p6/step6/`.

## Workflows

| Workflow | Outcome | Graph |
| --- | --- | --- |
| `onboard` (new) | Supplied models become an evaluable profile within the agreed scope, delivered and confirmed, with the case's lessons recorded | `_plan` → inspect → construct → register → fit → verify → record → present → confirm (`confirms: result`) → cleanup → `_final-report` |
| `compare` (new) | Two or more approaches to one method question stay separately identifiable candidates; the person chooses with evidence; the adopted method goes into tools/ and knowledge/ | `_plan` → frame → {build-a, build-b} → compare → choose (talk) → adopt → confirm (`confirms: result`) → cleanup → `_final-report` |
| `freework` (Step 5) | Whatever the conversation agrees | talk |

Design choices against the plan:

1. **The scope decides the work** (item 1). `onboard` agrees a scope in its plan: either static construction
   only, or construction plus the requested motion and interactions. `fit` is in the graph, but a static
   scope removes it with an adjustment based on the agreement. It is not completed empty.
2. **No proliferation** (item 2). General work stays in `freework`. `compare` exists because its outcome
   is different: candidates, a choice, and a lesson fed back into tools/ and knowledge/, not one model.
3. **Visible decisions** (item 3). Each node is an investigation, approach or decision. Node descriptions
   tell the agent to add an investigation or alternative as a node (`wfe run adjust`) rather than hide it
   inside a long step. Parameter iteration stays inside a node with its evidence. The parallel branches in
   `compare` mean both candidates are built; they are not a condition.
4. **Knowledge before confirmation** (item 6). In `onboard`, `record` writes `knowledge/cases/<profile>.md`
   before `present`/`confirm`. In `compare`, the comparison is written for every candidate before `choose`,
   and the adoption is written before `confirm`. That is why `choose` is a plain talk and the
   result-confirmation comes after `adopt`.
5. **Large files.** `onboard` publishes its delivery (`tools/delivery.py`) with `wfe run publish` as a named
   candidate. The confirmation asks about it with `--output` together with a small presentation.

## Versions (item 4)

Written into `knowledge/workflows.md`:

- An improvement to a workflow is an edit of its file and applies to new runs, because each run freezes its
  definition.
- Approaches that should coexist get distinct workflow ids.
- A request-specific change is a revision inside the run.
- Candidates are named when published, and each case record names a candidate's base, so lineage is
  readable without the run.
- A change that would rewrite completed work becomes a successor run.

`knowledge/cases/README.md` fixes what a case record holds: inputs by digest, applicability, failed
assumptions, approaches and candidates with their base, the adopted configuration, limits and evidence
links.

## Engine limits tested before proposing changes (item 5)

I ran a scripted rehearsal on the deployed WFE (`45ffada`) with a copy of `onboard`, in a private area, and
did not touch the playprey project:

| Situation | How it was expressed | Result |
| --- | --- | --- |
| Agreed scope is static construction | Plan question q1 answered, then `adjust --source answer:q1 --completing _plan` removes `fit` and rewires register → verify | Applied as revision r2, together with `_plan`'s completion |
| The person asks mid-run for a second construction candidate | `instruct` → the agent acknowledges (`incorporate`) → `adjust --source instruction:i1` adds `construct-alt` after the completed `construct`, before the pending `register` | Applied as r3; register then waited for both and started |
| Trying to adjust while a node runs, or with an unacknowledged instruction | — | Refused with a precise reason (`mid-node`, `instructions-pending`) |
| `wfe run revisions` | — | r1 (original), r2 (answer q1), r3 (instruction i1), each with its reason |

A repair or re-verification after a review answer is the documented p7 adjustment path and needs nothing
new. Delegation is not needed by these workflows. **No common-engine change is proposed.** The
limitation stays: edges order work and are not conditions or loops, and `knowledge/workflows.md` says so
for the agent.

## Guides

- `agent/guide.md` now points to `knowledge/workflows.md` and the case-record format.
- The node descriptions carry each step's contract: the input set, `.local/` for large files, adding
  assets with `tools/assets.py add`, and the distinction between mechanism and tuning.
- None of these restate the WFE wfrun guide.

## Also in this step

- The transport provisioned `wf-playprey.onboard` and `wf-playprey.compare` automatically.
- The workflows are not approved. As in the other managed projects, approval is the Developer's.
- Filed for Step 7: cagent `change/run-007` (predecessor run-006). It repairs the Linux client's NFS hang
  for good and proves the idle-drop case this time. The request records fresh evidence: eight CLOSE_WAIT
  sockets on the server, FIN-WAIT-2 on the client, and "not responding" in the kernel log. Its questions go
  to the Developer.
- A transient Git index lock in the workspace came from the service reading it while provisioning channels.
  Retrying succeeded; nothing was removed.

## Acceptance check

- **A reviewer can see what was tried and why the plan changed:** the rehearsal shows each change as a named
  revision with its authorizing record (answer, instruction). A real case shows it in Step 7.
- **The agent keeps its judgement within the requested scope:** nodes state outcomes and contracts, not
  procedures. The scope is agreed in the plan. Additions and removals are the agent's proposals, grounded in
  the person's records.
