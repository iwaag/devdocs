# workflow_editor fix2 — Step 4: verify and deploy

## Checks before deployment

- `npm run check`: 515/515, plus tsc and both builds. The focused files
  are `intakedefault`, `settings`, `contract`, `intakestore`,
  `delegateflow` and `basics`.
- `nctl drift`: 46 converged, 2 drifting (`agobserver`,
  `cagent-snake-e2e`). Both drifts existed before and have nothing to do
  with this change. The result was the same after deployment.
- No attempt was open. The six WFE stores were backed up first.

## Deployment

| Component | Deployed | How |
| --- | --- | --- |
| WFE on the coordinator | agdevworld `0d49456` (previous `1e2fbed`) | `wfe-release.sh deploy` (release check 515/515); the service and `bin/wfe` report the commit; no store change |
| Workflow opt-outs, Front guide | workspaces (Step 2, Step 3) | committed and pushed before the release |
| pj-agdev | pins `c63121e` | |
| agpc execution node (`gpupc`) | **pending** | Nautobot placement `agautolab-agpc` `wfe_version` → `0d49456` is applied (dry run: 1 update, then committed). The auto-mode classifier refused `nctl render production`, so the setup playbook has not run |

The agpc worker is not needed for this fix. Intake, routing, delegation
and the notifier run only on the coordinator. Until the playbook runs,
the agpc node's own `wfe` still loads a workflow without intake as "off".
That reading only affects local commands such as `wfe validate` on that
PC. nctl compares placements by existence, so the pending pin does not
show as drift.

## After the release

`wfe intake status` routes four more channels:

- `aghangout/change` (#477)
- `aghangout/freework` (#467)
- `characters/character` (#595)
- `characters/freework` (#596)

The nine converse/discuss workflows are not routed. The first transport
pass read each channel's history and settled every post as `done`. All
of them were notifier posts in run topics. No run was created and nothing
was posted.

## Trials (author: Omni Agent for the Developer)

**Delegation to a workflow with no intake declaration.** I opened the
conversation `fix2-s4-delegate` with Front. Message #269 asked Front to
entrust a read-only question to `characters/freework`: the cast, and the
first line of ponderer's lore. `characters/freework.yaml` has no `intake`.

| Step | Record |
| --- | --- |
| Dispatch | `front/converse/run-046` delegation d1, 05:50:23 |
| Delivery and child | request post 23045 in #596 › `request-front.converse.run-046.d1`; the intake created `characters/freework/run-007` (requester front-agent, on behalf of Omni Agent (for Developer)); `delegation.bind` 05:51:01 |
| Result returns | the child completed; `delegation.event` (completed, manifest `1c180e5a66eb`) queued the parent, which resumed at 05:51:34 and took the fact up |
| Judgement | `delegation.dispose` satisfied |
| Follow-up with the requester | Front's reply #271 gave the cast (front, forge, archsage, autolab, ponderer; rooms front, argue, project) and the lore line "カエルの大魔導師。", and named the child run |

**Explicit opt-out refuses delegation.** `agops/freework/run-001` was
asked to add a delegate node to `front/discuss` (`intake: {zulip: ""}`)
and to dispatch it once, with no fallback. The dispatch was refused with:
`front/discuss opts out of requests (intake: {zulip: ""}); a delegation
is delivered through its intake`. Nothing was posted, and no run of
front/discuss was created.

## Observations

- Front posted its result from the attempt that judged the child. It then
  ended without starting the `report` node that it had added after the
  delegate node. `front/converse/run-046` is therefore held as
  `interrupted` (report ready, nothing waiting). The requester already
  has the answer. I left the run held: resuming it would let Front post
  again. The node-ending lesson is already in Front's guide (run-027).
  This failure is not about intake or delegation.
- `wfe run show` prints the child's input as "entrusted by note \"\""
  for a delegated run. The record (`entrustedBy.kind = delegation`) is
  correct. This is a display oddity only, and it predates this change.
