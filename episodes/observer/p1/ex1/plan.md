# Observer p1 ex1 — stable conversations and retryable lookup failures

## Goal and scope

Fix the three defects reproduced in the p1 review:

- A destination accepted as `<channel>/<topic>` follows a reused name into an unrelated conversation.
- Renaming a watch topic before resolving it prevents cancellation from being recognized.
- A temporary destination-read failure ends the watch as `undeliverable`, losing the notification.

Use message IDs for identity and names for display. Distinguish confirmed absence/closure from an unsuccessful lookup. Keep p1's local-model evaluation, fixed interval, and one-shot notification behavior.

This is a breaking fix in a private experimental environment. Old record formats need no compatibility or migration; retire/recreate experimental watches as needed. Internal types, helper placement, and implementation order are discretionary. Security hardening, new approval flows, advanced scheduling, and a general messaging framework are outside scope.

## step1 — Anchor every accepted destination

- Continue accepting message links and `<channel>/<topic>`. At intake, resolve either form to a concrete message ID in the intended conversation and persist that ID in the accepted Zulip record. Keep the original text for explanation.
- Use the stored ID for every subsequent destination lookup, including after local-store loss and restart. A name reused by another conversation must not become a fallback destination.
- If intake cannot currently verify a destination, keep the request unaccepted and explain the lookup problem. Ask for a different destination only when the input is invalid or the destination is confirmed absent/closed.

Verify through intake and delivery: accept a name, rename that destination, reuse the old name, and deliver only to the original conversation. Repeat after reconstructing the local store from Zulip. Message-link input follows the same stored representation.

## step2 — Follow the watch's own anchor

- Resolve the existing watch ID to its current conversation before observing and again before notifying. A confirmed ✔ cancels; a confirmed deleted anchor ends the watch locally. A failed lookup leaves it pending and skips this attempt.
- Use the current conversation for failure reports, state notes, completion posts, and resolution. Refresh cached names as needed; matching lifecycle state alone is not evidence that the cached location is current.
- Recover renamed active watches by their existing IDs without creating another watch. Once cancelled or removed, they stop consuming evaluations and notifications.

Verify: rename a watch and continue it; rename then resolve it and confirm no evaluation or notification occurs; reuse its old name and confirm that conversation receives no state or completion posts. Also exercise cancellation during evaluation and a transient watch-anchor lookup failure followed by recovery.

## step3 — Retry uncertain delivery without re-judging the condition

- Make destination resolution distinguish **open**, **confirmed absent/closed**, and **lookup failed**. A small result type or explicit exceptions are both fine. Inspect the actual shared client's error behavior so transport errors are not accidentally translated into absence.
- Only confirmed absence/closure produces terminal `undeliverable`. On lookup or send failure, retain the met result and pending notification, record the reason, and retry at the ordinary interval. An unresolved failure for one watch should not prevent other watches from progressing.
- Apply the same distinction to notification read-back: failure to check for an earlier delivery is not proof that none exists. Skip sending and retry when read-back fails.
- Preserve ordinary duplicate suppression and restart recovery. A general exactly-once guarantee is unnecessary; describe the bounded read-back and any remaining race windows accurately.

Verify: destination-read failure, read-back failure after an ambiguous send, and send failure all recover to one notification with no further model evaluation. Confirm deletion/closure remains terminal. Include a restart while delivery is pending.

## step4 — Validate, deploy, and report

- Add the reproductions above as focused regression tests and run the existing Observer suite. Prefer controllable client failures over disrupting shared services.
- Inspect current service state with Nautobot/nctl and the ignored environment notes, then deploy the fix using the existing launchd conventions. Check for in-flight work before restarting.
- Demonstrate one anchored delivery across a destination rename and one cancellation after a watch rename in the actual runtime. Keep transient-error verification in controlled tests; another model-quality benchmark is unnecessary.
- Update the introduction and relevant documentation to describe the final contract. Correct p1's claims about rename handling and retry behavior by pointing to this fix and its evidence.
- Write `report.md` with changed behavior, test/live evidence, and remaining limitations. Keep machine details and detailed local evidence in ignored files. Commit/push changed repositories and any necessary dependency pointers.

## Implementation hints

- Main code: `pj-agdev/agobserver/src/agobserver/{intake,destination,anchor,worker,notify,store}.py`. The current destination is persisted as raw text in `intake.py`; normalization must reach the Zulip accepted record, not just the local cache.
- `worker.cancelled()` currently compares cached topic names. `reconcile()` skips updating records when state and acceptance are already present. Both contribute to stale watch locations.
- `destination.resolve()` currently returns an absent-looking result for a named destination's read exception; `notify.deliver()` then clears its pending flag. `already_delivered()` also returns the same value for read failure and no matching post.
- `pyagag/src/agag/zulip.py` supplies `message()` and `conversation_of()`. Check the installed dependency as well as the shared checkout when deciding where error classification belongs; update dependency pins if shared code changes.
- Existing `tests/{conftest,test_destination,test_lifecycle,test_notify}.py` provide a fake Zulip client. The existing destination-rename test uses a message ID from the outset, so it does not cover normalization of name input at intake. Extend that boundary explicitly.
