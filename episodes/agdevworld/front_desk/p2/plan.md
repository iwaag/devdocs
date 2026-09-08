# Front Desk p2 — settings and character conversations

## Goal and approach

Load character lore, portraits, and backgrounds from a configurable settings repository. Have agfront's `character_talk` turn the relevant real conversations into a short exchange between characters, replay it in Front Desk, and show portraits in history.

This is a private experimental environment and a breaking-change phase. Replace obsolete formats and implementation freely; backward compatibility and migration of old conversations are unnecessary. The steps define outcomes, not fixed class boundaries or tool sequences. Choose the smallest practical implementation; authentication redesign and a general-purpose asset platform are outside this phase.

## step1 — Settings repository and runtime updates

- Clone `https://github.com/iwaag/agdevworld-settings.git` beneath `agdevworld/.local/`, initially `.local/settings/`. Make repository URL, branch/ref, and destination configurable in a file, with a tracked example and ignored local configuration.
- Start from the existing `characters/<id>/lore.md`, `face.jpg`, and `rooms/front/bg.png`. Add a small manifest for display names, agent-to-character mapping, and room assets. Keep instance-specific mappings in a local override when needed. Adding a character should require settings changes only.
- Provide an explicit sync command. Fetch, check that the manifest and referenced files are usable, then switch the active settings revision together. If sync fails, report the reason and keep the previous usable revision. No Git fetch on each conversation; periodic execution can use the same command later.
- Expose the manifest and referenced images through the existing agentroom relay. Load them at runtime when the screen opens or the user selects a settings refresh. Include the revision in asset URLs so browser caching does not retain replaced images. Frontend rebuilds and service restarts should not be required for content updates.
- Make the same revision available to agents as readable files. Keep revisions used by saved dialogue so its original portraits/background remain available. A full Git commit identity plus a retained checkout/snapshot is sufficient; storage layout is discretionary.

Verify: initial sync, lore/image changes, character addition, repository replacement, and a failed sync. New settings become available without rebuilding the frontend.

## step2 — Character lore and relevant conversation evidence

- Supply `character_talk` with one settings revision per run, its character manifest/lore, the home conversation, and relevant remote conversations. Record the revision with the generated dialogue.
- Move the character definition out of the hardcoded guide into settings. Current lore calls Front “姐さん” and Autolab “親方”; read their complete lore rather than retaining the old gyaru description as a competing authority.
- Reuse `remotes_for_home` and `write_threads` in `agfront/src/agfront/zulip_listener.py`. Directly linked remote histories already reach each run. Let Front read additional relevant topics using `agentchat`; Autolab's nested work conversations are not necessarily in the initial thread set.
- Supply source message IDs, sender identities, and channel/topic references alongside the readable history. Existing `format_chatlog` drops IDs; retain metadata from messages already fetched where practical. Make unavailable or bounded history apparent to the agent.
- Keep character files accessible through a documented path/configuration that other agent workspaces can also use. Initially Front authors the on-screen exchange; other agents do not need new generation roles or character-speaking listeners.

Verify: direct user turns and callbacks receive the intended lore and evidence, including completed topics renamed with `✔`. A settings update affects the next run, without changing a run already in progress.

## step3 — Generate and persist a short dialogue

- In the existing `character_talk` run, generate the normal Front reply plus a versioned JSON dialogue. Use ordered turns with a character ID, text, and optional source references. Include the settings revision. A few short turns are the initial target; small talk can remain Front-only.
- Base other characters' lines on the relevant actual exchange. Character phrasing may be adapted, while results, progress, names, numbers, and links remain faithful to the evidence. Keep operational `agentchat` messages in ordinary professional language.
- Validate and serialize the result in agfront, then save the reply and dialogue together in one Front-authored Zulip post. A readable reply plus a fenced structured block is the suggested format; choose the exact schema and encoding during implementation. Character names in the script should not become live bot mentions.
- Extend `agentroom/src/agentroom/frontdesk.py` and `src/frontDeskState.ts` to return structured turns. Zulip remains the conversation record, including after page reloads and relay restarts. Keep machine-format blocks out of the rendered conversation and avoid duplicating the normal Front reply in playback/history.
- If the structured part is unusable, display the accompanying normal reply and record the formatting issue. Avoid rerunning the whole delegation just to repair presentation. This is a current-output failure path, not a compatibility requirement for p1 data.

Verify: multi-character and Front-only output, invalid structured output, source references, and reload/restart restoration. Preserve the existing event-driven completion/callback flow; presentation should not require another paid agent run.

## step4 — Scene playback and portrait history

- Replace fixed image paths in `src/scenes/FrontDeskScene.ts` with runtime settings. Keep Front's lower-left portrait and dialogue area; show other speakers with a portrait and text box in the upper left, switching characters as turns advance.
- Add ordered playback over turns. Page through a long turn before advancing to the next one. Queue new replies while the user is reading, with a visible way to advance to them. Playback controls and animation details are discretionary.
- Render history as portrait, display name, and text per turn; use a user icon for developer messages and a common icon for unconfigured speakers. Keep ACKs as receipt indicators and retain useful result/source links.
- Use each saved dialogue's settings revision during playback. A refresh selects the current settings for new content; it does not silently replace the faces or background of a dialogue being read. Report missing retained assets and use a practical fallback.
- Keep Japanese IME, emoji-safe wrapping, draft preservation, and readable narrow-screen layout working. Resetting old p1/demo conversation data is acceptable.

Verify in the browser: Front/Autolab exchange, a third character added through settings, long turns, portrait history, queued arrivals, conversation switching, settings refresh, and desktop/narrow layouts.

## step5 — Integrated verification, deployment, and report

- Run `npm run build` and focused agfront/relay tests for the changed contracts: settings revision selection, role selection on callbacks, dialogue parsing, and history restoration. Use shared input/output examples to keep producer and consumer aligned; avoid broad tests that only mirror implementation.
- Check environment state through `pj-clusterintent/nctl` or Nautobot before working with resident services. Deploy the changed frontend and restart changed listeners/relay using the local environment instructions.
- Through Front Desk, check casual conversation and one bounded delegated task through its actual result. Read the real remote exchange and verify that the resulting short dialogue reflects it, with Front at the bottom and the collaborator at the upper left. An ACK or a “started” report is not task completion.
- Change lore and an image, sync, and verify the next generated dialogue uses the new revision without a frontend rebuild or daemon restart. Reopen a previous dialogue and verify its recorded revision is still usable. Keep temporary verification settings in an ignored fixture repository if convenient.
- Write `report.md` with implemented behavior, commands/results, dialogue and source references, and remaining issues. Update runtime/setup documentation and relevant local notes. Commit and push changes in the affected repositories, including parent submodule pointers as needed.

## Useful findings and implementation hints

- The root-level `agdevworld-settings` checkout already contains Front and Autolab lore/portraits and the Front background. Inspect it before designing the manifest; no asset generation is required to start.
- `agfront/src/agfront/zulip_listener.py` owns `serve`, `front_prompt`, `run_front`, and home-topic-based `role_for`. Both direct requests and callbacks use this path. `roles.character_talk` already has its own profile in `agents.toml`.
- `pyagag/src/agag/topics.py` provides thread rendering and `TopicResult`; `pyagag/src/agag/zulip.py` provides remote/home links and reads across resolve renames. Prefer an agfront-local extension unless a shared change makes the implementation simpler.
- The relay already serves `/frontdesk`, conversation detail, and posting, using the ops event queue and a direct-read fallback for older topics. Build on this instead of introducing a second history store or full-realm polling loop.
- `FrontDeskScene.ts` currently displays `latest_reply`, preloads fixed images, and renders text-only history. Its refresh signature largely uses message IDs; include content/settings changes when revising it so edits are visible too.
- Existing `textLayout.ts` handles Japanese/emoji wrapping. p1 found that a geometry mask on container children did not clip as expected; include portraits when checking history clipping. The ignored `.local/deskshot.mjs` is a useful browser driver.
- Consult `pj-agdev/.local/devenv.md` for current Front Desk deployment details; earlier sections retain retired architecture. Planning-time nctl observations showed Front and local Autolab polling, while the other Autolab node was stale. Refresh observations before selecting a live verification target.
- Most changes belong to agfront, agdevworld, and the settings repository. pj-clusterintent changes are only expected if deployment/distribution needs them. Keep local machine details and credentials in ignored files, per existing project practice.
