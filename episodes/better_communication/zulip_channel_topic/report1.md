# Step 1 report — manual spike (curl only)

Date: 2026-08-12. Result: **every mechanic works with the realm's default
permissions.** No realm setting had to change; the plan's fallback knobs
(`can_create_public_channel_group`, topic-move limits) were never needed.

## Discovered ids

| user | id |
|---|---|
| Developer (owner) | 8 |
| Omni Agent bot | 9 |
| Devworld Assistant bot | 10 |
| Autolab Agstudio bot | 11 |
| Autolab Agautolab1 bot | 12 |
| Forge bot | 13 |
| Cagent bot | 14 |

## What was proven, in order

1. **A generic bot can create a public channel.** As Devworld Assistant:
   `POST /api/v1/users/me/subscriptions` with a new `subscriptions` name and
   `principals=[8,10,13]` created `#FreeForge` (standing channel) and the
   throwaway `#create-20260812-spike-0001`, subscribing Developer, assistant,
   and Forge in one call.
2. **Posting**: `POST /api/v1/messages` with `type=stream`, `to=<name>`,
   `topic=request` — message id 52 in the throwaway channel, id 53 as the
   announcement in `#FreeForge` (`#**channel-name**` renders as a channel
   link).
3. **Forge can read the topic narrow** with
   `narrow=[{"operator":"channel",...},{"operator":"topic",...}]` — the
   channel analog of `dm_history` needs nothing new server-side.
4. **A bot can resolve a topic**: `PATCH /api/v1/messages/{id}` with
   `topic=✔ <topic>`, `propagate_mode=change_all`,
   `send_notification_to_new_thread=false`. Works on its own topic (id 52)
   **and on a topic containing another sender's messages** (`mixed`, ids
   54–56, forge's reply included) — so the assistant can resolve after
   agforge answers.
5. **Developer search across public channels** finds both the resolved topic
   and the announcement:
   `narrow=[{"operator":"channels","operand":"public"},{"operator":"search","operand":"spike"}]`.

## Notes for the next steps

- curl pattern used throughout: HTTP Basic with the bot's email/API key from
  its env file, `-k` for the self-signed cert, `--data-urlencode` for every
  param (JSON values as strings).
- PATCH with a form body works fine through curl; `agag.zulip.call()` only
  sends bodies for POST/DELETE today — Step 2 must let PATCH carry a body
  too.
- `GET /realm` is not a real endpoint (harmless empty probe); the effective
  policy statement is simply "creation succeeded as a default-role bot".
- Leftovers kept deliberately: `#FreeForge` is the real standing channel;
  `#create-20260812-spike-0001` (topics `✔ request`, `✔ mixed`) stays as
  evidence, consistent with the workflow's own rule that channels are never
  archived.
