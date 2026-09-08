# Step 1 — settings repository and runtime updates

agdevworld `b0427fb`; agdevworld-settings `4f3b55f` (pushed). Nothing in
agfront or pyagag changed yet.

## What exists now

- **The settings repository has an index.** `manifest.toml`
  (`ag.settings-manifest.v1`) in `iwaag/agdevworld-settings`: for each
  character a display `name`, a `nickname` (姐さん / 親方, taken from the
  lore), the `lore` and `face` paths, and `agents` — the `agent` name an
  instance declares in its `#agents` roster (`front`, `autolab`). Rooms
  carry a `name` and a `background`. Adding a character is a new table plus
  its files, in that repository only. A short `README.md` says so.
- **Configuration is a file, not code.** `agdevworld/settings.example.toml`
  is tracked; `agdevworld/.local/settings.toml` is the ignored copy with the
  URL, the ref (branch, tag or commit id) and the destination
  (`.local/settings` by default, relative to agdevworld's root). Instance
  mappings go in `[overrides.characters.<id>]` (`agents`, `senders`), so a
  name that belongs to this realm never enters the shared repository. The
  relay finds the file by default or through `AGENTROOM_SETTINGS_CONFIG`.
- **An explicit sync.** `uv run agentroom-settings sync` (from
  `agdevworld/agentroom/`) clones or fetches, resolves the ref, snapshots the
  commit with `git archive` under `revisions/<sha>/`, reads the manifest,
  checks every lore (non-empty) and every image it names, and only then
  writes `active.json` and repoints the `current` symlink — both atomically.
  A failure prints the reason, records it in `last_sync.json` and keeps the
  previous revision active. `status` and `show` read it back. Nothing fetches
  on a request or a conversation.
- **Three relay routes**, in `agentroom/src/agentroom/settings.py` and
  `server.py`: `GET /settings` (config, active revision, last sync, and the
  manifest with lore inline and every asset addressed as
  `/settings/<revision>/<path>`), `GET /settings/<revision>` (a retained
  revision's manifest, `404` + `retained: false` otherwise) and
  `GET /settings/<revision>/<path>` (one manifest-named file, its content
  type, `Cache-Control: immutable`). Only files the manifest names are
  served — `LICENSE`, `../active.json`, an unnamed image all answer 404.
  The relay reads the active revision on every request, so a content update
  reaches the browser with no restart and no rebuild; the revision in the
  URL is the cache buster.
- **Agents read the same revision as files**: `.local/settings/current/`
  (active) and `.local/settings/revisions/<sha>/` (any retained). Step 2
  points `character_talk` at them.
- **The frontend read** exists (`src/settingsState.ts`: `readSettings`,
  `readRevision`, `assetUrl`, the types) and compiles; the scene starts
  using it in step 4, where the fixed image paths are replaced.

## Verification

Tests: `agentroom` `uv run pytest -q` → **164 passed** (15 new in
`tests/test_settings.py`, against a fixture Git repository in `tmp_path`,
no network): first sync; lore + image change becomes a new revision while
the old one stays readable by id with its own portrait; unchanged ref →
`changed: false`; adding a character; a manifest naming a missing face
fails the sync and keeps the previous revision (visible in `/settings`
as `last_sync.ok: false` beside the still-serving `active`); an unknown
ref; an unreachable URL on the first sync leaves nothing active and says
so; replacing the repository URL is followed in the same clone; a tag and
a commit id as refs; manifest checks (schema, no characters, path escape,
empty lore); overrides extend `agents`/`senders` without touching the
repository; the relay's asset URLs, content types and immutable header; the
relay serving only manifest-named files; a retained revision answering by
id after the active one moved on, with no server restart.

Live, against the real repository (`.local/settings.toml`):

```
$ uv run agentroom-settings sync
cloning https://github.com/iwaag/agdevworld-settings.git
fetching main from https://github.com/iwaag/agdevworld-settings.git
retained revision 4f3b55f654c8
active revision 4f3b55f654c8 (new); characters: autolab, front
```

A foreground relay on `:8095` answered `GET /settings` with the manifest
(`front`, `autolab`, room `front`), the portrait as `image/jpeg` (649,262
bytes) and the background as `image/png` with the immutable header;
`/settings/<rev>/LICENSE` and `/settings/<rev>/../active.json` → 404;
`/settings/0000000` → `retained: false`.

Live, against an ignored fixture repository
(`.local/settings-fixture/`, `AGENTROOM_SETTINGS_CONFIG` pointing at its
own config), the five scenarios the plan named:

| scenario | result |
|---|---|
| initial sync | `65b114b54322` active, characters: front |
| lore change + manifest naming a renamed-away face | `sync failed: characters.front: face characters/front/nope.jpg is missing at 07cba7d02ebf; keeping 65b114b54322… active`, exit 1; `status` shows `last sync: FAILED — …` under the still-active revision |
| fix + character added (autolab) | `14765683f062` active, characters: autolab, front; the new lore and face appear with no code change |
| repository replacement (URL switched to the GitHub one in the config) | `origin … → https://github.com/iwaag/agdevworld-settings.git`, `4f3b55f654c8` active; all 4 revisions retained, `current` → the new one |

`npm run build` passes with `settingsState.ts` in the bundle.

## Notes

- The snapshot of a revision whose manifest fails the check is retained
  too (`07cba7d` above). Harmless — it is never active and nothing serves
  it — and it saves a second `git archive` when the fix lands; left as is.
- `mkdtemp` makes a revision directory `0700`; the relay and the agents run
  as the same user, so nothing needed changing.
- The running launchd relay still serves the pre-p2 code; it is kickstarted
  when the frontend starts reading `/settings` (step 4/5). No plist change
  is needed: the config path defaults to `agdevworld/.local/settings.toml`.
- `pj-agdev/.local/devenv.md` gained a "Settings repository" section with
  the local paths and the first sync's revision.
