# operation_room p2 implementation plan — the conversation/task layer board

Based on p1's findings (p1/report.md), build the conversation/task-layer state
board to completion. This is the only layer that is fully and honestly
observable, and stalled detection is the core value of this screen.

Out of scope (p3 and later): the process/backend-layer tiles (listeners,
ComfyUI, ollama, launchd), routine-fire answer checking, the prompt_id join.

Destructive phase, private experimental environment. Only the minimal
constraints at the end apply; everything else is the implementer's discretion.

## Overview

```
roster from #agents intros ─┐
Zulip initial sweep ────────┼→ state engine in the relay (agentroom) → /ops JSON
Zulip event queue (deltas) ─┘            ↓
                       agdevworld operation room view
```

States follow p1/report.md §2:
`awaiting` / `acked` / `stalled` (default threshold 15 min, configurable) /
`done` (✔), plus a display-level `unknown`.

## Step 1: extend the intro contract (pyagag side; ENT-class prerequisite work)

Include the roster the observer needs (instance name, swept channels and
prefixes) in `#agents` `intro-<instance>`, machine-readably.

- Do not hard-code a roster on the observer side. In p1, a guessed roster
  produced 66 phantom stalled rows (report §3 #9). The intro is already "the
  contract", and the re-post-on-behavior-change convention is established, so
  freshness rides on that convention.
- Where: `pyagag/src/agag/intro.py` (intro generation). Each agent's sweep
  targets live in its `AgentSpec`, so they should transcribe into the intro
  automatically.
- Format is the implementer's choice (fenced block or key: value lines), as
  long as it (a) does not ruin the human-readable introduction and (b) is
  parseable by the observer, and (c) the format itself is documented (in the
  intro or in devpolicy).
- Done when every instance (6 of them) has re-posted its introduction. The
  re-post goes through each agent's own machinery — the observer and humans
  do not ghost-write intros.

## Step 2: a dedicated observer bot

- A full task-layer sweep reaches HTTP 429 at 183 calls (report §3 #10), and
  it spends the same quota the agents use. Mint a dedicated bot (e.g.
  opsroom-bot) to separate quotas.
- Create it via the provisioner credential (the path in
  `AGAG_ZULIP_ADMIN_ENV`) or the Zulip admin UI — either. Store the
  credential per the existing convention (`pj-agdev/.local/zulip/`, env file,
  mode 600).
- Operate this bot **read-only**. A bot post triggers paid agent runs and
  misdirects conversations (a known incident pattern). The one exception is a
  dedicated test channel, created for verification, that no agent watches.

## Step 3: the relay's state engine (`agdevworld/agentroom/`)

Promote the logic of p1's throwaway probes (`.local/opsprobe/`) into a real
implementation. No new service — add an `/ops` endpoint to the agent_room
relay.

- **At startup**: read the roster from the intros; initialize state with a
  full sweep.
- **Steady state**: incremental updates via the Zulip event queue
  (`register` → long-poll `events`). Apply new messages and topic changes
  (including the ✔ rename). No full sweeps in steady state; re-sweep only to
  resynchronize after queue expiry.
- Required building blocks for state computation:
  - The `agag.selfnote` parsers (`parse_served`, `without_selfnotes`,
    `last_real_speaker`). Keep the invariant: a selfnote never counts as a
    speaker.
  - `RESOLVED_TOPIC_PREFIX` (`agag/zulip.py:88`). Key topics by the bare name
    with ✔ stripped, so pre- and post-rename are the same row (the p9
    lesson).
  - The two awaiting routes (owner route / mention route) use the detection
    p1's probes already proved (details in p1/report2.md onward).
- The `/ops` JSON must carry **provenance** per row: the evidence for the
  state (the message id and time of the last real post, whether a matching
  served note exists, which route applied). The UI should only have to
  display it.
- Also report the relay's own health: whether the event queue is alive and
  the last event time. While the queue is dead, serve the data as `unknown`.
- In-memory state is fine (same policy as agent_room: no snapshot files).
  Restart = re-sweep. The stalled threshold is a setting (env or similar).

## Step 4: the operation room view in the frontend

- Add a new view to agdevworld. PanelGridScene reuse vs. a new component is
  discretionary (agent_room got by with reuse plus `panelHeight` /
  `nameFontSize`).
- Only two display requirements:
  1. **Render unknown as an explicit third state.** Do not let relay
     unreachability, queue death, or roster failure look like "all good".
     p9's 26 silent minutes were observationally identical to idle — a board
     that paints unknown in idle colors kills the reason this screen exists.
  2. **Attach provenance to every row** (e.g. "stalled — 17 min since the
     last real post, no served note"). Every green and every red is an
     inference from traces; showing the evidence is the only defense against
     phantom stalls.
- Sort stalled to the top. Grouping (per agent / per project) is
  discretionary.

## Step 5: verification

- **Look at screenshots.** Apply the agent_room step 5 lesson (three visual
  defects that passed every non-visual check) as-is. The ~40-line direct-CDP
  driver from step 5 works; `--headless --screenshot` hangs on
  PanelGridScene's infinite tweens, so don't use it.
- Prove stalled live: in the dedicated test channel (Step 2's exception),
  create an artificial "named, no served note" situation, watch it become
  stalled past the threshold, and watch it clear on reply and on ✔.
- Re-check the phantoms: confirm that with the intro-derived roster, the
  false positives p1 produced for Front and autolab are gone.
- Confirm the event-queue path: a ✔ rename lands as done (a live check of
  bare-topic keying).

## Traps (measured in p1 — read first)

- If the Zulip host is a `.local` name, mDNS lookups can stall for seconds on
  unanswered AAAA queries. If things feel slow, force IPv4 or use the IP.
- macOS Local Network permission attaches per binary. When running the relay
  under a new interpreter or launchd, verify connectivity first (p2 only
  talks to Zulip, but p3's ComfyUI/ollama will definitely hit this).
- `#front` is a public channel. When deciding what the observer bot
  subscribes to, note that public channels are readable without subscribing,
  but mention-search behavior differs (p1's probe implementation is the
  precedent).

## Constraints (minimal)

1. No hard-coded roster on the observer side. Read it from the intros.
2. Steady state is event-queue driven. Full sweeps only at startup and
   resync. Observe with the dedicated bot's credential.
3. Never render unknown as idle/normal.
4. Never count a selfnote as a speaker (go through `agag.selfnote`).
5. The observer bot posts nowhere except the test channel. No credentials
   committed.

Everything else (intro format, state-engine data structures, visual design,
grouping, tuning the initial stalled threshold) is the implementer's
discretion.
