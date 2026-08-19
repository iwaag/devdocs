# Report — channel folder

Checked whether Zulip channel folders are ready to adopt as the standard
grouping for a project's main channel + its derivative channels, per the
braindump. No code was changed — this is the requested check plus a one-line
pointer added to `pj-agdev/.local/devenv.md`.

## The feature exists and is live, unused

Zulip's Channel Folders is a real server feature (new in Zulip 11.0, feature
level 389), and this deployment is on it: Zulip 12.2, feature level 500
(`GET /api/v1/server_settings`). Confirmed live against the running instance:

```
GET https://agstudio.local:8543/api/v1/channel_folders
→ {"result":"success","msg":"","channel_folders":[]}
```

The API works; zero folders have been created so far.

`POST /users/me/subscriptions` — the exact endpoint
`agag.zulip.ZulipClient.create_channel` already calls
(`pyagag/src/agag/zulip.py:297`) — accepts a top-level `folder_id` to drop a
newly created channel straight into a folder at creation time (per the
server's own OpenAPI spec, confirmed inside the running container at
`/home/zulip/deployments/current/zerver/openapi/zulip.yaml:13218`). No
separate "move channel into folder" step is needed.

## Current implementation gap

- `create_channel` (`pyagag/src/agag/zulip.py:297`) has no `folder_id`
  parameter — adding one is a small, additive change whenever this is acted
  on.
- No helper wraps `POST /channel_folders` (folder creation) anywhere in
  `pyagag`.
- `create_channel` itself is currently **uncalled** anywhere in `pj-agdev`
  (`grep` across the workspace finds only its definition). Today's
  `pj-<slug>` project channels (`PROJECT_CHANNEL_PREFIX` in
  `agautolab/src/agautolab/zulip_listener.py:95` and
  `agautolab/src/agautolab/project_archive.py:48`) aren't provisioned through
  this client path — wiring folder placement into `create_channel` won't take
  effect until/unless project-channel creation actually runs through it.
- Today's model is **one channel per project + topic-per-unit-of-work inside
  it** (mission topics: `agautolab/src/agautolab/mission.py:133-140`;
  `#FreeForge`'s `create-*` topics: `pj-agdev/.local/devenv.md:201-215`).
  There's no existing "derivative channel" concept — topics already scale
  cheaply within one channel. Before writing folder-placement code, decide
  what a "derivative channel" actually is (a second `pj-<slug>-*` channel?
  a per-agent channel?), since if the answer is "more topics," a channel
  folder may not even be needed yet.

## Recommendation

Not urgent — nothing currently creates channels through the one code path
that would need the `folder_id` wiring, so this is a "ready when needed"
gap, not a bug blocking anything today. When a second Zulip channel per
project is actually decided:

1. Add `folder_id: int | None = None` to `create_channel`.
2. Add a small `ensure_channel_folder` helper wrapping `POST
   /channel_folders` (idempotent: `GET /channel_folders` first, since the
   list is already fetchable and empty today).
3. Decide folder-per-project vs. one shared "Projects" folder before coding
   either.
