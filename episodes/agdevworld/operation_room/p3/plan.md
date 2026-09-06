# operation_room p3 implementation plan — routine display and the agfront chat

Based on the braindump (p3/braindump.md) and its review. Scope: the routine
list, a per-routine chat with agfront, and progress display for the selected
session. The full process/backend-layer tiles are p4. Line 7 of the braindump
— posting an aggregate of all live topic information back into Zulip — is
**not adopted**: the p2 relay already performs that aggregation event-driven,
and writing it back to Zulip would only create a second source of truth that
can go stale (if it has value for in-system agents, that is a separate
episode).

Precondition: the relay is a launchd resident (p2ex2). Code reload is
`launchctl kickstart -k gui/$(id -u)/com.agdev.agentroom`.

## Overview

```
schedule.json (local read) ───┐
#front routine-* topics ──────┼→ relay /routines, /routines/<name>  (reads: observer bot)
#front front-routine-* fires ─┘
[selfnote][rootchat] traces ──→ relay reconstructs the session tree internally
chat send ────────────────────→ relay POST /chat  (write: developer credential)
```

## Step 1: routine reads in the relay (`/routines`)

- **Roster**: the `routine-*` topics in `#front` (standing requests); the
  fire conversations are `front-routine-<name>`. The observer bot already
  subscribes to both via the event queue, so the marginal cost is near zero.
- **schedule.json**: the relay reads
  `pj-agdev/.local/rtschedule/schedule.json` **directly as a local file**. Do
  not fetch the :8093 routine-gui — http.server sends no CORS headers, so the
  browser cannot read it directly, which is exactly why this goes through the
  relay. Pass the path via the plist env (no absolute paths in non-ignored
  files). Schema: `devenv/routine/dispatch.py:50-95` (`requests[]` /
  `events[]`, `fired_at`, 7-day pruning).
- Per routine, expose: the name, the standing request text, the next
  scheduled fire / the most recent fire, and **whether the last fire was
  answered** (did a Front reply follow the fire post — this implements the
  old p3 item "routine-fire answers" here).

## Step 2: session tree reconstruction

A "running routine session" = the tree of conversations spawned by one fire.
Root it at the fire in `front-routine-<name>` and connect what Front opened
or called (workplan-, workrun-, …) by following the
`[selfnote][rootchat] <channel>/<topic>` traces.

- The relay already sees every post in the realm through the event queue, so
  the rootchat notes are already received. **Never display them, but using
  them as internal link data is allowed** (both invariants hold: a selfnote
  never counts as a speaker, and never reaches the user).
- Stamp each tree node with the existing /ops state
  (awaiting/acked/stalled/done). Do not implement state computation twice.
- List **at most the 3 most recent sessions** (the braindump's number).
  Older ones only need to be reachable via drill-down.

## Step 3: chat send (`POST /chat`)

agdevworld's first write path. Keep the separation strict:

- **A credential separate from the observer bot.** Pass the developer's env
  via something like `AGENTROOM_CHAT_ZULIP_ENV`. If unset, chat is read-only
  and the view says so (the same degradation pattern as no
  OPSROOM_ZULIP_ENV → 503). No fallback to the observer bot.
- **Posts go only to routine-related topics in `#front`** (enforced in the
  relay). The GUI never writes directly into other agents' channels —
  dispatch is Front's job; that is what Single Entrance means.
- A post as the developer starting a paid Front run is the feature (that is
  what chat is). But **Zulip silently truncates over-long posts** (proven
  with comfynotify), so add an input length guard.

## Step 4: the routines view in the frontend

- Routine list cards (Step 1's content; highlight any unanswered fire).
- Selecting a routine shows: its session trees (max 3 sessions) and the
  **history-carrying chat** on `front-routine-<name>`. For the chat UI,
  finally make the dead `chatPanel.ts` what README_DEV promised — a thin
  wrapper over a Zulip topic in `#front`; this is the "later phase" arriving.
  History shows real posts only, selfnotes stripped.
- **The high-frequency checking of the selected session applies to non-Zulip
  signals only.** The Zulip side is already realtime via the event queue —
  add nothing there (polling that walks back the 429 lesson is not allowed).
  The non-Zulip signal, for now, is in-flight harness detection for the
  agents involved (a role workspace dir with no `run-NNNN.json` newer than it
  = a run in flight; p1's proven signal). The relay polls this every few
  seconds, and only for the agents involved in the selected session.
- The two principles — unknown ≠ idle, provenance on screen — apply to this
  view unchanged.

## Step 5: verification

- Screenshots (`.local/opsshot.mjs`). Visual verification has caught defects
  four phases running — mandatory again.
- Verify the read side (list, session trees, history) against real data;
  past routine fires exist, so the material is there.
- **Exactly one real chat round trip, deliberately**: send a short question
  from the GUI into a real routine topic, watch Front answer, and watch the
  answer appear in the chat history in real time. This buys one paid run.
  No automated tests or retries that post (unit tests for the send path mock
  Zulip).
- Confirm schedule.json is readable under launchd (path, permissions).

## Traps and hints

- Front's reply always goes to its `front-*` conversation with the developer
  (the p8 contract). A conversation held in the fire topic gets its answers
  there, so the chat view can simply read per topic.
- `#front` is public. Fires and replies already reach the observer bot's
  event queue (realm-wide subscription).
- The dispatcher rewrites schedule.json at will. Re-read it on every request
  (it is small; no inotify needed).
- After sending, wait for your own post to come back via the event queue
  before showing it in history (or dedupe if you render optimistically).

## Constraints (minimal)

1. The observer bot does not write to Zulip. Chat writes go through the
   dedicated env (developer), with no fallback.
2. GUI posts go only to routine-related topics in `#front` (enforced in the
   relay).
3. No added Zulip polling. High frequency is for non-Zulip signals, and only
   for the selected session.
4. Selfnotes are never displayed (internal link reconstruction may use them).
5. Tests that really post are manual and minimal in number. No credentials
   committed.

Everything else (session-tree data structures, chat UI look, in-flight poll
interval, whether this is a mode inside the ops view or its own view) is the
implementer's discretion.
