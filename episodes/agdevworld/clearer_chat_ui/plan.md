# Clearer Chat UI — Implementation Plan

## Goal and approach

Make progress reports, information/results, and requests for a human response distinguishable at a glance throughout agdevworld. Show both what each post means and which requests still await a response.

Start with pyagag's shared posting contract and a complete Front Desk implementation, then extend it to the other conversations. This is an experimental, unpublished environment in a breaking-change phase: backward compatibility is unnecessary. Replace obsolete contracts and update consumers together; migration adapters and historical backfills are not required. Implementers may choose the exact schema, module boundaries, and visual treatment.

## Useful findings

Paths below are relative to the workspace containing the projects.

- `pyagag/src/agag/reply.py`: `ag-reply` selects text for publication, but carries no intent. Its shared guide is attached to conversational roles through `prompt_with_guide(..., reply=True)`.
- `pyagag/src/agag/topics.py`: `requester_of` binds a response to the input actually processed; `mention_of` addresses that requester, including humans. A mention alone does not mean a response is required.
- `pyagag/src/agag/serving.py` and `delivery.py`: replies are prepared and journaled before delivery. Extend this path so retries preserve intent and do not rerun the agent.
- `pyagag/src/agag/selfnote.py`: progress detection currently recognizes posts composed of lines beginning with `🔧` or `💬`. Ordinary prose progress reports have no equivalent classification.
- `pyagag/src/agag/trace.py`: `awaiting_human` currently means the agent answered last in a human conversation. It is an inference, not an explicit request.
- `pj-agdev/agdevworld/agentroom/src/agentroom/frontdesk.py`: Front Desk groups agent answers under `answered`. The relay already reads a persisted Zulip mirror and exposes stale/unknown states.
- `pj-agdev/agdevworld/src/roomState.ts` and `frontDeskPlayback.ts`: Front Desk and Arguing Room share post types and playback. Character dialogue is a rendering of source posts, linked by message ID through `agentroom/src/agentroom/presentation.py`.
- `pj-agdev/agdevworld/src/chatPanel.ts`: routine chat has a separate rendering path that also needs the new labels.

## step1 — Define and implement the shared post contract

Add structured intent to conversational output and `agentchat send`, with common parsing and validation in pyagag.

- Begin with `progress`, `report`, and `response_request`. A response request identifies its intended recipient by user ID; question/confirmation wording can be a small optional distinction.
- Keep semantic intent separate from sender identity, transport acknowledgements, and execution status. An unclassified post must not imply a response request.
- Persist metadata with the source Zulip post. Prefer one delivery containing text and metadata, avoiding a second write that could be lost independently. Choose the encoding after checking existing block parsing and message rendering.
- Preserve metadata through reply repair, journal preparation, retries, and restart recovery. Keep machine fields out of human-facing text and character dialogue, while exposing their meaning to agents reading the conversation.
- Teach the shared reply guide and CLI help once, with concrete examples. Do not duplicate the contract across role guides.

Done when the parser and posting/delivery tests cover all intents, missing or malformed intent, nested fences, and retry recovery without lost metadata or duplicated posts. Document the selected wire format and examples beside the implementation.

## step2 — Derive outstanding response requests

Implement a shared read model from source messages and their metadata. Give each request a stable identity based on its message ID; use existing conversation anchors to follow renames.

- Preserve the original post's intent after a response arrives. Separately track whether the request is pending, has received a response, or was withdrawn/superseded/closed.
- Define response correlation explicitly. An explicit reference to the request is strongest; a simple next-post rule is sufficient only when the recipient and pending request are unambiguous. Multiple pending questions must not all disappear because someone spoke.
- Response receipt does not establish approval, task acceptance, or completion. Leave those decisions to the existing agent and acceptance workflows.
- Progress posts, acknowledgements, and unrelated agent messages do not settle a request. Human input arriving during an agent run must be considered against the processed-input boundary so a late-delivered question does not create a false wait.
- Reconcile `trace.py` with the explicit model. Distinguish inferred legacy turn-taking from a confirmed response request, or remove the obsolete inference where appropriate.
- Define behavior for edits, deletion, conversation closure, incomplete history, and stale mirrors. Expose uncertainty when available evidence cannot establish the current state.

Done when focused history fixtures demonstrate these transitions, including two outstanding questions, a third-party interruption, and restart/reconstruction from the same recorded history.

## step3 — Deliver the Front Desk experience

Update Front to produce the contract, extend the relay response, and carry the new fields through the shared room model and playback.

- Label posts with compact text and icons: progress, report, and a question/confirmation addressed to the user. Use restrained progress styling and a prominent response-request treatment; color alone is insufficient.
- Show current outstanding requests near the conversation header or composer, with a count and navigation to each source post. Support explicit reply correlation when more than one request is pending.
- Apply the same meaning in original and character-dialogue views. Rendering inherits source metadata; the presentation model does not reclassify a question from rewritten dialogue.
- Keep historical playback and current conversation status distinct. A question visible in old dialogue must not resurrect a settled request.
- Keep execution and rendering activity separate from response requirements: an acknowledgement or a rendering delay is not a question for the user.

Done when a Front Desk conversation visibly distinguishes a progress report, an ordinary result, and a question; replying updates the current wait indicator while retaining the question's historical label. Verify both views and page reload using the existing demo/fixture facilities and a browser.

## step4 — Extend to other conversations and consumers

- Reuse the contract and read model in Arguing Room and routine chat. Review other agdevworld conversation surfaces for consistent labels and outstanding-request display.
- Update autolab, forge, cagent, and other affected pyagag consumers, including generated-agent guidance where relevant. Cover direct `agentchat send` posts as well as final `ag-reply` output.
- Update dependency pins and deployment inputs together. Check `pj-clusterintent` for cagent and deployed-agent dependencies; a Nautobot schema change is not expected.
- Preserve existing mention/callback routing and acceptance behavior. Intent metadata itself should not introduce additional agent runs or a new approval workflow.

Done when each affected posting path can emit the new contract and all updated conversation surfaces derive meaning from the same source information.

## step5 — Validate, deploy, and report

- Run focused pyagag contract/lifecycle tests, relay state tests, affected consumer checks, and the agdevworld frontend build. Use existing project test guidance for required checks.
- Verify an end-to-end sequence: human request → progress → question → human response → report. Include multiple questions, an agent interruption, input during execution, a restart, and stale-history display. Use fixtures for fault cases and a small live trial for actual posting and UI behavior.
- Before inspecting or changing running services, obtain their registered/observed state through pj-clusterintent's Nautobot or `nctl`, following `pj-clusterintent/nctl/README.md`. Use existing local environment notes for deployment and refresh the affected services and frontend bundle.
- Commit and push changes in the affected repositories, including necessary submodule pointer updates. Keep machine-specific deployment facts in ignored local notes.
- Write `report.md` here with the chosen contract, delivered behavior, validation evidence, and remaining limitations; update developer documentation where the old semantics changed.

Success means users can recognize a request for their response immediately, see which requests remain outstanding, and continue reading progress/results without being falsely told to reply. The same meaning survives delivery retries, reloads, and character rendering.
