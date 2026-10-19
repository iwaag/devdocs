# slack_connect p1 — Slack as a Front conversation surface: report

Date: 2026-10-19 (JST). Steps: `report1.md` (inspection and connection),
`report2.md` (adapter and shared gaps), `report3.md` (deployment and trials).

## Outcome

The Developer talks with Front from the Slack app, including on the iPhone,
through direct messages with a private Front bot. A new DM is a new WFE
conversation; replies in its thread continue it. Front's replies, questions,
progress and results come back in the thread with their run. The same
conversations are in the Front Desk. Text works without the home network or
VPN. Run, artifact and desk links open over Tailscale. Free plan; no paid Slack
feature, Slack AI or Marketplace listing.

## Revisions

| Repository | Commits |
| --- | --- |
| agdevworld | `3095992` integration + WFE `via`/`author`; `d23ccbb`, `3b1dd5b`, `61778d4` live fixes |
| pj-agdev | `c935e02`, `83c2dab` (agdevworld pins) |
| devdocs | `a93d271` report1, `6452207` report2, `9157ddf` README_DEV, this step's reports |

Deployed: WFE `3095992`, web container from `3095992`, `slack-front`
`61778d4` (launchd `com.agdev.slack-front`), Nautobot service `slack-front`
(converged, applied).

## What was built

- `agdevworld/integrations/slack` (`slack-front`): Socket Mode adapter with a
  SQLite transport store (threads, inbox, message map, outbox). It stores an
  event before acknowledging it, keeps one WFE message per Slack message
  (submit token from the Slack message), polls linked conversations with
  `since`, delivers in order with rate limits, reconciles unknown sends by
  message metadata before any resend, backfills missed DMs on reconnection and
  every 15 minutes, and explains edits, deletions, attachments and other
  people without starting turns. Operator commands: `status`, `deliveries`,
  `resolve`, `inbox`, `retry`, `backfill`, `trial`, `fault`.
- WFE: person messages record `via` (store v2). The HTTP post accepts `via`
  and a trial `author`, and the Front Desk names the way a message came.
  Front's guide is unchanged.
- `scripts/slack-release.sh`: its own release, independent of WFE.

## Evidence summary

All scenarios in the plan's table were exercised live (details in
report3). Highlights:
- A real Front question was answered in-thread and the work continued.
- Two concurrent threads stayed apart.
- A Front Desk-route message was mirrored once.
- A simulated lost Slack response was reconciled, with no duplicate.
- Messages sent from the iPhone while the service was down were recovered.
- An image, an edit and a deletion from the iPhone were explained and started
  no turn.
- Long answers were split readably.
- Restarting the integration mid-run did not affect WFE or the run.

The iPhone checks are the Developer's report from their device: text with VPN
off, dictation, a background notification, and links over Tailscale. Slack
history and the transport store agree with that report.

Checks: `workflow_editor npm run check` (421 tests), `integrations/slack npm run
check` (7 transport tests), agdevworld `tsc`.

## Limitations

- One workspace and one Developer, DMs only. Channels, other users, file and
  audio-clip transfer, and bridging of existing browser-only conversations are
  deferred.
- Slack delivery is at-least-reconciled, not exactly-once: an unknown post
  that cannot be found in a readable thread is resent at most twice, and then
  left for the operator.
- Backfill covers the DM history the bot can read. The first start imports
  nothing earlier, and messages hidden by the free plan's 90-day limit cannot
  be recovered (recorded as a gap).
- Detailed links need the phone's Tailscale. Text does not.
- Front's own limits remain: one conversation slot, so a long Front turn delays
  the next reply. Slack shows that as waiting, not as a transport failure.
- A message from another Slack user and a captioned attachment were verified
  only by tests.

## Recovery

Defined in `agdevworld/integrations/slack/README.md` ("Operations"):
- `slack-front status` shows the connection, WFE, counts, unresolved
  deliveries and the last backfill.
- `deliveries`/`resolve` settle unknown posts.
- `inbox`/`retry` handle refused input.
- `backfill --from` re-reads history.
- `launchctl kickstart -k gui/$(id -u)/com.agdev.slack-front` restarts the
  integration alone.
- Rollback: `scripts/slack-release.sh activate <area> <commit>`. For WFE,
  activate `248726b` and restore the conversation store backup
  (`.local/agdev/.local/backups/slack-p1-predeploy-*`).

## Interventions

- The automated permission classifier refused the WFE release activation; the
  Developer approved it manually.
- Account-only action by the Developer: creating the Slack app from the
  manifest, issuing its tokens and writing them to the ignored configuration.
