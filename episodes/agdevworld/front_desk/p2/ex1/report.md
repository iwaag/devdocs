# front_desk p2 ex1 — the Forge character, proved with one real image

A third character was added by settings alone and carried a real media
task from the Front Desk to a generated file and back into a scene between
姐さん and ピーちゃん. Steps: `report1.md` (settings), `report2.md` (the
request and delivery), `report3.md` (verification of the file and the
scene), `report4.md` (close-out).

## Settings

- agdevworld-settings `2c21078` adds `[characters.forge]`: name `Forge`,
  nickname `ピーちゃん`, the lore and 1024×1024 portrait that `206cbde` had
  already placed under `characters/forge/`, `agents = ["agforge"]` (the
  `agent` name in agforge's `#agents` roster, message 4926). No override,
  no code, no rebuild, no restart: `agentroom-settings sync` made
  `2c21078ae237` active, the relay served the manifest and the
  revision-addressed portrait at once, the running screen showed the new
  revision in its header.

## Task outcome

- `#front` › `front-desk-20260908-ex1`, request 5318 (23:17). Front opened
  `#agforge-agstudio1` › `assetplan-star-workshop-robot` (5321), agforge
  registered Work F2-28 「星空の工房ロボットイラスト 生成プラン」 (5328),
  Front posted the start in `assetrun-star-workshop-robot` (5331), agforge
  delivered (5335 in the plan topic, 5337 in the run topic) about three
  minutes after the request. Every callback came home to `-ex1` — the
  fresh topic avoided p2's first-anchor routing.
- Delivered object: `[S3KEY] files/2026-09-08/65265f51327a459d802154dcd8b1e9c5.zip`
  → `star_workshop_robot.png`, PNG 1024×1024 RGB, no text, blue/purple
  starry workshop with a small robot (on the easel's canvas rather than
  painting at it — the generator's reading of the loosely specified
  action). Opened and measured independently of Front's report.
- Front's completion reply 5338 (`run-0028`, revision `2c21078…`) carries
  the URL, the key and the source id, and an `ag-dialogue` scene with
  `character: "forge"`: 「……完成した。星空、工房、小さなロボット。仕様どおりだ。」
  answered by Front. The earlier 5333 scene already had a Forge line
  paraphrasing 5328. The run's `characters.md` held the full Forge lore.

## The screen

Forge appears upper-left with the supplied portrait and
`Forge（ピーちゃん）`, Front lower-left; turns and pages advance by click,
wheel and `◀ ▶`; the history shows both portraits and every turn with no
raw JSON or duplicate; a fresh load and reopening from the history
rebuild the scene at `2c21078ae237` from Zulip; the narrow viewport plays
it too. Screenshots `agdevworld/.local/shots/p2ex1/s1-*`, `s3-*`.

## Fixes

None. Nothing in the application needed to change for a new character.

## Remaining issues

- Forge's turns carried empty `sources` in both scenes (the Autolab turns
  in p2 cited theirs); the readable reply cited 5335, so the scene shows
  the download chip but no `📎 #5335` chip. Front's judgement, worth
  watching before any guide wording is added.
- Front attributed "仕様どおり" to agforge, whose delivery only said "1
  file(s)". True of the file, but not something agforge reported.
- A French fragment ("Depêche envoyée —") opened 5323 inside a Japanese
  paragraph; the p2 preface rule only drops paragraphs with no Japanese.
- Narrow viewport: the history panel covers the scene, link chips draw
  over its text, and the input placeholder runs under the Send label.
  p2's empty dimmed Front box on a collaborator-first turn also stands.
- The subject was interpreted loosely (robot in the painting). A stricter
  prompt would be a re-trigger of F2-28, not an application change.

## References

- Conversation 5318–5338; agforge topics 5320–5337; runs Front
  `run-0026…0028`, agforge `front/run-0046`, `assetrun/run-0020`; $0.75.
- Settings `iwaag/agdevworld-settings` `2c21078`; evidence workspace
  `agfront/.local/topics/front/front-desk-20260908-ex1/3/character_talk/`.
