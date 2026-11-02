# improve_wfrun p2: handout

Items handed over from p1 ([p1 report](../p1/report.md)).

## gimmickimg `partial-animation` is published but cannot run

- **What**: `developer/pj-gimmickimg` has `devdocs/workflows/partial-animation` on its default branch
  (integrated by `freework/run-018`). It fails validation with `confirms-type`: node `review`
  "confirms the result but is "review"; only a talk node asks the person."
- **State after p1**: WFE now discovers it from the published source. The executor provisioned its
  channel (`#846 wf-gimmickimg.partial-animation`) without a restart. `wfe delegate targets
  gimmickimg` reports it `NOT READY`, so requests and delegations to it are refused until the
  definition is fixed.
- **Why it was left**: the definition is gimmickimg's content. p1 changed nothing in the project.
- **To do**:
  - Have gimmickimg fix the definition, e.g. by asking its agent or freework. Either make `review` a
    talk node or move the result confirmation to a talk node.
  - Check with `wfe delegate targets gimmickimg/partial-animation` that it is `ready`.
  - Run it once to confirm it works.
- **Root cause on the WFE side**: integration does not validate the definitions a run adds, so the
  run that created the workflow finished without noticing it was unrunnable. p1 lists this as
  follow-up work: validate added definitions at integration time, or report them as not runnable at
  once.
- **Related**: the gimmickimg workspace is 6 commits behind its published branch. That is not
  needed for discovery or execution. Updating it is optional (Update workspace /
  `wfe workspace sync`).
