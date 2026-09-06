# agent_room implementation plan

Add the "agent state display through Zulip observation" described in
braindump.md to agdevworld. No snapshot (pre-fetching to files): read Zulip
live at request time.

This is an unpublished, experimental environment, so destructive changes and
dropping backward compatibility are fine. The constraints listed below are
only the ones that would break or contradict something if violated; everything
else is at the implementer's discretion.

## What to build

1. A small backend that reads Zulip live (agdevworld currently has no backend)
2. An agent list built from the `intro-<instance>` topics in `#agents`
3. A flat list of unresolved topics (no `✔ ` prefix) across the `pj-<slug>` /
   `work-<slug>` channels
4. A new frontend view showing all of this as rectangular cards

## 1. Backend

agdevworld is currently "a pure frontend" (see README_DEV.md; the assistant
service was removed wholesale in `modernize_agdevworld` p1). Hitting the Zulip
REST API directly from the browser would expose the API key and hit CORS, so
build a small relay server.

**Model it on an existing implementation**:
`pj-clusterintent/cagent/src/cagent_api/server.py` — the window server
(stdlib `http.server` + `ThreadingHTTPServer`, port 8790, **unauthenticated**).
cagent distinguishes three doors — node (mTLS) / human (bearer) / window
(unauthenticated) — but agent_room is read-only in a private environment, so
one window-style unauthenticated door is enough. No bearer tokens, no mTLS.

**Strongly recommended: do not hand-roll the Zulip reading logic — reuse
`pyagag` (`agag.zulip.ZulipClient`, `pyagag/src/agag/zulip.py`).** Reasons:
- `RESOLVED_TOPIC_PREFIX = "✔ "` (zulip.py:88) is already defined there, and
  `channel_topics()` returns resolved topics too, so re-implementing this
  constant yourself risks drift. There is a real past incident where a lookup
  missed the ✔ rename, read an empty topic, and a mission silently stalled
  for 26 minutes (devdocs/README_DEV.md:175-179). This is the one place to
  ride the existing code.
- Rate-limit handling (`RateLimited`, `rate_limit_backoff`) is already built in.
- `channels()` / `channel_folders()` / `stream_id()` / `topic_history()` and
  the rest are all there.

Hitting the Zulip REST API directly from Node/TS is a viable alternative, but
be aware that it means re-implementing the resolved-prefix handling and the
rate limiting yourself. The choice is the implementer's.

**Credentials**: no read-only-scoped Zulip credential exists today (Zulip has
no concept of a read-only API key). Existing bot credentials live in
`pj-agdev/.local/zulip/*.env` or each agent repository's `.local/zulip.env`
(`front-bot`, `autolab-*-bot`, `forge-bot`, `cagent-bot`, `developer`, etc.).
This is a private experimental environment, so reusing an existing one (e.g.
`developer.env` or `cagent-bot`) instead of minting a new bot with `agag init`
is fine. A dedicated new bot is also fine. Either way — don't let this block
the work. **But never commit the credential file itself** (follow the existing
`.local/` gitignore convention; this also matches the styles.md rule against
local environment info in non-ignored files).

**Enumerating channels for the unresolved-topic list**: code that walks
`pj-<slug>` channels plus their derived `work-<slug>` channels does not exist
anywhere yet (creation/archival code exists, but no read-side enumeration —
the naming rules around `pj-agdev/agautolab/src/agautolab/project_archive.py:100`
are a useful reference). This has to be written fresh. Simply listing
`channels()` and picking names starting with `pj-`, plus `work-`-prefixed
channels in the same channel folder, should suffice. Grouping per project vs.
one flat list is the implementer's call (flat is faster to ship).

**Caching**: the "no snapshot" constraint forbids pre-fetched files, not an
in-memory cache that lives only while the backend process runs. If Zulip API
call volume becomes a concern, a light in-memory cache is fine. Not required.

## 2. Agent list

Use the `intro-<instance>` topics of the `#agents` channel (append-only) as
the source, directly. These are "the contract" (README_DEV.md:184), so the GUI
should not over-process or over-interpret — showing the posted content plainly
is enough.

- Displaying the body of the latest introduction post is sufficient (the
  convention is to re-post after a behavior change). Showing the full history
  as well is fine if wanted.
- **Hide every line tagged `[selfnote]` / `[selfnote][served]`.** These are
  machine-to-machine notes, deliberately hidden even from `chatlog.md` /
  `agentchat read` (README_DEV.md:156-161). Raw Zulip reads will include them,
  so filter them at least once, in either the backend or the frontend.
- Do not display harness/model/backend information (as decided at braindump
  time). The introductions are designed not to carry it, and this matches the
  "Agent ≠ Model" policy (README_DEV.md:211-213).

## 3. Unresolved work list

- The braindump's worry ("topic naming rules differ per agent, so this may be
  hard") is only half right. **Resolved/unresolved itself is a uniform Zulip
  mechanism (the `✔ ` prefix), independent of naming conventions.** What each
  prefix *means* (`workplan-` / `assetplan-` / `workrun-`) differs per agent —
  don't try to unify that; defer it.
- For the first pass, a flat list of "channel name + raw topic name" is enough
  (it also matches the card-display granularity the braindump asked about).
- Which agents/projects to sweep is the implementer's call. All `pj-`-prefixed
  channels should do for a start.

## 4. Frontend view

- The existing `PanelGridScene` (`src/scenes/PanelGridScene.ts`) is one
  config-driven grid scene shared by the four views `nodes` / `workspaces` /
  `autolab` / `tasks` (`src/views.ts`, `src/viewSwitcher.ts`). Adding a fifth
  view config on top of it is the path of least resistance, but a separate new
  component is also allowed (per the user's instruction: either is fine).
- Card contents: agent/instance name, entrance (channel name), intro text,
  unresolved-work count (badge). Click to open the flat list of unresolved
  topics — that much is plenty.
- Leave the existing `chatPanel.ts` (pointed at the dead `/api/chat`) and the
  profile-editing path in `detailPopup.ts` alone. Out of scope.

## Constraints (minimal)

1. No snapshot-file approach for this feature (no build-time / pre-run fetch
   like `scripts/fetch-cluster-state.mjs` → `public/*.json`). The backend
   reads live, per request.
2. Exclude `[selfnote]`-family lines from anything user-facing.
3. Unresolved is decided by the presence/absence of the `✔ ` prefix (do not
   invent a different rule).
4. Do not display harness/model/backend information.
5. Do not commit Zulip credential files (follow the existing `.local`
   convention).

Everything else (backend language, whether to reuse PanelGridScene, grouping
granularity, caching, visual design) is left to the implementer.
