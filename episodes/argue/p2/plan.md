# Argue p2 — a human-led arguing room and separate character rendering

## Goal and approach

Source: [braindump.md](braindump.md). Build an Arguing Room in agdevworld, initially reusing the Front Desk, so a real human can start, read, and continue an argue. Keep the actual discussion free of character settings. Front reinterprets recorded speech into character dialogue in separate, non-reactive memo topics. Apply the same separation to the existing Front Desk.

This phase establishes an interface and evidence for later improvements to discussion quality. It does not prescribe a study-first sequence, fixed discussion rounds, or a test of how ambitious a project must be. Keep p1's participation and outcome behavior as the starting point; adjust facilitation when actual use gives a reason.

This is a breaking redesign in a private experimental environment. Backward compatibility and preservation of disposable experimental state are unnecessary. Replace obsolete paths instead of adding compatibility layers. Choose schemas, package boundaries, names, queue implementation, and deployment order freely. No new authentication, sandboxing, or production hardening project is needed. Keep secrets and machine-specific facts in ignored files.

## Step 1 — Make memo conversations non-reactive

- Establish a shared distinction between actionable conversations and presentation-only memos. A dedicated memo channel is a simple starting point: it can be recognized before its first message and remains recognizable through topic renames. Choose another representation if it is equally clear and reliable.
- Apply the same exclusion at ordinary intake, mention intake, restart/resync recovery, and execution-time eligibility checks. Already queued work must also respect it. Memo content, including actual mentions, commands, and copied metadata, must not start an agent or command handler.
- Cover consumers outside the shared listener, especially ComfyUI notifier's command intake and any independent cagent/recovery paths. Make mirror data available for display without treating memo conversations as work or obligations.
- Give memo-to-source links their own meaning. Avoid using `rootchat` as presentation provenance: it participates in callback routing and evidence discovery. Keep memos out of ordinary discussion context and `threads/`.

Verify with focused fixtures: plain text, a real bot mention, a notifier command, copied selfnotes, a rename/resolve, and restart with pending work all remain quiet in memos. An ordinary argue invitation still works.

## Step 2 — Separate Front's discussion and character-rendering runs

- Run substantive Front Desk conversations and argues without character lore or generated dialogue in their context. Human input is posted verbatim under the existing human identity; retain p1's human desire and argue anchor semantics.
- Give Front a separate presentation role. Its input is a snapshot of recorded source speech, relevant context, speaker identities, and one pinned settings revision. It produces character dialogue; the posting layer validates and saves it to the memo topic. The role needs no delegation or discussion-writing tools.
- Reuse the existing dialogue validator, source references, settings snapshots, and run records. Preserve factual claims, uncertainty, disagreement, names, and links when re-voicing speech. Identify archsage and `sage:<name>` separately even though they share an account; a missing character can use a plain speaker fallback.
- Trigger rendering from new source speech, including specialist contributions, independently of the substantive serving queue. Use a small durable job mechanism so slow or failed rendering does not block discussion. Coalesce bursts where useful; do not render acknowledgements or bookkeeping notes as dialogue.
- Save source anchor/message IDs, a source-content fingerprint, settings revision, and renderer version with each result. Repeated processing of the same job reuses its recorded result; retries must not rerun the discussion or delegation. Handle the crash between posting a memo and recording local completion by recognizing the existing result.
- Support explicit reinterpretation with another settings revision and retain the earlier result. An edited/deleted source must be identifiable as changed; choose a simple stale-result or regeneration policy. Neither browsing history nor restarting should cause unlimited paid rendering.

Verify: discussion runs receive no character settings or memo text; rendering receives the intended sources; a rendering failure leaves the discussion usable; retry/restart does not duplicate a result; another settings revision creates a distinct interpretation of the same sources.

## Step 3 — Expose argues and their interpretations through the relay

- Add the Arguing Room's list, create, read, post, and resume operations. Identify an existing argue by its anchor message ID and resolve its current location; do not make a renamed topic or reused display name become another conversation.
- Read source history, memo results, and generation status from the existing mirror. Reuse the relay's read/write credential separation and submission-token handling. Avoid a fresh Zulip history sweep for each browser refresh or rendering job.
- Submit human input directly into the source argue. Reopening resumes that discussion; it does not reopen or execute downstream projects/studies. Reuse existing completion behavior where it fits rather than adding another completion engine.
- Return enough provenance to switch between original speech and character dialogue, follow a rendered line to its sources, select a saved interpretation, and request reinterpretation. Surface pending/failed rendering and stale mirror data alongside the available history.
- Adapt the Front Desk API to the same source/memo relationship. Shared conversation/presentation helpers are sufficient; a general workflow framework is unnecessary.

Verify: creation, duplicate-submit handling, reload, anchor-following after rename, resolved history, and an unavailable renderer. Ordinary repeated reads should add no Zulip API calls once the mirror holds the conversation.

## Step 4 — Build the Arguing Room and update the Front Desk

- Reuse Front Desk portraits, dialogue playback, history, composer, and settings loading. Parameterize the room/background and data adapter where useful instead of maintaining two large copies of the scene.
- Add navigation to the Arguing Room, conversation creation/selection/resumption, and original/dialogue views. Let the human read every participant's contribution and reply directly without waiting for character rendering. Input always targets the source discussion, including when the character view is active.
- Make source references accessible from rendered lines. Show which interpretation is being displayed and offer explicit reinterpretation with current settings; older interpretations retain their pinned assets.
- Register the existing archsage character and argue background in the settings manifest. Check agent-to-character mapping against the published roster; retain logical sage labels in the view.
- Move the existing Front Desk to the same separated rendering flow and remove its obsolete combined judgment/character path. Old record formats need not be migrated.

Verify in the browser: human post, multi-speaker history, source lookup, pending/failed rendering fallback, room switching, reload/resume, and reinterpretation after a settings change. Check that extreme lore changes the presentation without entering the substantive run's inputs.

## Step 5 — Deploy, exercise with a human, and report

- Inspect current service state through Nautobot or `pj-clusterintent/nctl`, using ignored environment notes for local details. Update affected dependency pins and deployed listeners, including independent consumers that need the memo exclusion. Reuse existing services; no new agent account is required for Front's presentation role.
- Run focused tests in changed packages and the frontend build/browser checks. Demonstrate memo silence across a restart, normal argue participation, and rendering recovery. Use fixtures for routing faults and command probes; a broad benchmark campaign is unnecessary.
- Make the room available for a real human to start and lead a discussion, read an agent's contribution, reply, and later resume it. A project/study outcome is not required to prove this interface. Do not substitute Omni Agent posts under the Developer credential for evidence of human use. If the human session is pending, finish the deployable work and report that validation as pending.
- Record substantive versus presentation run counts/costs and enough API evidence to identify accidental replay or polling. Note usability observations, especially whether Front moves ahead before the human can contribute, as input to later argue improvements rather than imposing a new discussion protocol now.
- Update relevant developer documentation and ignored environment notes. Write `report.md` with delivered behavior, validation, human-session observations or pending status, and remaining limitations. Commit/push changed repositories and necessary submodule pointers.

## Useful implementation findings

- `pyagag/src/agag/listen.py`: `_intake`, `_consider`, `owed`, and `recover` are the main shared routing points. Recovery hooks and pending queue execution matter as much as incoming message filtering.
- `pj-agdev/comfynotify/src/comfynotify/commands.py` has independent live/catch-up intake. It deliberately accepts commands in resolved topics, so marking memos with a checkmark cannot provide silence.
- `pj-agdev/agfront/src/agfront/zulip_listener.py` currently gives `character_talk` both character settings and operational responsibilities in one run. `argue.py` already has a separate substantive role and `handoff=False`; use this separation as a starting point.
- `pj-agdev/agfront/src/agfront/dialogue.py`, `settings.py`, and `evidence.py` provide dialogue validation, pinned settings, and message-ID evidence. The existing Front Desk schema embeds dialogue in the substantive reply; replace that coupling.
- `pj-agdev/agdevworld/agentroom/src/agentroom/frontdesk.py` and `src/scenes/FrontDeskScene.ts`, `src/frontDeskState.ts`, `src/frontDeskSettings.ts` are the main reuse points. Audit Front-specific sender, channel, topic-prefix, and completion assumptions when adapting them to multiple speakers.
- At planning time, `agdevworld-settings/characters/archsage/` and `rooms/argue/bg.png` exist, but `manifest.toml` has neither entry. Merely adding the image files does not expose them through the settings service.
- [better_zulip_call p1](../../better_zulip_call/p1/report.md) documents the persisted mirror and API budget. Reuse the running mirror; avoid introducing another realm reader solely for rendering.
- [argue p1 report](../p1/report.md) records one live project outcome with Omni-authored human turns. It is useful integration evidence, but not yet evidence of human-led discussion quality.
