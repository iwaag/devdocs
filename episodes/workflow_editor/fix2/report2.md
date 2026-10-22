# workflow_editor fix2 — Step 2: runtime and existing definitions

## Runtime assumptions

Each runtime reader asks whether `intake.zulip === 'requests'` of the
loaded model. The loader's new reading therefore reaches all of them
without a change of their own:

| Reader | Where | Change |
| --- | --- | --- |
| Intake discovery | `server/intake.ts` `routes()` | An unreadable workflow file used to be skipped silently, so a wrong-typed intake looked like an opt-out. It is now a routing problem (`wfe intake status`): `<project>: <file> cannot be read: $.intake.zulip: …` |
| Notifier channel provisioning | `server/notify.ts` | None. Every workflow already gets its `wf-<project>.<workflow>` channel, whatever its intake. All newly eligible workflows are provisioned |
| Delegation target check | `server/delegate.ts` | An unreadable target is named with its problem, not reported as "no workflow". An opt-out is refused as `opts out of requests (intake: {zulip: ""})` |
| Workflow summaries, peers, validation context | `server/workspace.ts`, `server/peers.ts`, `src/inspector.ts`, `cli/projects.ts` | None (they read the model). Messages name the opt-out |

Tests:
- `intakestore.test.ts`: a workflow without intake is routed; an opt-out
  is not; a wrong type is a reported problem.
- `delegateflow.test.ts` (new case): a child workflow with no intake
  accepts a delegation. The child is created from the request post,
  entrusted by the delegation and bound to the parent. A child with
  `zulip: ""` refuses dispatch, and its channel is not read.

## Inventory of registered workflows

Every workspace's file, and `origin/main` of each project, agree. 13
workflows had no intake. All the others declare `requests`, except
agops' three check fixtures (`check`, `context-check`, `session-check`),
which already declare `""`.

| Workflow | Purpose | Decision |
| --- | --- | --- |
| aghangout/change, aghangout/freework | project work | **enabled** (the run-045 targets) |
| characters/character, characters/freework | project work | **enabled** |
| front/converse, archsage/converse, forge/converse | one message of a conversation (conversation lane) | explicit opt-out |
| front/discuss, archsage/discuss, forge/discuss, jevlab/discuss, rts-vs-bot/discuss, aghangout/discuss | one turn in a discussion (conversation lane) | explicit opt-out |

Conversation and discussion turns are created by the conversation and
discussion services, with the source they serve. A request topic or a
delegation would create a turn with nothing to serve. So their opt-out
follows from their purpose, not from the old default. Each file now has
`intake: {zulip: ""}`, with a comment that says why. The commits were
made in the registered workspaces, after fast-forwarding them to
`origin/main`, and pushed:

| Project | Commit |
| --- | --- |
| front | `83815ef` |
| archsage | `34575c4` |
| forge | `46ffdf9`, then `d8e38cb` |
| jevlab | `b32109b` |
| rts-vs-bot | `61fbae3` |
| aghangout | `3eca7c5` |

`46ffdf9` also picked up forge's untracked local fixture
`provide-review-check.yaml` by mistake. `d8e38cb` untracks it again, and
the file stays in the workspace as before. `wfe validate`: all nine are
valid. These commits come before the release (Step 4), so the
conversation and discussion channels are never routed.

## Newly eligible channels

Before the release I read the history of the four channels that become
routed (`wf-aghangout.change`, `wf-aghangout.freework`,
`wf-characters.character`, `wf-characters.freework`). They hold only the
notifier's own posts in `run-…` topics (14, 3, 7, 18). Run topics are
never input, and there is no `request-…` topic. Their cursors start at
0, so the first pass reads that history, and nothing in it becomes a
request or an instruction.

## Existing runs, frozen definitions, digests

- Run snapshots are not rewritten. The intake and delegation read a
  workflow as it is now in its workspace, never from a run's frozen copy.
  A frozen copy without intake is re-read as `requests`, but nothing
  routes by it.
- Recorded digests (`run.create` `definitionDigest`, `run.adjust`,
  `delegation.dispatch` `targetDigest`) are evidence. Nothing recomputes
  them and compares. For files without intake, a digest recorded before
  the release differs from one computed after it. None of the 13 files
  holds an approval record, so no approval becomes stale. Approvals are
  records, not permissions, in any case.
- Discussion workflow versions store the digest of each version
  (`server/discussionWorkflows.ts`). A comparison between a version
  recorded before the release and one recorded after can call an
  unchanged file without intake "behavioural". This is display only, and
  no recovery is needed.

## The Ponderer publication request

`front/converse/run-045` (conversation `20261022-001309-j4e9lp`, the
Developer's message #258) made `aghangout/freework/run-001` as a plain
request by `front-agent` on behalf of iwaag, entrusted by run-045. That
run is **completed and integrated**: "Ponderer now has portrait, lore and
style (characters export 0631d47, commit a514c72); cast.json room argue;
checks pass; deployed at /hangout/".

Front's reply (#259) asked the Developer to check the aghangout run. It
said that aghangout "does not accept delegations from other projects".
Nothing ever returned the result to the conversation. No recovery run or
second publication is needed or made. The request has no delegation and
will not get one. The only thing missing is telling the Developer, which
the final report does. Step 3 handles Front's framing.
