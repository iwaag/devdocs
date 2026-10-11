# p6 step 1 — inbound and conversation contracts

Plan: [plan.md](plan.md) §Step 1.

## What was inspected

The current code, not earlier reports:

- agdevworld `78caec2`, the installed release (`wfe version`, `service/current`);
- agautolab `ca8e807`;
- the operating area's registry, `zulip.json` and notifier state.

| Mechanism | Where | What p6 reuses |
| --- | --- | --- |
| Run reducer | `shared/run.ts` | Every input becomes a history entry under the run lock. The entry's `receipt` makes a retransmission a no-op (`operate`). Attempt fencing applies to executor operations. `question.answer` / `take-up` are separate records. |
| Request receipts and queue | `server/exec.ts` | `request()` creates the run and then queues a job keyed `request:<receipt>`. `afterAnswer()` queues a resume per answer. `finished()` resumes for an answer that arrived while the asking attempt ran. `reconcile()` requeues lost jobs. |
| Publication checkpoints | `server/exec.ts`, `server/publish.ts` | Commit-addressed links come from published checkpoints. |
| Notification delivery | `server/notify.ts`, `agautolab/wfnotify.py` | Protocol 2 provides frozen deliveries, claims, `begin` before POST, marker search, relocations and one sender. Today it has one destination per run: the `run-…` topic in `wf-<project>.<workflow>`. |
| Listener routing | `pyagag agag.agent.topic_filter`, each agent's `instance.py` | Legacy listeners sweep their own channel and their prefixes. These are `workplan-`, `workrun-`, `bmining-`, `front-`, `routinerun-`, `argue-`, `assetplan-`, `assetrun-`, `watch-`, `cagent-` and `change-`; empty prefixes are dropped. Neither `request-` nor `run-` matches any of them. |

One gap found in the existing code. `ExecStore.request()` takes its receipt only on the queued job; `run.create` carries none. A crash after `createRun` and before `enqueue` would create a second run when the same request is retried. Startup recovery requeues the orphaned run. It cannot tell a retry from a new request. The contract below closes this: the run itself carries the source receipt, and step 4 implements the lookup.

## The contract

agdevworld commit `7fbd51f`:

- [`workflow_editor/shared/intake.ts`](../../../../pj-agdev/agdevworld/workflow_editor/shared/intake.ts): types and pure rules;
- [`workflow_editor/docs/intake.md`](../../../../pj-agdev/agdevworld/workflow_editor/docs/intake.md): the written contract;
- [`workflow_editor/test/intake.test.ts`](../../../../pj-agdev/agdevworld/workflow_editor/test/intake.test.ts): 13 tests.

The plan's numbered rules map to the implementation as follows:

| Plan rule | Implementation |
| --- | --- |
| 1. Explicit channel → project/workflow; no prefix inference | `Route {realm, streamId, project, workflow, workspace}`. A channel counts only if the notifier recorded its stream id, the workflow declares `intake: {zulip: requests}`, and the host enables the project. `routeMessage` never reads a channel name. |
| 2. `request-…` = one request, one run; the first eligible post is the braindump | `topicKind`; `classify` → `request` for an unknown request topic. Bot and unauthorized posts are ignored, so the first *eligible* post becomes the braindump. |
| 3. Original text, sender, message id, time and place are recorded; edits are not applied | `InboundMessage` carries all of them. `SourceRef` is stored with the input it became. Edits are out of scope (doc). |
| 4. Bot posts and `run-…` posts are never input | `ignored: bot` is checked first, even for a configured id. `routeMessage` ignores `run-topic` even for bound topics. |
| 5. A question names its id and expected action; answers bind by reference or to an unambiguous pending question; the binding is persisted | References are quote-and-reply / message links (`…/near/<id>`) to the post that carried the question, or `q<n>:` at the start. Otherwise the binding goes to the one open question **whose post is older than the reply**. A post therefore cannot answer a question its author could not have seen, even after downtime. The classification is stored once and never derived again. |
| 6. Ambiguous replies need clarification; late replies are recorded and explained | `clarify` (ambiguous / unknown-question / not-yet-asked) and `late` (taken-up / withdrawn). Both are applied nowhere and both get a request-topic notice. |
| 7. After completion, a new request needs a new topic | `closed: completed / cancelled / refused`. The post is kept and explained, and no run is created or reopened. A *stopped* run is not over, so input becomes an instruction. |

- **Receipt / application / take-up** are three facts with separate evidence:
  - an inbox row written with the cursor move;
  - a run history entry carrying `zulip:<realm>:<message id>`;
  - `question.take-up` / `instruction.ack` by the attempt.
- **Inbox stages** are `received → classified → (applied →) done`, plus `failed` with resumption at the failed stage. `canMove` / `nextStage` encode this. Crash handling relies on the run operation's receipt, not on an inbox flag. For a request, the run is found by its `run.create` receipt before anything is created.
- **Posts of one request** are settled in message-id order (`wait`).
- **Rename / move.** A topic's identity is its first message (`topicFirst`, supplied by the transport):
  - a rename or resolve is followed and recorded;
  - an old name reused by a new topic is a `topic-name-reused` problem;
  - a request moved to another channel is `moved-across-channels`.

  Nothing is guessed from a name.
- **Additional instructions** are defined here as run history. `instruction.receive` never changes node state. `instruction.ack` has the responses `incorporate | clarify | new-run`. The boundary enforcement is step 5.

## Checks

- `node --test test/intake.test.ts`: 13/13 pass. The tests cover realm and receipts, topic kinds, input shape, channel routing, identity across rename/resolve, reused names, cross-channel moves, bots/unauthorized senders, the first eligible post, waiting order, the three binding routes, posts older than the question, ambiguous/late/unknown replies, completed/cancelled/stopped runs, reference parsing and stage transitions.
- `tsc --noEmit`: clean.

The full suite and the build run at step 7 together with the code that uses this contract.

## Departures and decisions

- **Identity by first message** is my concrete answer to "retain the recorded binding when identity can be established". Zulip gives topics no id; the first message is the only stable handle. If that message is deleted, identity is lost and posts are reported as a problem, not guessed.
- **An open question absorbs replies.** While exactly one visible question is open, every eligible post is its answer, even one the person meant as an instruction. The agent reads the text when it takes the answer up. The alternative was to classify meaning in the intake, which the plan reserves to the agent.
- **Stopped runs accept instructions.** Execution `stopped` (a failed node) or a hold still records instructions. They are taken up at the next attempt after a person resumes the run.

## Next entry point

Step 2: create `pj-ppt` (project `ppt`) with `study-ppt` / `wedo-ppt` and the `study` and `create` workflows. The workflow fields `execution` and `intake`, and the result-confirmation marker on the `create` talk node, are added to the model in step 3. Step 2's definitions are written with them in mind.
