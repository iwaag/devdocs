# zulip_channel_topic2 — plan

Date: 2026-08-12. Braindump: `braindump.txt`.
Prior art: `../zulip_channel_topic/report.md` (FreeForge topic-per-request),
`pj-agdev/devdocs/episodes/agautolab/external_taskman/s5/` (prep/dev separation, mission-ignore failure).

Goal: prove the standing-channel / disposable-topic pattern generalizes to
autolab projects: one `#pj-<name>` channel per project, one `mission-*` topic
per mission, only the autolab side reacts. This is a **workflow test**, not a
coding-agent performance test — a thin game implementation is fine, but a
mission that never creates an autolab job is a workflow failure.

This environment is a private experimental LAN. No security hardening, no
backward compatibility, no migration paths required. Delete and replace
freely. Prohibitions in this plan are the minimum needed to keep the test
meaningful; everything else is implementer's discretion.

## Decisions fixed by this plan

- **Project**: `whack-a-mole`. Channel `#pj-whack-a-mole`, repos
  `autodev/whack-a-mole` + `autodev/whack-a-mole-direction`.
- **Mission 1 (game concept, specified here per braindump 2-1)**: a
  whack-a-mole browser game. Single static page; moles pop up at random in a
  3x3 grid, clicking/tapping one scores a point, 30-second round, final score
  shown with a restart button. Mouse and touch. No backend, no assets beyond
  what the coding agent draws with CSS/canvas/emoji.
- **Mission 2**: at game start, show a staff roll (credits) that the player
  can skip with a click; then the game begins as before.
- **Topic naming**: `mission-YYYYMMDD-HHMMSS-<6 hex>` — same generator shape
  as FreeForge's `requestTopicName` (`agdevworld/assistant/zulip.mjs:138`).
- **Routing rule**: prefix-only, channel-agnostic, exactly like FreeForge.
  agforge fires on topic prefix `create-`, autolab fires on `mission-`.
  Disjoint prefixes are the whole filter; do not add channel allowlists
  unless Step 1 disproves this.
- **Listener placement**: a new launchd-resident Zulip listener on agstudio
  (mirror of `com.agdev.agforge-zulip`), credentials
  `agautolab/.local/zulip.env` (bot already provisioned). Handler is a
  **bridge to the node window** (`POST /window`), not a new conversational
  agent — the window stays the node's single desire entrance; the listener is
  a transport in front of it. Record the two-entrance interim as debt, as
  zulip_receive/zulip_channel_topic did.
- **Resolution**: `✔ ` rename via the existing `resolve_topic` mechanics. In
  Phase A the Omni Agent resolves directly; in Phase B the assistant gets a
  `POST /api/autolab/missions/resolve` endpoint (clone of
  `/api/freeforge/resolve`).
- **Plane multi-project**: stop hard-wiring one project UUID. Minimal cut:
  node `.local/plane.env` keeps credentials + state IDs only; the Plane
  project UUID travels in the mission text (there is precedent — mission text
  already carries the Plane issue ID, `agdevworld/src/planeState.ts:96-108`).
  Assistant passthrough (`assistant/plane-passthrough.mjs`) must accept a
  project ID per request instead of the pinned one. Breaking the old
  single-project config is fine.

## Budgets

Budget generously everywhere; do not economize.

- Autolab missions: pass an explicit `max_sessions` of **20+** for
  implementation missions and **10+** for bootstrap missions. Never accept a
  window-chosen budget of 2 (that starved S5).
- Window runs, listener test runs, live Zulip runs: run as many as needed;
  paid runs in the tens are acceptable for this episode.
- Implementation agents working these steps: no token thrift; prefer one
  thorough pass over several timid ones.

## Step 1 — Firing-rule spike (curl only, no code)

Prove the routing before writing anything.

1. With the autolab bot (`agautolab/.local/zulip.env`): create channel
   `#pj-spike`, subscribe all agent bots (forge 13, devworld-assistant 10,
   cagent 14, autolab) + Developer (8).
2. Post a message under topic `mission-<stamp>-spike` in `#pj-spike`.
3. Verify **negative firing**: agforge listener log
   (`agforge/.local/out/zulip-listener.log`) shows no run; cagent listener
   (DM-only) shows no reaction. Post a `create-*` topic in the same channel
   and confirm agforge *does* fire there (channel-agnostic prefix behavior
   confirmed as feature, not accident) — set `AGFORGE_ZULIP_LOG_ONLY=1`
   first if you want that check free, or accept one paid run.
4. Resolve both topics with the `✔ ` PATCH; confirm resolved topics stop
   matching.

Hints: the FreeForge Step-1 spike (`../zulip_channel_topic/report1.md`) is the
template — realm defaults already allowed bot channel creation and
foreign-topic resolution, so expect zero permission changes. Zulip API base
`https://agstudio.local:8543`, self-signed TLS (`-k`). Report: `report1.md`.

## Step 2 — Fix the known dev-start killers (precondition, braindump 1-4)

The prep/dev separation *structure* already exists (S5 Step 0 vs Step 1);
what broke was dev start itself. Fix the two recorded causes before Phase A:

1. **Mediator mission-ignore** (S5's direct cause of death: both sessions
   answered "what should I build?" and exit 10'd with zero autolab jobs).
   Apply the ENT recommendation from `s5/report.md`: make mission consumption
   observable at mediator startup (e.g. the session record proves
   `MISSION.md` was read), and treat a generic-entrance reply as a failed
   session so `drive.sh`/session accounting surfaces it. Do **not** solve
   this with task-specific prompt padding.
2. **CLAUDE.md leak**: harness project instructions from above the working
   directory reach in-system agents and have killed runs on permission
   denials (`agautolab/agent/CHARTER.md:11-15` warns about this). Verify the
   node-side execution directory placement blocks the leak; fix placement if
   not.

Acceptance: one throwaway mission on a node (any trivial task) creates an
autolab job, and the session record shows mission consumption. Deploy to
agautolab1 via the usual gitea-push + ansible route
(`pj-agdev/.local/devenv.md` "Updating an autolab node"). Report:
`report2.md`.

## Step 3 — Phase A: Omni Agent manual walkthrough

Omni executes the whole procedure by hand, using in-system agents only for
the actual coding (braindump line 22). Leave a Deus-Ex-Machina one-liner per
handoff candidate.

**3a. Project start (no tasks planned or started):**
- Gitea: create `autodev/whack-a-mole` + `-direction` via API
  (`POST /api/v1/orgs/autodev/repos`, token
  `agautolab/.local/gitea/autolab-agent.token`), seed the direction repo
  (`GUIDE.md`, `concept.md`, `.gitignore` with `.local`) per
  `agautolab/AGENT_GUIDE.md:104-121`; register in the node's
  `.local/projects/projects.md` clone layout.
- Plane: create a fresh project for `whack-a-mole` in workspace `agautolab`
  (credentials: `pj-agdev/.local/plane-credentials.env`); note its UUID and
  state IDs for the mission text.
- Zulip: create `#pj-whack-a-mole`, subscribe all agent bots + Developer.
- **Prohibition (the one that matters)**: no Plane issue, no task planning,
  no mission in this sub-step. Prep and dev start must be separable in the
  evidence.

**3b. Mission 1**: post the whack-a-mole mission (concept above + Plane
project/issue references) as topic `mission-<stamp>-<hex>` in
`#pj-whack-a-mole`; Omni manually bridges the topic text to the node's
`POST /window` with a generous `max_sessions`; the coding agent implements.
When the driver finishes and the result is acceptable-as-workflow-evidence,
post the outcome in-topic and resolve (`✔`).

**3c. Mission 2**: same flow with the staff-roll mission. Post it only after
Mission 1's driver has fully exited — the gateway 409s concurrent missions.

Hints: node choice is agautolab1 (or the agstudio gateway if the node
misbehaves — either is fine, note which). "Implementation may be thin" applies
to the game, not the workflow: each mission must produce an autolab job.
Report: `report3.md` (+ `report3a/b/c.md` if splitting helps).

## Step 4 — Workflow implementation

Turn the walkthrough into standing machinery. Suggested cut lines (rearrange
freely):

1. **agautolab Zulip listener** (new, in `agautolab/`): pyagag
   `serve(client, handler, accept=...)` with
   `accept = channel message && topic.startswith("mission-")` — copy the
   shape of `agforge/src/agforge/zulip_listener.py`. Handler: forward the
   topic content to the configured node's `/window`, reply in-topic with the
   window answer; if a mission starts, poll `GET /status` and post the
   terminal outcome (done-note or budget exhaustion) in-topic. A 409 (mission
   already running) is just reported in-topic. launchd unit modeled on
   `devenv/launchd/com.agdev.agforge-zulip.plist.in`.
2. **Assistant endpoints** (`agdevworld/assistant/server.mjs`), mirroring the
   freeforge trio:
   - `POST /api/autolab/projects` — project start: Gitea repo pair (or
     delegate repo creation to a bootstrap mission, implementer's choice —
     S5 Step 0 proves the mission route works but cost 6 window runs),
     Plane project creation, `#pj-<name>` channel creation + subscriptions.
     Needs `create_channel` added to `assistant/zulip.mjs` (pyagag has the
     reference implementation, `agag/zulip.py:188`).
   - `POST /api/autolab/missions` — post a `mission-*` topic into a project
     channel.
   - `POST /api/autolab/missions/resolve` — `✔` rename, clone of
     `/api/freeforge/resolve`.
   Document all three in the assistant `GUIDE.md` (Tool Giving — the agent
   only uses what it can read about).
3. **Plane multi-project change** as fixed above. Break the old pinned-UUID
   config without compatibility shims; update the S5-era `Three Choice Quiz`
   wiring only if something still depends on it.

Tests: follow the existing patterns (`agforge/tests/test_zulip.py`,
`assistant/tests/zulip.test.mjs`). Live smoke of the listener against
`#pj-spike` from Step 1 is cheaper than debugging in Step 5. Report:
`report4.md`.

## Step 5 — Phase B: end-to-end through the assistant

1. Omni Agent asks the devworld assistant, in one conversation, to start
   project `whack-a-mole-2` (fresh name so Phase B provisions from zero) and
   run the two missions (same concepts as Step 3).
2. Assistant drives: project start → mission 1 topic → autolab implements →
   resolve → mission 2 topic → implement → resolve.
3. Acceptance:
   - both mission topics resolved (`✔`), each having produced a real autolab
     job;
   - Gitea repo pair, Plane project, and Zulip channel all exist for the
     project;
   - **only autolab reacted** to `mission-*` topics — check agforge and
     cagent listener logs for silence (braindump's final assertion);
   - prep and dev start are separate, evidenced steps.
4. Omni interventions during this step are failures of the workflow — allowed
   for rescue, but each one is recorded as a Deus-Ex-Machina note and feeds
   the debt list.

Report: `report5.md`, then the episode `report.md` rollup with records
(profiles, costs, run counts) and the debt/seeds section (two-entrance
interim, unresolved-topic message cost, whatever Step 5 surfaces).
