# slack_connect p1 — Step 2 report: the adapter and shared gaps

Date: 2026-10-19 (JST). agdevworld `3095992` (previous `248726b`).

## What was added

### `agdevworld/integrations/slack` (`slack-front`)

An independent Node package (Node 26, TypeScript run directly, `node:sqlite`,
`@slack/socket-mode` 3.1.0, `@slack/web-api` 8.2.0). Modules:

| Module | Role |
| --- | --- |
| `config.ts` | validated configuration (tokens, Developer member ID, optional workspace, WFE base, phone link base, LAN→phone origin map, trial author); placeholders rejected |
| `store.ts` | threads, envelope ids, inbox, Slack↔WFE message map, outbox, meta |
| `bridge.ts` | classification, inbox → WFE, WFE → outbox, delivery, unknown-send reconciliation, backfill and sweep, `!status`/`!help` |
| `format.ts` | Slack markup → plain text; Front Markdown → mrkdwn; phone-reachable URLs; deterministic splitting that keeps code fences balanced |
| `slack.ts` | the Web API calls (no SDK retries), failures sorted into `rate` / `refused` / `unknown` |
| `wfe.ts` | the conversation API as a non-browser client (no `Origin`, `via`) |
| `main.ts` | `run`, `check`, `status`, `threads`, `deliveries`, `resolve`, `inbox`, `retry`, `backfill`, `trial`, `fault` |

Behaviour, briefly (details in the package README):

- **Identity and routing.** Only `message.im` events of the configured
  workspace from the configured member become input. A root is a new
  conversation (id minted from the root's time); a thread reply uses the
  root's mapping however recent other threads are. The transport never sets
  `re`; Front's own interpretation and clarification apply.
- **Deduplication.** Inbox row per Slack message; envelope ids remembered;
  WFE submit token `slack:<team>:<channel>:<ts>`.
- **Input boundaries.** Bot messages (this bot included) are ignored; another
  person, an edit, a deletion and an attachment-only message get an
  explanation in the thread and start no turn; a caption is passed on with a
  note that the attachment was not.
- **Presentation.** Replies labelled 返答 / 質問 / 進捗 / 結果 with their run
  linked through the tailnet name; questions say how a thread reply is read;
  results, questions and split answers carry a Front Desk link. Front Desk
  words of the person are mirrored, labelled; Slack-originated words are not
  echoed (`via` and the message map). A newer message of a run coalesces its
  queued progress. The receipt is a 📥 reaction (WFE has it, the turn was
  requested), distinct from Front's replies.
- **Reliability.** Store before ack; inbox in Slack order with backoff; cursor
  advance and enqueue in one transaction; ordered delivery per thread with
  per-channel spacing and `Retry-After`; unknown outcomes reconciled by
  metadata key or bot/text before a resend (≤ 3 sends), otherwise left visible
  with `resolve`; backfill on each connection from the last live moment with
  a 5-minute overlap, plus a 15-minute sweep; 90-day free-plan gap recorded.
- **Trials.** `slack-front trial` posts a visibly marked bot message and
  records the WFE message as `Omni Agent (for Developer)` via `slack-trial`,
  never as the Developer.
- **Fault injection.** `slack-front fault lose-next-response` makes the next
  successful post look unanswered, to exercise reconciliation on the live app.

### WFE shared support (the only API change)

- Person messages keep `via` (conversation store schema v2, additive
  migration). `POST …/messages` accepts `via` and an optional `author`
  (default: the configured person); `wfe conversation post --via`; `show`
  prints it; the Front Desk shows `iwaag (via slack)`.
- Documented in `workflow_editor/docs/conversations.md`. Front's guide is
  unchanged (transport-independent).

### Release and service

- `scripts/slack-release.sh build|activate|deploy|status` — releases under
  `<area>/slack/releases/<commit>`, `current` symlink, health = a new pid with
  a Slack connection. Independent of `wfe-release.sh`.
- launchd job `com.agdev.slack-front` prepared (plist in the ignored
  `pj-agdev/.local/launchd/`), not yet loaded: it needs the tokens.
- Nautobot: service `slack-front` and placement `slack-front-agstudio`
  (`manual_toolchain`, `process_pattern`) committed via `nctl desired apply`
  (dry run: 2 creates, 0 conflicts; then committed).

## Verification

| Check | Result |
| --- | --- |
| `workflow_editor npm run check` (tsc, 421 tests, vite build) | pass |
| agdevworld `tsc --noEmit` (Front Desk) | pass |
| `integrations/slack npm run check` (tsc, 6 transport tests) | pass |
| WFE release `3095992` built, checked and activated | service and `bin/wfe` report `3095992` |
| Conversation store after activation | `user_version` 2, 29 messages kept, `GET api/conversations` answers |
| Slack release `3095992` built and checked | `slack/releases/3095992…` |

The transport tests run against a fake Slack and a fake WFE and cover: a
redelivered envelope, a backfilled copy and a lost WFE answer each giving one
message and one turn; concurrent threads; Front Desk mirroring without echo;
queued output surviving a restart; bots, strangers, edits, deletions and
attachments; unknown sends found by metadata (no resend), absent (one resend)
and unreadable (left unknown); backfill of a missed root and thread reply in
Slack order while WFE is down; balanced, deterministic splitting.

## Notes

- The first activation attempt of the WFE release was refused by the
  automated permission classifier; the Developer then approved it manually,
  and it was activated.
- Live checks of the scopes, metadata behaviour and notifications wait on the
  Slack app (report1, "Provisioning").
