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

## Addendum (2026-08-30): the wiring exists now

The recommendation above is done, and the trigger was not a second channel
per project but a second *agent*. `agag provision` created every agent
instance channel unfiled, which stayed invisible while there were two of
them and became visible at eight: the developer looked for
`arxivsage-agstudio1` in the `agents` folder after `sage` p1 and it was not
there, along with `front`, both `agecho` channels and `agping-agstudio1`.

pyagag `ce99a68` implements steps 1–3 of the recommendation as
`agag.provision`: the folder is resolved by **name** (`AGENT_FOLDER =
"agents"`, minted if the realm has none — folder creation is not
idempotent, so the lookup comes first) and passed to `create_channel`.
Step 1 was already in place from this episode; what this adds is a caller,
plus `ZulipClient.set_channel_folder` for the case the original note did not
cover — `folder_id` files a channel only at **creation**, and provisioning
an instance whose channel already exists merely joins it. That method
(`PATCH /streams/<id>` with `folder_id`, Zulip 12.2 / feature level 500) is
also how a channel older than its realm's folders gets filed.

The question this episode left open — folder-per-project vs. one shared
folder — is still open for `pj-<slug>` channels, which remain outside this
path. Only agent instance channels are provisioned through `create_channel`.

## Addendum (2026-09-04): the open question closed, and the drift it caused

The developer asked whether the realm's folders looked right. They did not,
and the cause was the question this episode left open above: with no decision
between folder-per-project and one shared folder, `pj-` channels — which are
still filed by hand — were dropped into whichever folder was nearest.

What the audit found:

- **Folder 1 was named `pj-runsmoke2` and held seven projects.** 66 channels:
  `ghtrends`, `mediagen`, `papers`, `rtnotes`, `runsmoke2`, `studyarxiv` and
  `studynourl` with all their `work-` channels. Its description said
  `Project channels and their work channels` — shared-folder wording under a
  single-project name, which is the open question showing up as an artifact.
- Only `pj-simpleshooter` (folder 2) matched the documented standard.
- `pj-foodchain`, `pj-runsmoke1` and runsmoke1's four `work-r-*` channels were
  unfiled.
- **One mis-filed `pj-` channel drags a fleet.** `ensure_work_channel`
  (`agautolab/src/agautolab/zulip_listener.py:436`) inherits the parent's
  folder deliberately, so `rtnotes` alone accounted for 20 channels. The
  inheritance was working exactly as written — it faithfully propagated a
  wrong parent, and faithfully left runsmoke1's children unfiled.

**Resolved in favour of folder-per-project**, which is what the braindump and
the standard already said. Eight folders minted (`pj-foodchain`,
`pj-ghtrends`, `pj-mediagen`, `pj-papers`, `pj-rtnotes`, `pj-runsmoke1`,
`pj-studyarxiv`, `pj-studynourl`), 67 channels moved with
`set_channel_folder`, and folders 1 and 2 re-described to match. Every one of
the realm's ten projects now owns its folder; the only unfiled channels left
are the realm defaults (`Zulip`, `general`, `ops`, `sandbox`),
`zz-allpublic-20260813`, and one stray.

The re-filing read each channel's **project from its own description** (`[AUTO]
project: <slug>; mission: …`, written by `ensure_work_channel`) rather than
from its name. `work-r-*` vs `work-r2-*` vs `work-r3-*` are runsmoke1,
runsmoke2 and rtnotes — a name-prefix heuristic would have merged them.

## Still open after this

- **`pj-<slug>` channel creation has no folder code path at all.** Nothing
  calls `create_channel` for a project channel, so filing stays manual and
  the drift above will simply recur. This is now the one gap worth closing:
  the decision it was blocked on is made.
- `front-comfy-command-relay` (stream 109) is a stray — that name is a
  *topic* in `#front`, and a Front run created a channel from it, holding two
  selfnotes under a topic literally named `4`. Left unfiled and unarchived
  pending the developer's call.
