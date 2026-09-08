# Front Desk p2 ex1 — verify the Forge character with a real media task

## Goal and scope

Use Front Desk to request one image from agforge, receive the actual result in the same conversation, and replay an evidence-based exchange between Front and Forge with their portraits. Prove that the new character works through settings changes, including portrait history and reloads.

This is a private experimental environment. Keep fixes small and practical; backward compatibility is unnecessary. The implementer chooses task wording, browser tooling, and any implementation adjustments needed from observed failures. One completed image task is sufficient; video/music and a general media preview UI are outside this exercise.

## step1 — Register and load Forge

- Inspect the current settings repository and active synced revision. Planning found `characters/forge/lore.md` and `face.jpg`, but no `[characters.forge]` entry in `manifest.toml`. If still missing, add the entry with name `Forge`, nickname `ピーちゃん`, those file paths, and the agent identity declared by agforge's introduction. Local sender/instance aliases can use the existing settings override.
- Preserve the supplied lore: a media-generating robot with mysterious phrasing, conflicted about imitating an artist while appearing emotionless. Character wording comes from the full lore, not a new hardcoded guide example.
- Make the settings commit available at the configured repository/ref, then run `uv run agentroom-settings sync` from `agdevworld/agentroom`. Check `status` and the relay's `/settings` response for Forge, its lore, and a working revision-addressed portrait URL. Refresh the screen's settings.
- Confirm settings-only addition works without a frontend rebuild or listener/relay restart. If application changes prove necessary, record the specific gap and verify the corrected path.

## step2 — Request one image through a fresh Front Desk conversation

- Check service state through `pj-clusterintent/nctl` or Nautobot, then check the relevant relay/listener and media-generation availability. Read the current agforge introduction in `#agents`; let Front choose the live instance and its workflow.
- Start a new `front-desk-<id>` conversation. Use a fresh asset request topic for this exercise so callbacks return to this conversation. p2 showed that reusing a topic anchored by another Front conversation sends callbacks to that original home.
- Suggested user request: 「agforge に、星空の工房で小さなロボットが絵を描いている画像を1枚作ってもらって。正方形、1024×1024、PNG、文字なし、青と紫を基調にしたイラストで。細かな仕様は任せるので、依頼と生成開始まで進めて、完成した画像のリンクをここで報告してね。」
- Let Front handle planning questions and trigger the run that agforge opens. Answer any material clarification in the same Front Desk conversation. Exact prompt wording and reasonable size/format adjustments may follow the available generator.
- Observe delegation and callbacks through to completion. Read status while waiting; extra “how is it going?” posts can start more runs. If something fails, diagnose and retry only the necessary part, recording what changed.

## step3 — Verify the actual result and character dialogue

- Confirm an image was generated, open/download it, and check its basic format, dimensions, and requested subject. Record the delivered object key and the plan/run/result message references. A plan registration, ACK, or start notification is not completion.
- Verify Front's completion reply arrives in the original desk conversation. Its `ag-dialogue` block should contain `character: "forge"` and Front turns, use the settings revision containing Forge, and cite the real agforge messages. Check the relay's parsed `dialogue` and `dialogue_error` if the scene is absent.
- Read the source messages and compare facts: generation success/failure, asset details, and result links must match. Forge's short lines should reflect ピーちゃん's lore, while actual operational messages between Front and agforge remain ordinary professional language.
- In the browser, advance through the exchange: Front stays lower-left, Forge appears upper-left with the supplied portrait and name, and turns/page controls work. Check the result and source links. The download may open outside the scene; an inline image gallery is not required.
- Open history and verify both characters' portraits and text, with no raw JSON or duplicated reply. Reload and reopen the conversation to confirm the saved scene, sources, and revision-specific portrait still work. Check a narrow viewport as well as desktop.

## step4 — Close out with evidence

- If fixes were needed, run focused tests and build/deploy checks appropriate to those changes, then recheck the failing behavior. For settings-only work, sync/API checks plus the real conversation and browser inspection are sufficient.
- Write `report.md` with the settings revision, task outcome, source message references, delivered object key, screenshots, any fixes, and remaining issues. Keep machine-specific details, downloaded artifacts, and screenshots in ignored storage. Record any work done on an in-system agent's behalf as a handoff candidate, following the existing development policy.
- Commit and push changed settings/code/docs and relevant parent submodule pointers. Do not claim completion solely from Front's report; include the independently opened generated image and observed Forge scene.

## Useful references and findings

- `../report.md` and `../report5.md`: p2's completed implementation and the first-anchor callback issue. Use a fresh task instead of reopening its ghtrends work.
- `agforge/params/intro.md`: `assetplan-*` plans only; agforge opens `assetrun-*`, and a post there starts generation. Result delivery mentions the requester once in the plan topic. Re-read the live introduction when executing rather than hardcoding an instance channel.
- Forge delivery includes a download URL and `[S3KEY]`. URLs expire; record the durable key too. The introduction describes re-signing if a later check needs a fresh URL.
- `agdevworld/agentroom/README.md`: settings sync/configuration, sender overrides, `/settings/<revision>`, and parsed Front Desk dialogue. `agfront/agent/guides/character_talk/guide.md` already describes arbitrary character IDs from `characters.md`; Forge should not require a special routing branch.
- `agdevworld/.local/deskshot.mjs` and `pj-agdev/.local/devenv.md`: existing browser driver and deployment notes. Planning-time `nctl status` confirmed Nautobot authentication and a healthy worker; this alone does not prove agforge or its generator is ready, so check those when executing.
