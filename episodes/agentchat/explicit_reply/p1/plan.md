# Explicit reply p1 — Reliable conversation continuation

Reference: [advice.md](advice.md). Give agents an explicit public reply boundary and make unanswered requests survive interruptions, delivery failures, and handoffs. A conversation should continue without a human posting again merely to wake it.

This is a breaking phase in a private experimental environment. Backward compatibility and migration of disposable state are unnecessary. Replace contracts, schemas, and obsolete code where useful; the implementer chooses storage, APIs, and component boundaries. Reuse existing services and avoid expanding this into security hardening. Keep credentials and machine-specific facts in ignored files.

## Step 1 — Make unanswered work recoverable

- Reproduce the ack/restart and failed-send paths below with focused fixtures, then fix them in the shared listener/topic lifecycle.
- Distinguish receipt, execution, reply prepared, and delivery confirmed. Track the triggering message IDs and the input boundary actually processed; an ack or an unrelated later post is not evidence that a request was answered.
- Keep pending work durable until its reply is confirmed or a terminal failure is explicitly recorded. Retry transient failures with bounded backoff; exhausted retries remain visible and recoverable rather than disappearing from the queue.
- Persist prepared replies before sending. After an ambiguous send, look for the delivery's stable identity before retrying. Prefer retrying delivery over rerunning the model or completed external actions. For interruption during execution, retain available evidence and define how the next run reconciles uncertain actions.
- Ensure posts arriving during a run remain outstanding after the current reply, including when that run requests topic resolution. Use one completion rule across normal execution, restart recovery, and callback handling.

Verify: crash after ack; crash before/after sending; server accepts a send but its response is lost; handler exception; post-run read failure; additional input during execution. Assert eventual response or an explicit failure state, with no duplicate confirmed delivery. Transport receipts do not prove that the user's task succeeded.

## Step 2 — Separate public replies from model output

- Introduce an explicit reply contract, with fenced `ag-reply` blocks as the starting design. Post the selected text; retain other emitted text in existing run evidence. Multiple reply blocks concatenate in order, and replies may contain ordinary code fences.
- Separate model reply text, machine blocks, and system-generated notices before composing the final post. Parse machine blocks from the model output independently; preserve notices such as desire registration and completion validation.
- Update conversational roles and their shared prompt construction to explain the reply mechanism once where practical. Remove redundant prose prohibitions. Structured-output roles such as `present` retain their own output contracts.
- Missing, empty, or malformed replies are explicit generation failures, not successful silence. Choose a bounded repair strategy and provide a visible system failure if it cannot produce a reply. Repair must not repeat already-applied machine-block effects.
- Retire the advice's unmarked-output fallback: compatibility is not required. Record reply parsing and delivery outcomes alongside the run identity and posted message ID.

Verify: preamble plus reply; multiple blocks; nested code fences; missing/empty/unclosed blocks; machine blocks; system notices; a rendering based only on posted speech. Reuse the #7222/#7262 examples from the advice as fixtures; do not rely on a model happening to omit its preamble.

## Step 3 — Bind handoffs to requests

- Capture the requester, triggering message ID, home conversation, and intended response destination at intake. Use stable message/anchor IDs to locate conversations at delivery time rather than treating display names as identity.
- Replace automatic handoff selection from the latest other speaker with the recorded request relationship. A third party speaking during a run must not silently change who receives its answer; a failed lookup remains pending rather than silently removing the handoff.
- Preserve deliberate delegation through `agentchat` and home-directed callback replies. Agents decide whom to consult and what to do next; the transport records and delivers that decision. Keep argue invitations explicit so ordinary replies do not cause agent ping-pong.
- Tie callback served marks to confirmed handling of the relevant input. Account for coalesced requests and multiple outstanding delegates; answering one must not consume another. Define visible outcomes for deleted or closed destinations and follow existing replacement relationships where applicable.

Verify: third-party interjection; simultaneous delegate replies; repeated callback; restart between home reply and served mark; rename/resolve/replacement; reuse of an old topic name. Confirm that completion reaches the requesting conversation without starting a reciprocal reply loop.

## Step 4 — Carry forward the context needed to continue

- Extend the existing conversation-in-prompt mechanism with a compact continuation view: new input, current goal and agreed conditions, outstanding requests, and what to do when their results arrive.
- Derive request IDs, delivery state, and pending relationships from recorded evidence. Let the agent maintain semantic summaries and intended next actions, with references to source messages. Choose a lightweight representation; a separate memory service is unnecessary.
- Refresh the view for every serving, including callbacks. Make omitted history and unavailable evidence explicit, and keep newer user instructions authoritative over older summaries. Preserve access to the underlying conversation when more detail is needed.

Verify a multi-turn delegation with a restart and history truncation: the next run can identify what arrived, what remains pending, and the next decision without restarting completed work. Include a user correction made while a delegate was running.

## Step 5 — Integrate, exercise, and report

- Inventory consumers of the shared contracts: Front's conversational roles, autolab, forge, archsage, cagent, Observer's conversational role, and generated-agent templates. Update affected consumers and dependency locks together; remove obsolete paths where useful.
- Inspect service state through Nautobot or `pj-clusterintent/nctl` before working on running services. Use the ignored environment notes for deployment and update affected processes. A pyagag push alone does not update pinned installations.
- Run focused parser, lifecycle, routing, and consumer tests, then a small end-to-end experiment covering request → delegation → callback → final response. Exercise interruptions with local fixtures first. Describe any proposed live test posts and their destinations for authorization before sending them; other implementation and verification can proceed independently.
- Check that Front Desk/Arguing Room rendering receives only public speech and that conversation status reflects outstanding replies. Reuse existing status surfaces and logs; a new dashboard is optional. Measure lost replies, duplicate deliveries, unnecessary reruns, and recovery delay against the fixtures.
- Update relevant developer docs and ignored deployment notes. Write `report.md` with changes, evidence, remaining gaps, and any live checks still pending. Commit/push changed repositories and required submodule pointers.

## Implementation hints

Paths are relative to the shared projects directory. Findings below are from source inspection; confirm them with fixtures rather than treating them as reproduced live incidents.

- `pyagag/src/agag/listen.py`: `_resume_running()` requeues ack-only work, but `owed()` rejects a topic whose last speech is the bot's ack. `_execute()` calls `queue.finish()` even after handler exceptions. Existing restart fixtures block **before** posting the ack, leaving the after-ack interruption worth adding explicitly.
- `pyagag/src/agag/topics.py`: `serve_topic()` posts only when the joined body is nonempty; final-send failures escape the handler exception boundary. `resolve_after` returns before the ordinary new-input recheck. `handoff_mention()` reads the latest speaker at send time and returns an empty mention on read failure. `TopicResult` is a natural place to reconsider the output contract.
- `pj-agdev/agfront/src/agfront/argue.py`: `serve_argue()` currently joins model text and system notices into one section. Extracting only marked text from that combined section would discard notices. Handle extraction before that join or replace the result structure.
- `pyagag/src/agag/harness.py`: successful runs may have `empty_final`; the conversation layer must decide how to answer. Not every harness user produces conversational text, so apply the reply contract at the appropriate boundary.
- `pyagag/src/agag/selfnote.py`, `chat.py`, `zulip.py`, and `mirror/`: existing home/served notes, anchor and replacement readers, strict lookup errors, and persisted mirror data provide useful foundations. Extend these rather than creating a second unrelated routing system.
- `pj-agdev/agobserver` already uses delivery identity and read-back after uncertain sends; Front's routine delivery recovery and rendering jobs offer related recovery patterns. Reuse the applicable ideas without importing role-specific workflow into the generic listener.
- `devdocs/README_DEV.md` documents callback semantics and conversation context. `pj-agdev/agfront/uv.lock` and other consumer locks pin pyagag; deployment verification must check the versions actually imported.
