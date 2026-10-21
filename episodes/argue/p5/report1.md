# argue p5 — Step 1: inventory and concrete contracts

## Observed state (2026-10-21)

| Component | State |
| --- | --- |
| WFE service | agdevworld `2eb1976`, conversation store v10, launchd `com.agdev.wfe-service`, one process on `:8098` |
| aghangout | served `96d1b99` (`<area>/hangout/current`), Gitea `developer/pj-aghangout` |
| Executor | worker `studio` idle in both lanes (`default`, `conversation`); no queued job |
| Discussions | 3 open (`p16-s3-room`, `p16-s4-provenance`, `20261020-233458-r93d5v`), none with an open invitation |
| Archive | destination `ponderer` (`conversations.json` `discussions.archive`); writer clone `<area>/.local/discussions/archive` |
| Settings repository | GitHub `iwaag/agdevworld-settings` `47d5d71` (8 commits): `manifest.toml` (`ag.settings-manifest.v1`), 4 characters (front, autolab, forge, archsage), 3 rooms |
| nctl drift | 46 converged, 2 drifting: `agobserver` `service_missing` (pre-existing since earlier episodes) and `cagent-snake-e2e` `service_missing` (on agautolab1; unrelated to this episode). `aghangout` and `agdevworld-web` converged |

Only aghangout consumes the settings repository today: its `public/`
holds converted copies of three portraits and three room backgrounds
(`public/ASSETS.md`), and `src/characters.ts` holds the lore, names,
accents, rooms and a role line by hand, plus a `ponderer` entry with no
portrait or lore. Other mentions are historical run records (forge
`provide/run-013`, `knowledge/asset-provenance.md`). The manifest's
`agents` field still maps characters to retired runtime names (`agforge`,
`autolab`, the Zulip-era `front`).

## Decisions

### Character project

- Managed project **`characters`**: Gitea `developer/pj-characters`,
  workspace `<area>/pj-characters`, agent `characters-agent`.
- The settings history is merged unchanged into the new root
  (`--allow-unrelated-histories`, as pj-ponderer took argue's). The GitHub
  repository is left as it is and is no longer read.
- Owned material:
  - `manifest.toml`, the human-edited index;
  - `characters/<id>/` with lore, portraits and provenance;
  - `rooms/<id>/`;
  - `tools/characters` (`validate`, `normalize`, `export`, `add`) with
    `--help`;
  - workflows `character` (add or update one character) and `freework`.
- Consumer format: `characters export <dir>` writes `characters.json`
  (`ag.characters.v1`) and web-sized assets. Normalization commits the
  web-sized derivatives beside the originals, so an export needs no image
  tool.
- Runtime identity leaves the character data. The `agents` lists are
  removed, and the migration is recorded in the project's `HISTORY.md`.
  `ponderer` becomes a character with neither portrait nor lore: the
  explicit fallback case.

### Consumer mapping (aghangout)

- aghangout vendors one export at `app/characters/`, pinned by the source
  commit recorded in `EXPORT.json`. `npm run characters -- <rev>` refreshes
  it, and `app/cast.json` maps WFE participants to character ids, rooms and
  role lines.
- Changing or adding a character is an export refresh plus a mapping line:
  no UI source changes.
- Mapping:
  - `front`, `forge`, `archsage` and `ponderer` → characters of the same
    id;
  - every other participant (`argue`, `aghangout`, `lowerpoly`,
    `rts-vs-bot`, `jevlab` …) is unmapped. It gets a plain plate with
    initials, and its bubbles use a neutral voice;
  - `autolab` stays a character with no current participant;
  - person messages stay original.

### Generation, storage and reads

| Concern | Choice |
| --- | --- |
| Renderer (contract owner) | aghangout `renderer/render.ts`, installed by `scripts/deploy.sh` as `<hangout dir>/renderer/<commit>` with `renderer-current` beside `current`. That directory is not served. `describe` names the renderer version, model profile, character export revision, cast and per-participant character hashes. `render` takes one source snapshot on stdin and writes one structured result |
| Model call | Claude Code CLI in print mode: no tools, no MCP, no setting sources, no session persistence, JSON schema output. Probe: `haiku` answered in 5.2 s, with cost and usage reported by the CLI |
| Job host | The WFE service process (agdevworld `server/dialogue.ts`), a worker loop with its own small concurrency. It never claims an executor slot, so the `conversation` lane and the delivery of original speech are untouched |
| Storage | `<archive workspace>/.local/character-dialogue/dialogue.sqlite`, today `pj-ponderer/.local/character-dialogue/`. The path is resolved from `discussions.archive` through the registry. One writer: the WFE dialogue worker. Run copies take only regular files directly in `.local/` (`server/runspace.ts`), so this folder is never copied into a run. The archive writer uses its own clone and never sees it |
| Jobs | Durable rows keyed by the cache identity: renderer version, character hash, language, source hash, context ids and hashes. A lease covers restarts; 3 tries, then `failed`. Each result is published in one transaction with its job |
| New speech | The worker follows new agent messages from a checkpoint taken at its first start, so history is never rendered wholesale |
| History | A reader's read queues at most a bounded window per request; the queue is idempotent by key |
| Reads (presentation path) | `GET api/discussions/<id>/dialogue?lang=ja`, `POST api/discussions/<id>/dialogue/retry`, and `wfe dialogue status\|show\|retry` |
| Ordinary reads | Original APIs, archive, `wfe discussion show/state`, overviews and run context are unchanged and never read the dialogue store |

### Trial discussion

`p16-s3-room` (aghangoutの会議室シーンをどう作るか), messages #124–#152:

- 22 messages, 14 of them agent messages, all in English;
- long speech (up to 4,736 characters; 29,612 in all);
- disagreement between forge and archsage, synthesized by argue;
- archsage's question, uncertainty about provenance, and results.

Speakers: forge and archsage (characters), argue and aghangout (unmapped),
and Omni as the person.

The discussion is still open, so a new message posted there would start a
conductor turn. Asynchronous arrival is therefore checked in a new check
discussion.

## Baseline (ignored, `pj-agdev/.local/argue-p5/step1/`)

- `p16-s3-room.api.json`, `p16-s3-room.show.json`: the original records.
- `wfe-version.json`, `aghangout-status.txt`: the current releases.
- `conversations.json.before`.
- `drift-before.json`.
