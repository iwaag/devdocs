# slack_connect p1 — Step 3 report: deployment and the experience

Date: 2026-10-19 (JST). Host names, Slack IDs and raw evidence:
`pj-agdev/.local/devenv.md` ("slack_connect p1 on this host") and
`pj-agdev/.local/slack-connect-p1/`.

## Deployed

| Component | Version / state |
| --- | --- |
| WFE (service and `bin/wfe`) | agdevworld `3095992` (conversation store v2, `via`) |
| Front Desk (web container) | rebuilt from `3095992`; Slack words shown as `iwaag (via slack)` |
| Slack integration | `slack-front` release `61778d4`, launchd `com.agdev.slack-front` (KeepAlive), Socket Mode connected |
| Slack app | "Front", private, one workspace, created by the Developer from `manifest.yaml` |
| Catalog | Nautobot service `slack-front`, placement `slack-front-agstudio`; `nctl observe agstudio` then `nctl drift --host agstudio`: converged, `slack-front-agstudio=applied` (fresh observation at the end of the step) |

Releases during the step: `3095992` → `d23ccbb` (unknown-send matching
tolerates Slack's stored text) → `3b1dd5b` (a reply with no run names its
author) → `61778d4` (tombstone deletions; live mark after reconnection). Each
came from a live finding below.

## Live contract checks

- `auth.test`, `conversations.open` (`im:write`), Socket Mode
  (`connections:write`), `message.im` events, `conversations.history/replies`
  (`im:history`), `chat.postMessage` and `reactions.add` all work with the
  manifest's scopes; no further scope was needed.
- **Message metadata works with the bot token**: every post carries its
  delivery key and `conversations.replies(include_all_metadata)` returns it.
  Delivery identity is therefore the metadata key; the text fallback is
  unused so far.
- **Slack rewrites stored text**: emoji become `:shortcodes:` and `&` is
  escaped. The text fallback compares normalised text (`d23ccbb`).
- **Deleting a root that has replies** arrives as `message_changed` with a
  `tombstone` message, not as `message_deleted` (found on the iPhone; fixed in
  `61778d4`, re-checked on the iPhone).

## Trials

Automated trial messages were posted by the bot marked "🧪 試験メッセージ（Omni
Agent (for Developer) が送信。開発者本人の発言ではありません）" and recorded in
WFE as `Omni Agent (for Developer)` via `slack-trial`. The Front Desk-route
trial was posted over HTTP with that author. The Developer's own messages
were sent from their iPhone.

| Scenario | Evidence |
| --- | --- |
| New root and thread reply | A: one conversation, reply `front/converse/run-014` in its thread, Slack 23 s after the trial; C: root + two thread follow-ups → three turns (`run-016`, `run-018`, `run-024`), each reply in C's thread |
| Two concurrent threads | B and C posted 3 s apart: separate conversations and runs (`run-015`, `run-016`); replies landed in their own threads |
| Question and answer | B: Front asked a real question (`q1`, options A/B/C) which reached Slack labelled 質問; the thread answer "B でお願いします。" started `run-017`, which read it as the answer and delivered the summary; `run-015` was continued by it. The transport set no `re` |
| Front Desk continuity | D (HTTP, author Omni): mirrored once into C's thread as "🖥 Omni Agent (for Developer)（開発者本人ではありません）（Front Desk から）", Front's answer (`run-019`) in the same thread; no Slack message was echoed back (inbox/outbox: 0 duplicates). Slack conversations open in the Front Desk under the tailnet name (screenshot `shots/frontdesk-trial-c.png`) |
| Retry and restart | Envelopes are acked after the store write; redelivery/backfill/lost-answer dedup covered by tests; four service restarts during the step left no duplicate WFE message (each Slack message has one WFE message and one `turn` link) |
| Unknown Slack send | `fault lose-next-response` on B's answer: the post's outcome became unknown, 16 s later it was found in the thread by its metadata key and marked sent; one send, no duplicate |
| Disconnection | Service stopped (`launchctl bootout`); the Developer sent a new root and a reply in C's thread from the iPhone; on restart the backfill from the last live moment recovered both (2 messages), routed to a new conversation and to C, and Front answered both (`run-023`, `run-024`) |
| Input boundaries | Bot echoes: every bot post (30 rows) ignored. From the iPhone: edit → ✏️ notice; deletion of a root with replies → 🗑 notice; image only (`file_share`, image/jpeg) → 📎 notice; none started a turn. Other users and captioned attachments: covered by tests only (no second person in the workspace) |
| Delivery | Japanese throughout; a 3,114-character Front answer in one post; an 8,458-character trial reply split into 3 parts (3,448 / 3,468 / 1,564), fences balanced, numbered, desk link on the last; run links and the Front Desk link use the tailnet name and answered 200; LAN links in replies are mapped (`linkMap`) |
| iPhone | Reported by the Developer: with Tailscale off, a dictated message was sent and answered; a background notification of Front's reply appeared; `!status` answered; with Tailscale on, a run link and the Front Desk link opened. Their dictated text ("音声テストです。今は何時ですか？") is in WFE as `iwaag via slack` |
| Service isolation | G: `com.agdev.slack-front` killed and restarted (`kickstart -k`) while `run-022` was executing; the run completed 11 s later and its reply was delivered once after the restart. WFE (pid 47667) and the executor (pid 34036) kept their pids through every Slack restart |

Transport store at the end: inbox 16 done / 30 ignored (bot posts and
non-input), outbox 33 sent, 0 unknown, 0 failed.

## Notes

- The Developer's two first device messages were brief ("テスト　今日は何曜日",
  "ok!") rather than the suggested wording; they are the real evidence and are
  kept as sent.
- An `:wave:` emoji message was the Developer's first "image" attempt; it is
  ordinary text and was passed to Front as such. The image test was then
  repeated with a photo.
- Not exercised live: a message from another Slack user (the workspace has
  one person), a captioned attachment, an audio clip, Slack-side outages
  longer than a restart, and the 90-day history gap. The transport tests cover
  the first two; the others are documented recovery paths.
