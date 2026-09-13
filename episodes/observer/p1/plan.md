# Observer p1 — one-shot natural-language watches

## Goal and scope

Add an Observer agent that accepts a watch in Zulip, periodically observes its target using a local model and tools, and notifies the requesting conversation once the condition is met. Waiting spans short evaluations rather than one long model session.

This is a private experimental environment and a breaking-change phase. Backward compatibility and migration are unnecessary. Choose the package layout, harness, tools, record format, and polling implementation freely. Replace existing code where useful; security hardening, elaborate command restrictions, and approval workflows are outside this phase. Give the agent enough command access and usage information to observe its targets.

The minimum product is one dedicated agent channel, one topic per watch, fixed-interval evaluation, one terminal notification, and restart recovery. Sequential evaluation is sufficient. Start with files visible from the Observer host and a specified Zulip conversation.

Defer topic-local command shortcuts, recurring watches, condition editing, adaptive scheduling, event-driven optimization, distributed observation, dashboard work, and ComfyUI notifier replacement. Support simple cancellation by resolving the watch topic; interruption of an evaluation already running can wait for a later phase.

## step1 — Establish the agent and request contract

- Inspect the environment through Nautobot/nctl and the two projects' ignored environment notes. Select an available local model and verify tool execution from the actual agent runtime.
- Create an Observer instance with its own Zulip bot, channel, and introduction. Reuse the shared agent foundation where helpful. Keep its identity distinct from the existing Opsroom Observer bot.
- Accept one watch per topic in the instance channel. The request supplies a natural-language condition, observation target, and destination conversation; capture the requester identity too. A message link is a useful destination format because it supplies a stable message ID.
- Keep the accepted request and lifecycle readable in that topic. Store enough progress to resume after restart, such as the last observed message, latest result, and notification record. Zulip is the request record; an ignored local execution store is fine.
- Acknowledge in the Observer topic without mentioning the requesting agent. Missing information gets a concrete question there; a request needing clarification is not repeatedly evaluated as an active watch.

Verify: a valid request becomes active, its destination is identifiable, and the introduction explains how to request and cancel a watch.

## step2 — Observe through short local-model evaluations

- Run a persistent worker with a configurable fixed interval. Evaluate active watches even when Observer was their last speaker. With no active watches, make no model calls.
- Give each evaluation only its accepted request, relevant previous observation, fresh evidence, and concise tool instructions. Allow additional reads when needed; do not pass the whole realm or unrelated development guides.
- Provide practical file/command and Zulip-reading tools. Let the model choose how to gather evidence and judge the natural-language condition; a predicate language or per-condition plugin framework is unnecessary.
- Return a small result distinguishing **met**, **not met**, and **unable to observe**, with evidence or a reason. Preserve enough context to recognize a question already answered or previously examined.
- An observation failure remains distinguishable from a negative condition and can retry on the next interval. Bound each evaluation so an unreachable target or stuck model does not block the worker indefinitely. Keep routine polling details in local records instead of posting every poll.
- Stop scheduling a watch after its topic is resolved. Recheck cancellation before notifying; recovering active watches after restart must not depend on a fresh Zulip post.

Verify: an unchanged watch is reevaluated without new messages, independent watches make progress, a failed read is not reported as success, and an empty queue causes no inference.

## step3 — Notify and finish

- On fulfillment, deliver a concise notification with the watch reference and observed evidence to the designated conversation. Route it so the requesting agent is served through its normal listener.
- Record successful delivery and finish the watch visibly. Recover pending work and delivery state after restart; ordinary repeated polling must not notify again.
- Retain failed deliveries for retry. A stable watch identifier in the notification can support read-back after an ambiguous send result. A general exactly-once messaging system is unnecessary; document any remaining crash window.
- Resolve the destination from its message anchor when sending so a rename does not redirect it to a different conversation. If the destination is absent or already closed, record an undeliverable terminal outcome in the Observer topic rather than creating a replacement conversation.

Verify: fulfillment causes one notification and no further evaluation; a normal restart does not replay it; delivery failure remains visible and recoverable.

## step4 — Deploy, prove, and document

- Add focused tests for lifecycle/restart recovery, cancellation, and notification replay. Use controlled observations for failure cases rather than building an exhaustive framework.
- Install the worker as a launchd service using the existing deployment conventions. Update deployment declarations where needed and publish the actual entrance/tool contract.
- Demonstrate a file watch with an explicit completion signal, then a natural-language conversation watch: an unrelated progress post stays quiet, an unanswered request directed at the specified recipient triggers notification. Use the real local model for these judgments.
- Exercise at least one request made by an existing in-system agent and show that the notification resumes the intended conversation. Observe the full path, including whether the acknowledgement accidentally wakes the requester.
- Restart with a pending watch and again after delivery. Check the retained request, resulting notification, and finished state. Record model/harness, evaluation duration, incorrect judgments, and remaining unproven cases.
- Update relevant guides, `devdocs/README_DEV.md`, and ignored environment notes. Write `report.md`, then commit/push changed repositories and necessary submodule pointers. Keep machine details and detailed local evidence in ignored files.

## Implementation hints

- `pyagag/src/agag/agent.py` and `templates/` provide instance/listener scaffolding. `agent_config.py` and existing agents' `agents.toml` show backend configuration; select the actual endpoint/model from local state rather than copying a historical name. Follow `devpolicy/agent_records.md` for run evidence.
- `pyagag/src/agag/zulip.py`: `sweep_topics` skips a topic when this bot spoke last. `sweep_serve(on_sweep=...)` calls that hook after startup/queue-registration sweeps, not on a fixed timer. Observer needs a separate due-watch trigger; a launchd plist change alone does not supply it.
- `pj-agdev/comfynotify/src/comfynotify/{cli,commands,notifier,tickets}.py` contains a small persistent polling worker, intake tracking, delivery, and recovery examples. Reuse selectively; the ComfyUI-specific protocol need not become Observer's protocol.
- Existing callbacks use `[selfnote][rootchat]` and `[served]`; inspect the intended recipient's route when integrating. An ordinary post in another agent's owned topic can start it, and a mention in Observer's topic can call it back. Acceptance and polling chatter therefore matter operationally.
- `conversation_of` in the shared Zulip client resolves a message ID to its current conversation. Existing resolve/rename lessons apply even though old formats need no compatibility.
- A file's existence alone need not mean download completion. For the first proof, use a producer that renames a completed file or writes a completion marker. A conversation judgment needs the intended recipient and enough recent context to distinguish an unanswered request from progress or an already answered question.
- `pj-agdev/devenv/launchd/` contains service templates. Opsroom Observer belongs to the agentroom state reader; its credentials and role are not the new agent's identity. No dashboard integration is needed to prove p1.
