# Agent session p1 ex3 — step 1: baseline, task contracts and budget

Date: 2026-10-15. Frozen before the first model call. The frozen contract
(exact request texts, staged follow-ups, scripted replies, acceptance
criteria, order and budget) is `contract.json` in the ignored notes
`pj-agdev/.local/agent-session-p1-ex3/step1/`. The driver reads its texts
from that file, so they cannot drift between arms.

## State before relying on services

- `nctl status`: Nautobot 3.1.3 authenticated, worker idle (0 pending
  jobs), all five submodules clean, `ok: True`.
- `wfe observe`: executor available. The only run is jevlab
  `experiment/run-001`, held `stopped` by iwaag before plan agreement. It is
  not touched. Nothing is queued and no attempt is open; no agent process
  is running on the host.
- Repositories: pj-agdev, agdevworld and agautolab are on `main`, equal to
  origin, clean.

## Frozen baseline

| Item | Value |
| --- | --- |
| WFE (deployed) | agdevworld `a642e27` release, `service/current`; `wfe version` agrees. This is ex2's compact continuation. |
| Executor | agautolab `1f1ca20` code (`9382b1b` tests), `com.agdev.agautolab-wfexec` running and idle |
| Harness | claude_code 2.1.283 (the extension's native binary), `claude-sonnet-5`, the host's default profile, the executor's tool allowlist, Max subscription |
| Common contract | wfrun `guide.md` sha256 `cf7ff7b2…` (agautolab `1f1ca20`), unchanged since ex2 |
| Project guide | RTS `agent/review.md` sha256 `fd1fa285…` |
| Node catalog | `node-catalog.json` sha256 `1d35e919…` (3 templates) |
| Documentation policy | `default` (from `improve-game`) |
| RTS baseline | pj-rts-vs-bot `979d6a7` (the production workspace, equal to origin) with study-rts `d2734db` |
| Environment | the driver drops the Omni session's `CLAUDE*`/`ANTHROPIC*` variables (launchd-like), as in ex2 |

Every run gets the same trial workflow commit on top of `979d6a7`: two
copies of `improve-game` that differ only in id, name and
`execution.session` (`fresh` / `run`), with the Zulip intake line removed
(trial input routing). Nothing else in the workflow, guides or catalog
changes.

**Isolation.** Each of the six runs gets its own private area (bare remotes,
workspace clone, registry, exec store, presenter ports) in the session
scratchpad, cloned from that one trial commit. So every run starts from the
same commit, and an arm cannot see the other arm's run records or code. In
ex2 both arms shared a workspace. Run copies (p9) publish only to their run
branch, so nothing reaches `main`. The trial cwd is outside
`/Users/eiji/projects`, so the workspace's own `.claude/settings.json` does
not reach the agents (ex2 finding).

## Task candidates rechecked at the baseline

`game/` is 1,693 lines (`sim.js` 657, `main.js` 627, `bot.js` 157, a
headless test 233). The candidates are absent:

- **A, rally points**: no rally state or command. A trained unit spawns
  toward the map centre; a worker is sent to gather (`updateBuilding`).
- **B, waypoints**: orders are a single `order` per unit; Shift is used only
  for additive selection.
- **C, save/load**: no `localStorage`, serialization or load path. The
  state is mostly plain data plus a derived `byId` map. A `window.rtsDebug`
  hook (`game()`, `fastForward()`) already exists for checks.

None needed a substitute.

## Frozen task contracts

The requests are in Japanese, the language of earlier RTS requests. The
summaries below are in English; the exact texts are in `contract.json`.

| Set | Request (summary) | Staged follow-up (first result check) |
| --- | --- | --- |
| A | Rally points for HQ and barracks: select own building, designate ground, a visible marker, help text; units trained afterwards (workers too) move there; existing units unaffected; a changed point affects later production; restart clears it; a simulation command refusing other players' and missing buildings; headless tests for validation and spawn-to-movement. | Add a clear-rally action; units trained afterwards behave as before; units already heading there may continue. |
| B | Shift-right-click appends a movement destination; units visit them in order; a plain move replaces, Stop clears; selected units' routes drawn; dead units handled safely; attack/gather/build keep working, with stated queue semantics; no attack/build/gather queues or pathfinding rewrite; headless tests for append/replace/stop/order. | Number each waypoint on the route in arrival order, so the remaining count is visible. |
| C | One local slot with Save/Load controls and a versioned format; restore time, resources, entities with ids, orders, production queues, research and bot state, and continue; reject incompatible/corrupt data without damaging the live match; rebuild references and reset transient UI/timing; check round trip, continuation, corrupt data and a browser reload; no cloud, replay or migration. | Show the saved match's time; disable Load when no valid slot exists. |

**Acceptance criteria** (judged in step 4 from the arm's own branch, not
from the agent's claims):

- **A**:
  - command validation;
  - a trained unit moves to the point;
  - existing units unaffected, and a change applies to later production
    only;
  - help and a marker;
  - restart clears the points;
  - follow-up clear;
  - existing checks pass, and the feature is tested.
- **B**:
  - ordered append;
  - replace and Stop;
  - a dead unit is safe;
  - routes drawn;
  - other commands work, with their queue semantics stated;
  - numbered waypoints;
  - existing checks pass, and the feature is tested.
- **C**:
  - one slot, controls and a version;
  - round-trip state;
  - continued simulation with rebuilt references;
  - corrupt or incompatible data rejected, live match intact;
  - transient state reset;
  - a save survives a browser reload;
  - the saved time is shown and Load is disabled without a valid slot;
  - existing checks pass, and the round trip is tested.

The staged follow-ups are disclosed here as staged requirements, used to
exercise continuation. They were chosen before any run, are never in the
initial request, and are sent in both arms as the answer to the first
result check.

## Scripted person inputs

All inputs are labelled `Trial Person (scripted trial input)`. They are
recognised by the recorded question, not by node name (ex2's rule):

- **A plan agreement** (the question records a plan document):
  「この計画で進めてください。」
- **The first result check** (a presentation, deliverables, or a node that
  confirms the result): the task's staged follow-up.
- **A later result check**: answered only after objective evaluation of
  the arm's presented result:
  - It passes: 「確認しました。これで合格です。」, the only acceptance text.
  - It fails: one defect-repair answer naming the observed defects. At
    most one per arm.
  - A run still failing after its repair is stopped and reported as
    incomplete.
- **Any other question**: 「お任せします。依頼の範囲で、あなたの判断で進めてください。」.
  This is never acceptance. Exact replies are logged.

Answers are delivered only after the attempt has ended and its process is
gone (the executor no longer holds it). In ex2 an answer arrived while the
fresh arm's attempt was still running, so that arm continued in its own
process. The evaluation pause before a later result check is a natural
wait; it is recorded.

## Order and budget

- **Order**: A fresh → A run → B run → B fresh → C fresh → C run. There is
  one execution slot, and both arms of a set finish before the next set.
- **Cap**: $45 reported list-price cost for all six runs, including failed
  attempts and repairs.
  - Per run: $9 and 45 minutes of active time.
  - Between attempts, completed attempts count at their reported cost.
  - A running attempt counts at a live estimate from its stream log. This
    estimate is priced at list rates and was calibrated against ex2's logs,
    where it was 0–30 % above the reported figures. It errs toward stopping
    early.
  - At a ceiling the driver stops the run and the evidence is kept. There
    is no replacement and no budget increase.
- **Fit**: ex2's small display change cost $1.84–3.02 and 9–16 active
  minutes per run. Tasks two to three times larger fit the ceilings; save/load
  is the tightest and is constrained to one slot and the current schema.
  No contract was revised.
