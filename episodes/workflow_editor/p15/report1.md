# p15 Step 1 — inventory and implementation boundary

Step 1 settles the boundary before any change. The current state was read
from the code at agdevworld `dbd974b` (the deployed WFE release), the
operating area, the Front, forge and archsage workspaces, the Slack bridge
and `nctl drift --host agstudio` (converged; `agdevworld-web` is the only
agdevworld service registered).

## What exists today

**The embedded Front Desk** (agdevworld `src/`, about 6,000 lines). It is one
Phaser scene serving two rooms, the Front Desk (`/?view=frontdesk`) and the
Arguing Room (`/?view=argue`). Its parts:

| Part | Files | Source of data |
| --- | --- | --- |
| Scene: background, portrait, dialogue box, link chips, reply pager, history column, buttons | `scenes/FrontDeskScene.ts` (1,650 lines) | — |
| Playback: agent posts as turns, a cursor, pagination | `frontDeskPlayback.ts`, `textLayout.ts` | — |
| Input: hidden IME textarea, Enter sends, Shift+Enter newline | `frontDeskInput.ts` | — |
| Room adapters: list, open, post, close | `roomState.ts` (`deskAdapter`: WFE conversations; `argueAdapter`: relay) | WFE `api/conversations`; relay |
| Demo fixture | `roomDemo.ts` | none |
| Characters, lore, portraits, backgrounds | `settingsState.ts`, `frontDeskSettings.ts` | relay `/settings`, which syncs the `agdevworld-settings` repository |
| Contexts panel | `contextPanel.ts`, `contextState.ts` | relay `/contexts` (legacy shared-context repositories) |
| Progress panel | `progressPanel.ts`, `progressState.ts` | relay `/progress` (Zulip mirror and Observer board) |
| Post meaning, re-voicing, interpretations | `postMeaning.ts`, `Presentation` types | relay; retired since p8 (`presentation: null`) |
| Close a conversation / argue | `frontDeskClosePanel.ts`, `completionState.ts` | WFE `close`; relay `/complete` |

The desk assumes one entrance throughout: the title, the prompt ("Say
something to Front…"), the portrait choice and the adapter's single
conversation list.

**WFE conversations** (`server/conversations.ts`, `cli/conversation.ts`,
`docs/conversations.md`, HTTP under `api/conversations`):

- **General already.**
  - Conversation identity is separate from runs.
  - Submit tokens make a resubmission one message.
  - Turns are durable: message first, then a request with receipt
    `conversation:<conv>:<msg>`. `dispatchDue` repairs a failed or
    interrupted dispatch.
  - Replies carry their run, attempt, `re` and kind. A retry key makes them
    idempotent.
  - Links record the agent's reading. Pending questions are derived.
  - The view shows each linked run's shared status and its view URL.
  - `via` and `author` support non-browser clients and trial actors.
- **Assumes one entrance.**
  - `conversations.json` has one global `entrance {workspace, workflow}`.
  - `dispatch` reads that global entrance, not a recorded destination, so
    changing the file would re-route old conversations.
  - A conversation records no destination at all.
  - The list and view never say who the conversation is with.
  - The CLI help and the docs say "the entrance (Front)".
- **Missing for an independent client.**
  - No target discovery.
  - No way to create a conversation with a named destination.
  - No refusal when a post would go to another destination.
  - No artifact information beyond what replies write in their text.

**Workflows.**

- Front `converse`: one `talk` node, `node: studio`, `lane: conversation`.
  Its guide is written around `wfe conversation show|reply|link`.
- forge has `provide`, `develop-capability`, `refresh-catalog` and
  `freework`.
- archsage has `consult`, `study`, `backfill-wedo` and `freework`.
- All of forge's and archsage's workflows run in the default lane, and their
  guides say "the person reaches you through the run's request topic or the
  run view". Neither has a conversational workflow.

**Execution lanes.** Worker `studio` has `lanes: {conversation: 1}`, and the
executor claims in `["conversation"]` beside the default. One conversation
slot is shared by every conversational workflow that declares the lane.

**Active clients and links.**

- The Slack bridge (`com.agdev.slack-front`, release `61778d4`; 10 bound
  threads) posts to `api/conversations/<id>/messages` with `via`. A new DM
  is a new conversation, created implicitly by its first post, and it
  silently relies on the global entrance. Its replies link the Front Desk
  (`deskUrl` = `<linkBase origin>/?view=frontdesk&conv=<id>`).
- WFE Home links "Front Desk ↗" (`/?view=frontdesk`).
- The Front guide and README_DEV name the desk URL.

**Store state now.**

- 23 conversations, 38 person messages (all `started`) and 48 replies.
- Every turn went to `front/converse`.
- No conversation run is active. The only unfinished run anywhere is
  jevlab's person-held experiment.
- No message is pending dispatch.

## Feature and owner map

| Feature | Owner after p15 | Notes |
| --- | --- | --- |
| Conversation identity, destination, messages, replies, links, pending questions, dispatch, recovery | WFE | Destination recorded per conversation (Step 2) |
| Target discovery (which project/workflow pairs accept conversations, and whether each is usable now) | WFE `GET api/conversations/targets` | Identity only: project, workflow, names, availability |
| Linked work status, run links | WFE conversation view (existing `runs[].status`, `url`) | Replaces the relay's progress panel for this app |
| Result/artifact access | WFE: run view URLs, `raw` file route, open presentations of linked runs | Added to the conversation view (Step 2) |
| Character names, nicknames, lore, portraits, room backgrounds, project→character mapping | aghangout (its own assets, copied from `agdevworld-settings` with provenance) | No relay settings route |
| Scene, playback, pagination, input, history, conversation navigation, character choice | aghangout | Reimplemented in DOM/CSS with the playback/pagination logic ported (below) |
| Display preferences (text speed, last character, panel state) | aghangout, browser `localStorage` | Not authoritative state |
| Context | WFE Home `#/home/context` (link) | The relay's legacy context panel is not carried |
| Detailed run administration | WFE run views (links) | |
| Settings repository sync and the relay's `settings`, `contexts`, `progress`, argue routes | Retired with the desk (Step 5) | Their only consumer is the desk/Arguing Room |
| Relay `/budget`, `/cost` | Kept | WFE Usage reads them |
| Re-voicing, interpretations, post meaning, Arguing Room | Retired | Arguing Room viewer not required (plan) |

**Scene technology.** aghangout keeps the visual-novel presentation
(background room, character portrait, dialogue box, page cursor, history) in
DOM and CSS instead of a Phaser canvas. The desk's canvas needed a hidden
textarea for IME, its own grapheme wrap and hand-drawn link chips. In DOM,
long replies, links, selection, keyboard focus and narrow layouts work by
default. The turn/cursor model of `frontDeskPlayback.ts` and the grapheme
pagination idea of `textLayout.ts` are ported. Room adapters, relay types and
re-voicing are not.

## API gaps closed in Step 2

1. **Durable destination.** Store schema v3 adds `workspace` and `workflow`
   to conversations. Dispatch and `dispatchDue` use the recorded
   destination.
2. **Targets.** `conversations.json` becomes `{person, targets: [{workspace,
   workflow}]}`. The service lists them with project and workflow names and
   availability (`ok` / `no-workspace` / `no-workflow`). A target not
   listed cannot start a new conversation. An existing conversation keeps
   dispatching to its recorded destination even after the list changes, and
   a destination that vanished fails visibly (`dispatch_state: failed`, with
   the reason) without falling back.
3. **Creation and refusal.** A post that creates a conversation must name
   `target`. A post that names a target different from the conversation's
   destination is refused (409). Nothing defaults to Front.
4. **Client view.** The list and view carry the destination. The view adds
   per linked run its open presentations (resolved URLs) and the run's
   `raw` file base. Replies keep their own links.
5. **CLI/docs/help** follow (`wfe conversation targets`, `post --to`, and the
   help text no longer says "the entrance (Front)").

## History decision

Keep the existing store and migrate it in place. The v3 migration sets every
existing conversation's destination to `front/converse` once; it is the only
destination ever used. This costs one migration step, keeps the Slack
threads' conversations working, and needs no second schema. No Zulip-era
conversation is imported or read.

Active delivery obligations at cut-over:

- no active conversation runs;
- no pending dispatches;
- 10 Slack thread bindings, whose conversations keep their ids and get
  `front/converse` as their destination.

## Deployment approach

- **Project.** `wfe create pj-aghangout` in the operating area creates the
  Gitea repository `developer/pj-aghangout`, workspace `aghangout`, with
  devdocs, guide and workflows, then registers it. This is the path p8 used
  for Front.
- **Application.** Vite + TypeScript static bundle, `base: './'`, with a
  configurable API base. The default is the same origin's `/wfe/`.
- **Hosting.**
  - Presentations are per-run, frozen and on their own origin, so they
    cannot write conversations.
  - Boast pages are sandboxed to an opaque origin.
  - Neither suits a persistent writer. The selected route is a same-origin
    path on the agdev origin: nginx (`:8093`) serves `/hangout/` from a
    host directory mounted into the web container read-only,
    `<area>/hangout/current/`.
  - aghangout's own deploy script builds the app into
    `<area>/hangout/releases/<commit>/` and switches `current`. Changing
    and redeploying aghangout never rebuilds agdevworld. Only the one-time
    nginx/compose mount is agdevworld's.
  - Writes are same-origin, so the existing write-origin check accepts
    them. Independent hosting uses WFE's `allowedOrigins` (documented, not
    enabled).
- **Identity.** Browser posts are the configured person
  (`conversations.json` `person`), as every editor write is. No credential
  is in the bundle.
- **Links.** WFE Home gets an aghangout entrance. The Slack bridge's desk
  link becomes the aghangout conversation URL. The legacy
  `/?view=frontdesk&conv=<id>` is redirected to the new URL only if it costs
  nothing (an nginx rule). Otherwise it is dropped.
- **Access proof.** No privilege beyond the browser's same-origin writes is
  needed: aghangout calls only `api/conversations…` and reads run views and
  `raw` files. A project agent's runs need nothing new. Step 2 verifies this
  through `:8093` from another host.
- **Nautobot.** `agdevworld-web` covers the nginx route. aghangout is
  recorded as a service placed on agstudio (manual toolchain) in Step 5.

## Responsiveness note for Step 3

Three conversation targets share one `conversation` slot. Step 3 measures
first, and raises the lane to 2 only if a turn waits behind another
target's turn.
