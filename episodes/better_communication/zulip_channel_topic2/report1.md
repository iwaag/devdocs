# Step 1 report — firing-rule spike (curl only)

Date: 2026-08-12. Result: **routing proven exactly as planned; prefix-only,
channel-agnostic filtering holds with zero permission or code changes.**

## What was done, in order

1. **Channel creation by the autolab bot.** As Autolab Agstudio (user 11,
   `agautolab/.local/zulip.env`): `POST /api/v1/users/me/subscriptions`
   created `#pj-spike` and subscribed Developer (8), Devworld Assistant (10),
   Autolab Agstudio (11), Autolab Agautolab1 (12), Forge (13), Cagent (14) in
   one call — same realm defaults that let the assistant bot create
   `#FreeForge` in the prior episode.
2. **Negative firing on `mission-*`.** Posted message id 75 under topic
   `mission-20260812-222107-spike`. The agforge listener log
   (`agforge/.local/out/zulip-listener.log`) stayed at its baseline line
   count; the cagent listener log
   (`pj-clusterintent/.local/cagent-window/zulip-listener.log`) showed no
   events after its 09:01Z queue registration. Nobody reacted — the
   `mission-` prefix is currently unclaimed, as intended.
3. **Positive firing on `create-*` in the same channel.** Posted message id
   76 under `create-20260812-222159-spike2`. agforge fired within a second
   (`chat run a4513a3c…`, channel='pj-spike'), answered in-topic, profile
   `sonnet`, $0.1037, 7.4 s. One paid run, accepted per the budget rule.
   This confirms channel-agnostic prefix routing as a **feature**: agforge
   needed no subscription-list or allowlist change to serve a brand-new
   channel. No channel allowlists needed — Step 1 does not disprove the
   plan's routing rule.
4. **Resolution stops matching.** `PATCH /api/v1/messages/{75,76}` with
   `topic=✔ <topic>`, `propagate_mode=change_all`,
   `send_notification_to_new_thread=false` — both succeeded (autolab bot
   resolving agforge's reply-bearing topic included, i.e. foreign-message
   topics are resolvable by a default-role bot). A follow-up message (id 81)
   posted into the resolved `✔ create-…` topic triggered no run over a
   2.5-minute watch.

## Notes for the next steps

- curl pattern identical to the FreeForge spike: HTTP Basic with the bot
  env-file credentials, `-k` for the self-signed cert,
  `--data-urlencode` per param.
- cagent's listener is DM-only (`partners=[…]` window requests only); it was
  silent for both topics, as expected.
- Leftovers kept deliberately: `#pj-spike` with topics
  `✔ mission-20260812-222107-spike` and `✔ create-20260812-222159-spike2`
  stays as evidence and as the Step-4 live-smoke target.
- Deus Ex Machina note: did the firing-rule spike for the autolab agent —
  handoff candidate.
