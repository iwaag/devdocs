# argue p5 — Step 2: the character project

## Result

| Repository | Revision | Change |
| --- | --- | --- |
| developer/pj-characters (new, Gitea id 152, project `characters`, workspace `<area>/pj-characters`) | `1b01dfa` | the settings history (8 commits, `7b24fec`…`47d5d71`) unchanged, then manifest v2 (`6465841`) and the managed project (`1b01dfa`) |
| developer/pj-aghangout | `5179b41` | pinned export in `app/public/cast/`, `app/cast.json`, `src/characters.ts` reads them; `public/characters`, `public/rooms`, `public/ASSETS.md` removed |

GitHub `iwaag/agdevworld-settings` is unchanged at `47d5d71` and is no longer
read.

## The project

- **Data.**
  - `manifest.toml` (`ag.characters-manifest.v2`) lists five characters
    and three rooms:
    - `front`, `forge`, `archsage`, `autolab`;
    - `ponderer`, with a name and an accent only: the case of no
      portrait, lore or style.
  - Each character has its lore, a new `style.md` (how they speak, 1–2
    lines derived from the lore by Omni, marked as such), an accent,
    a portrait and provenance as known.
  - Originals are unchanged. Web derivatives (`web/*.jpg`) are recorded
    in `normalized.json` by source sha256.
- **Runtime identity removed.** The v1 `agents` lists (`front`, `agforge`,
  `autolab`, `archsage`) are gone. `HISTORY.md` records why and where they
  went: each consumer maps its own participants, and validate refuses
  `agents` with that explanation.
- **`tools/characters`** (Python stdlib; Pillow or `sips` for images):
  `list`, `show`, `validate`, `normalize`, `add [--update]`, `export`, each
  with `--help`.
  - `tests/test_characters.py` has 7 tests covering missing portrait,
    stale derivative, the runtime-identity field, bad TOML and color, a
    path outside the character's folder, and add + export; all pass.
  - Every diagnostic names the file and the fix, for example:
    `manifest.toml [characters.forge]: portrait names
    characters/forge/face.jpg, which does not exist; add the image or
    remove the field`.
  - `export` writes nothing when validation fails.
- **Consumer format**: `docs/format.md` (`characters.json`,
  `ag.characters.v1`, plus web assets).
  - Each character carries `hash`, and `source.commit`/`clean` name what
    was exported.
- **Project definition**:
  - `project.yaml` with areas `characters`, `rooms`, `tools`, `docs`;
  - workflows `character` (shape → edit, normalize, validate → confirm)
    and `freework`, both valid;
  - agent `characters-agent` with guide `agent/guide.md`.

## aghangout consumes the export

- `npm run characters -- [<rev>]` exports the characters project at a
  revision into `app/public/cast/`. The source is this checkout's origin
  with `pj-aghangout` replaced by `pj-characters`, or `CHARACTERS_REPO`.
  Refreshing from Gitea `main` reproduced the committed export
  byte-for-byte.
- `app/cast.json` (`aghangout.cast.v1`) holds the mapping.
  - Lobby and meeting rooms: `argue`.
  - `front` → `front` (room `front`); `forge`, `archsage` → their
    characters (room `project`); `ponderer` → `ponderer`.
  - Each entry carries the lobby's role line.
  - Every other participant, and a mapping to a character the export lacks,
    is plain: WFE's project name, initials and the default accent.
- `src/characters.ts` keeps its interface (`characterOf`, `rank`,
  `initials`, `LOBBY_ROOM`, `MEETING_ROOM`). It adds `character` (the
  character id or null) and `style`. No other source file changed.
- Checks:
  - `npm run check`: typecheck, build and 25 unit tests. The characters
    test now covers a mapped character, ponderer without a portrait and
    unmapped `argue`.
  - `npm run browser-check` (Chromium, with WFE's embedded graph): all
    pass.

## Addition and update reach a preview without UI changes

In a scratch clone of pj-characters:

- `tools/characters add sage --name Sage --nickname しおりさん --color
  #6fb0ff --style-file … --portrait <a drawn PNG> --provenance …`;
- `tools/characters add forge --update --nickname ピーちゃん（更新）`;
- commit `a716fbf`.

In a scratch copy of aghangout's `app/`:

- `CHARACTERS_REPO=<clone> npm run characters`;
- one `cast.json` line (`"sage": {"character": "sage"}`);
- `vite build` and the fixture.

`src/` was byte-identical (`diff -rq`). The lobby card read
`ピーちゃん（更新）` with the exported face, and the new-discussion picker
listed `Sage（利用不可）` (the fixture's unavailable `sage` participant).
Screenshots: `pj-agdev/.local/argue-p5/step2/preview-*.png`. Nothing from
the demonstration was pushed.

## Not deployed yet

aghangout `5179b41` is pushed but not served. It is deployed with the
dialogue work in Step 6.
