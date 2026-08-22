# agag_builder p3 — step 2: Zulip provisioning client

## Done

pyagag commit `eed0e5e` adds the three owner-side operations required by
provisioning:

- `ZulipClient.create_bot(full_name, short_name)` creates a generic bot and
  returns `user_id`, `email`, and `api_key`.
- `ZulipClient.user_by_email(email)` checks both the owner-visible
  `delivery_email` and the realm-visible `email` field.
- `ZulipClient.update_channel_description(stream_id, description)` updates
  an existing channel by id.

`create_bot` handles both known Zulip response shapes. A complete creation
response is returned directly. If the response contains only the user id,
the client fetches the profile and regenerates the new bot's key. Missing
identity or credential fields fail explicitly. The docstring records that a
caller must perform the existence guard before calling it, because key
regeneration is unsafe for an already-running bot.

## Verification

- Focused Zulip tests: 95 passed.
- Full pyagag suite: 405 passed.
- Added fake-call coverage for the complete response, both fallback calls,
  malformed responses, delivery-email lookup, and channel-description
  updates.
- The commit was pushed to GitHub so downstream lockfile upgrades can resolve
  the implementation from the canonical deployment source.
