# p15 Step 4 — the managed aghangout application

`aghangout` is an ordinary managed project: Gitea `developer/pj-aghangout`,
workspace `aghangout`, agent `aghangout-agent`. It was created with
`wfe create` (the path Front was created by in p8) and registered in the
catalog. Its application is in `app/`. It was checked in a browser against
an isolated stand-in for WFE before any live use.

## The project (pj-aghangout `b37c314`)

| Item | Content |
| --- | --- |
| `project.yaml` | Intent and goals: meet the project agents, presentation owned here, conversations and work owned by WFE. Areas `app` and `scripts`. Project guide `agent/guide.md` |
| `README.md` | Using it, development commands, files, the connection to WFE, deployment |
| `agent/guide.md` | aghangout-agent: what it can see (README, the contract, fixture, checks, deploy script, Playwright), and the facts it works under (WFE is the authority; partner = destination; characters are presentation; when a change is ready; where it is served) |
| Workflows | `freework` and `change` (change and check → deploy → show the person) |
| `scripts/deploy.sh` | Builds a committed revision in a clean export (`npm ci && npm run check`), installs `releases/<commit>/` and switches `current`. Also `status` and `activate <commit>` (rollback). The directory comes from `AGHANGOUT_DEPLOY_DIR` or the ignored `.local/deploy.env` |

## The application

Vite and TypeScript, no framework, no canvas: 26 kB of JS and 10 kB of CSS.

| Part | Files | Notes |
| --- | --- | --- |
| Contract client | `src/api.ts` | Types for targets, listing, view, messages, runs and posts. `ApiError` says whether a failed write may have arrived (`uncertain`). `apiBase()` uses the same origin's `/wfe/`, or `apiBase` in an optional `config.json`. Conversation ids and submit tokens are minted here |
| Data handling | `src/model.ts` (tested) | Link segments (ASCII URLs; Markdown labels; sentence punctuation and following Japanese stay text). `merge` for `?since=` reads. `standing` of each message of the person: answered (a reply names it or comes from its run, never by recency), working, not started (with WFE's reason), quiet. Pending questions. Linked work, unfinished first. Playback cursor (first unseen line on opening; follows new lines; holds them behind ▶ when reading back). Grapheme-safe reveal |
| Characters | `src/characters.ts`, `public/` | Front, forge and archsage with their names, nicknames, lore, faces and rooms (copied from `agdevworld-settings@47d5d71`, provenance in `public/ASSETS.md`). The lobby is the izakaya scene, Front's room her desk, forge's and archsage's the workshop. Any other target gets a plain presentation from what WFE says |
| Browser state | `src/local.ts` | Text speed, open panel, last-seen line, last conversation per partner, drafts, and an outbox of messages not yet confirmed (kept with their tokens). Storage failures are tolerated |
| Presentation | `src/lobby.ts`, `src/room.ts`, `src/styles.css` | See below |

**What the person sees.**

- **Lobby:** a card per target, with face, name, role, project, workflow,
  agent and the last line. An unavailable target is disabled, with WFE's
  reason.
- **Room:** the partner's room behind them and their portrait.
  - The dialogue box plays one line at a time.
  - The name plate shows the character, the actual author
    (`forge-agent`), the kind, `re #n` and the time.
  - Links in the text open. Chips carry the line's run (with its short
    status), serving review pages, and "answer this" for a pending question.
  - The status line shows: "Forge is on #n · Working… run ↗"; not-started
    messages with their reason ("WFE retries it at the same destination");
    pending questions; a closed conversation; WFE not answering, with the
    last read kept.
- **Composer:**
  - Enter sends; Shift+Enter makes a new line; IME composition never sends.
  - "Answering #n" names a question.
  - The outbox shows sending and failed messages with the reason, Retry
    (same token) and Edit.
- **Panels:**
  - History: every message, the person's included, with `via` and `re`.
  - Work: linked runs, status, run view, review pages, and links to
    Context and the project in WFE.
  - Talks: this partner's conversations, a new one, and close (no work
    stopped).
- **Navigation:**
  - `?with=<project>` opens the last or newest conversation with that
    partner, or a new one.
  - `?conv=<id>` takes its partner from the recorded destination. An
    address naming another partner cannot retarget it; the page corrects
    the address.

**Relay-backed desk panels and their replacements.**

- Progress → the status line and the Work panel, from the conversation
  view.
- Contexts → a link to WFE Context.
- Settings sync → aghangout's own assets.
- Re-voicing, interpretations, post meaning and the Arguing Room were not
  carried over.

## Checks

- `npm run check`: typecheck, build and 8 unit tests in `test/model.test.ts`.
  They cover links, merging, standing (including that a reply to #2 does
  not answer #1), questions and work, playback, reveal, character fallback
  and id format.
- `npm run browser-check` against `fixture/server.ts`, which serves the
  bundle under `/hangout/` and scripts the conversation API on one origin.
  - The fixture covers replies, progress plus a result with a linked
    provide run and review page, questions, slow turns and an unavailable
    archsage, plus control routes for failed posts and an unreachable WFE.
  - **38 checks, all passing in Chromium and in WebKit.** They cover:
    - the lobby: three partners; unavailable and why;
    - the address and narration; destination shown; reply played; author
      and `re` on the plate; run chip; text links;
    - a long reply scrolling inside the box, with no sideways overflow;
    - Shift+Enter; ← and →;
    - a question: status, "answer this", "Answering #n", cleared after the
      answer;
    - a failed post: the reason shown; the draft and the failed message
      survive a reload; Retry is one message in WFE;
    - switching to forge: another conversation; forge's lines are forge's;
      the Work panel lists the provide run and its review page; History;
    - a conversation link opens with its recorded partner, and nothing of
      forge's is in the Front conversation; an address naming another
      partner cannot retarget it;
    - an unavailable partner's room: the reason shown and Send disabled;
    - WFE down: the last read stays and the page recovers by itself; no
      page errors;
    - phone (390×844): lobby and room fit, the dialogue box sits above the
      composer, panels go full-screen.
- Two problems were found and fixed in these checks:
  - Room backgrounds did not render: a relative `url()` in a CSS custom
    property resolves against the stylesheet's folder. Fixed with absolute
    asset URLs.
  - The phone header overflowed. It now has a shorter back button and a
    portrait standing over the dialogue box.
- Screenshots: `pj-aghangout/.local/shots/`, ignored. Chromium and WebKit
  each have desktop lobby, long reply, forge work, unavailable and WFE down,
  plus phone lobby, room and history.

Live integration and deployment are Step 5.
