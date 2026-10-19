# slack_connect p1 — Step 1 report: inspection and connection preparation

Date: 2026-10-19 (JST). Host-specific names, addresses, tokens and Slack IDs are
in `pj-agdev/.local/devenv.md` and `pj-agdev/.local/agdev/slack/`, not here.

## Observed state

| What | Observation | Kind / age |
| --- | --- | --- |
| Nautobot / nctl | `nctl status` ok: Nautobot 3.1.3, 1 celery worker, node dumps 2.4 h old (agstudio), submodules clean | fresh runtime check, 18:30 JST |
| WFE service | launchd `com.agdev.wfe-service` running release agdevworld `248726b` (p8) | fresh `launchctl list`; release from devenv notes |
| Executor | `com.agdev.agautolab-wfexec` running, `conversation` lane on worker `studio` | fresh `launchctl list` |
| Catalog | agstudio placements are `manual_toolchain` with a `process_pattern` (agautolab, agobserver); no Slack service yet | `nctl desired export`, fresh |
| Repositories | pj-agdev `870d15e`, agdevworld `248726b`, both clean before this episode | fresh `git status` |
| Tailnet | Tailscale on agstudio with MagicDNS; the iPhone is a tailnet member (offline at check time); `:8093/wfe/` and `api/conversations` answer 200 under the MagicDNS name | fresh curl from agstudio |
| Slack | no app, token or CLI on this host | fresh search |

## WFE paths traced

- **Write-origin check** (`server/api.ts`): a non-GET with an `Origin` header
  is refused unless same-origin or configured; a client that sends no `Origin`
  is a supported non-browser client (the CLI-equivalent path tests use). The
  integration is such a client and sends no browser headers.
- **Person attribution**: `POST api/conversations/<id>/messages` records the
  configured person (`conversations.json` `person`). The integration therefore
  validates Slack identity itself: only the configured workspace and Developer
  member ID are accepted as that person.
- **Shared gap closed in WFE** (agdevworld, uncommitted until Step 2's commit):
  - person messages record `via` (store schema v2, `ALTER TABLE` migration):
    the HTTP body, `wfe conversation post --via`, the view and
    `wfe conversation show` carry it, and the Front Desk names the speaker
    `iwaag (via slack)`. Slack-originated words are told apart from browser
    words in the authoritative record, and the integration can tell its own
    posts from browser posts when polling.
  - the HTTP body may name `author` (default: the configured person), as the
    CLI's `--author` already can, so an automated trial actor is recorded as
    itself rather than as the Developer.
  - A focused store test covers `via` and the v1→v2 migration; the existing
    conversation tests pass.
- **Reply cursor**: `GET api/conversations/<id>?since=<id>` returns messages
  with ids above the cursor; ids are global and monotonic, so a per-conversation
  cursor of the last relayed id is sufficient.
- **URL generation**: run URLs in the view use `zulip.json viewBase`, else
  `present.json answerBase`; both are the LAN `.local` name, which the iPhone
  cannot resolve over Tailscale. The integration builds its own links from the
  structured run name with a configured iPhone-reachable `linkBase` (the
  MagicDNS name, verified to answer) and maps only URLs in reply text whose
  origin is exactly the configured LAN origin. Front Desk:
  `<origin>/?view=frontdesk&conv=<id>` (the desk derives its WFE URL from the
  host it is opened under, so it works under the MagicDNS name too).

## Slack contract chosen

- **Connection**: Socket Mode with an app-level token (`connections:write`).
  Up to 10 connections per app; connections refresh every few hours; Slack
  does not promise to replay events missed while disconnected.
- **Acknowledgement**: the SDK (`@slack/socket-mode` 3.1.0) acks only when the
  handler calls `ack()`. The integration writes the envelope to its SQLite
  inbox synchronously, then acks (Events API deadline 3 s); model execution
  and WFE availability are never on the ack path. An event is listened to once
  (`slack_event`), not also through its typed emitter.
- **Subscription**: bot event `message.im` only (no `app_mention`, no
  `message.channels`): one route per input.
- **Bot scopes**, from the calls made:
  - `chat:write` — `chat.postMessage` (replies, notices, mirrored desk text);
  - `im:history` — receiving `message.im`; `conversations.history` /
    `conversations.replies` for backfill and send reconciliation;
  - `im:write` — `conversations.open` to find the Developer's DM at start;
  - `reactions:write` — `reactions.add` for the transport receipt (a
    reaction does not notify, unlike a receipt post).
- **History limits**: the 2025 `conversations.history/replies` limits
  (1 request/minute, 15 items) apply to commercially distributed
  non-Marketplace apps; an internal single-workspace app keeps Tier 3. Free
  plan: messages older than 90 days are hidden, so backfill beyond that is a
  recorded gap.
- **Delivery identity**: `chat.postMessage` `metadata`
  (`event_type: agdev_delivery`, payload: the delivery key), read back with
  `include_all_metadata`. The docs page currently says metadata needs an
  app-level token; this must be checked live. Fallback identity: same bot
  user, same thread, same text, posted after the attempt.
- **Rate limits**: one message per second per channel; SDK automatic retries
  disabled (`retries: 0`, rate-limited calls rejected) so that the outbox
  decides about `Retry-After` and about timeouts (unknown outcome).

## Packaging

`agdevworld/integrations/slack/`: an independent Node package (Node 26 runs
TypeScript directly, `node:sqlite` as in WFE, the same `tsc`/`node --test`
checks), with Slack's official SDK (`@slack/socket-mode`, `@slack/web-api`).
It talks to WFE only over HTTP. It is deployed from a release copy of an
agdevworld commit (`scripts/slack-release.sh`) under the area's
`.local/agdev/slack/`, run by its own launchd job, and registered in Nautobot
like the other agstudio processes (service + `manual_toolchain` placement).

## Recovery choices

- Inbox keyed by the Slack message (`team/channel/ts`), also storing the event
  id; the WFE submit token is `slack:<team>:<channel>:<ts>`, so a duplicate
  event, a backfilled copy or a crash after the WFE post is one message and one
  Front turn.
- Outbox rows are written in the same transaction as the per-conversation
  cursor advance; a row is `sending` before the call. A crash or timeout leaves
  `unknown`, reconciled against thread history before any resend; rows that
  cannot be reconciled stay visible with `status` and a `resolve` operation.
- Backfill on every (re)connection from the last time the connection was known
  live, with overlap; history of mapped threads uses `conversations.replies`.

## Provisioning

Prepared: `agdevworld/integrations/slack/manifest.yaml` and the ignored
configuration `pj-agdev/.local/agdev/slack/config.json` (mode 600, with
placeholders).

**Pending: an account-only action.** No Slack account access is available to
the agent (no configuration token, no browser session). The Developer was asked
to create the app from the manifest, generate the app-level token, install it,
and write the two tokens and their member ID into the ignored configuration.
All independent implementation continues meanwhile; live verification of the
scopes and of metadata behaviour waits on it.
