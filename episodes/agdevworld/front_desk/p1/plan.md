# Front Desk p1

## Goal and scope

Build a graphic-novel-style Phaser scene for chatting with agfront, requesting work, and receiving its results. Prove the flow with a few casual exchanges followed by one completed `ghtrends` task.

- Use this directory's `bg.png` as the background and `agfront.jpg` as the lower-left portrait. Start with the supplied images unchanged.
- Draw the visible GUI in Phaser: dialogue beside the portrait, a prompt bar at the bottom, and a separate history area that can be shown or hidden.
- Replies to the user use an emoji-rich Japanese gyaru style. Posts to other agents use ordinary professional language.
- This is a private experimental environment. Backward compatibility is unnecessary; replace existing pieces where useful. API details, layout, and class boundaries are the implementer's choice. A new authentication system or general-purpose chat platform is outside this phase.

## Implementation

### 1. Phaser scene and input

Add a dedicated `FrontDeskScene` to `agdevworld`, accessible through a route such as `/?view=frontdesk`, with a link from the existing dashboard. Fill the screen with the background and preserve the portrait's aspect ratio.

Implement the latest reply, pagination for long replies, scrollable history with a visibility toggle, and send status. Preserve the draft while browsing history. Make result links and other useful references accessible.

Prototype Japanese IME and emoji input early. The visible input bar belongs to Phaser; a hidden textarea may handle IME and paste. Check that composition-confirming Enter does not send, long text remains readable, and emoji are not split. Animation and responsive layout details are discretionary.

### 2. character_talk and event-driven turns

Add `profiles.character_talk` and `roles.character_talk` to `agfront/agents.toml`, plus `agent/guides/character_talk/guide.md`. Keep this independently configurable from the ordinary front role; use the existing front model and tools as an initial reference. The new guide defines the voice, not the profile name alone.

Recommend `front-desk-<conversation-id>` topics in `#front`, which fit the existing `front-` sweep. Support starting and resuming conversations. Select the role and guide from the reply destination's home topic, using the same selection for callbacks.

Each run reads the current messages, makes any necessary requests, replies to the user, and exits. Resume on the next user post or agent callback rather than holding the harness open for an answer or approval. Explain this in the new guide; do not carry over the old guide's `agentchat wait` advice. `agentchat read --since` is available for checking existing evidence.

Use the existing introductions, `agentchat`, `rootchat`, and `served` for discovery, delegation, and supervision. Let Front choose the agent and workflow from those contracts. Apply the character voice only to user-facing replies; use ordinary language for outbound agent messages.

### 3. Relay and conversation integration

Extend `agdevworld/agentroom` with Front Desk conversation listing, history reads, and posting. Keep Zulip as the history authority and reuse the existing event queue for updates. Restore history after a page reload or relay restart, including resolved topics.

Reuse the existing Developer credential for posting. Keep boundary checks practical: validate Front Desk destinations and message length, and prevent duplicate submit clicks. Show uncertain send outcomes and avoid automatic retries that could duplicate work. Keep credentials and local environment details in ignored files, following existing practice.

Hide `selfnote` content and display ACKs as receipt status rather than agent answers. A waiting indicator does not require a running harness. Make connection failures visible and retain unknown state when evidence cannot be read.

### 4. Verification and deployment

- Run `npm run build` and agfront/relay tests appropriate to the changes. Cover character_talk selection for direct posts and callbacks, ordinary front routing, history restoration, destination checks, and duplicate sends.
- Check images, long replies, history, Japanese IME, emoji, and resizing in the browser. Rebuild the web deployment, restart changed resident processes, and verify the deployed behavior.
- Exchange a few casual messages through Front Desk, then request: “Run ghtrends once and see it through to completion.” Answer any necessary conversational clarification through the same screen.
- Record that Front exits after delegation, resumes when the other agent mentions it, and reports in the original conversation. Inspect actual posts for the two intended writing styles.
- Verify the ghtrends output and index update are committed, the work is complete, and Front Desk receives the result. An ACK or delegation report alone is insufficient.
- Write a concise `report.md` with results, conversation/work references, and remaining issues. Commit and push changes. Keep machine-specific evidence in ignored storage.

## Findings and implementation hints

- `agdevworld/src/main.ts` selects the entrance and loads Phaser only for world views. A dedicated scene is easier to manage separately from `PanelGridScene`.
- In `agfront/src/agfront/zulip_listener.py`, `serve`, `front_prompt`, and `run_front` currently select front unconditionally. `handle_mention` resolves the original conversation through `rootchat_home` and calls the same `serve`, providing a shared point for role/guide selection. `run_role` also accepts a profile argument.
- The listener already rechecks posts arriving during a run. `note_served` records handled callbacks. Reuse these mechanisms before introducing another waiting loop or conversation ledger.
- The relay's `chat.py` currently permits routine conversations only. `server.py` and `ops.py` connect HTTP reads/writes and the event queue. Give Front Desk an appropriate entrance instead of merely broadening the routine destination check.
- The actual routine name is `ghtrends`. `routine-ghtrends` contains Front reports as well as instructions: select the request author's newest post, as existing `routines.py` does, rather than the newest post overall.
- The observed ghtrends request asks for one previously uncovered trending repository, GitHub API figures, and a summary under `main/repos/` plus an update to `main/index.md` in the same commit. Read the current standing request again when executing. The latest run observed during planning was verification-only and is not evidence of successful task execution.
- During planning, `nctl status` was healthy, the target host's drift was converged, the relay was live, and its chat credential was configured. Check service state through `pj-clusterintent/nctl` or Nautobot during implementation. No pj-clusterintent code change is currently expected.
- The existing ignored `.local/opsshot.mjs` is useful for browser checks. Consult `agdevworld/README_DEV.md` and `pj-agdev/.local/devenv.md` for deployment and verification. Environment notes retain historical descriptions; prefer current code and observations when they disagree.
