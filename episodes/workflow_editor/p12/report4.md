# p12 Step 4 — validate, deploy and exercise the workflows

## Tests and checks

- WFE full suite in each release build (`npm run check`): 387 tests at
  `a6161ec`, plus the share-folder and integration-queue tests added later;
  `tsc` clean. Focused: `test/boast.test.ts` (revision-consistent assets,
  preview, absent vs failed, boundaries, headers, ranges, shares) and
  `test/integration.test.ts` (new p12 case).
- `checks/boast.ts` (24 checks) passed in Chromium and WebKit before
  deployment (report2).

## State read before runtime work

`nctl status`: Nautobot, worker, dumps and submodules ok; `nctl drift
--host agstudio`: converged (agdevworld-web applied). WFE release
`a129662`, no open attempt, executor idle.

## Deployments (agdevworld `scripts/wfe-release.sh`)

| Release | Why | Result |
| --- | --- | --- |
| `a6161ec` | Steps 1–3 | service and `bin/wfe` report it |
| `2dcd056` | shared folder references had no display URL (found on the first playprey page) | healthy |
| `a1bfedf` | acceptance inside an attempt did not queue integration (found in the RTS trial) | build checked; activation refused while an attempt was open, activated when the slot freed; healthy |

The `/wfe/` nginx route needed no change: state, commit-addressed page,
assets, `play/` subpage and inline share listings all answered through
`http://<host>:8093/wfe/`.

## Initial pages (Step 4.3)

Requested `boast` in both projects with a short request leaving content to
the agents.

- **rts-vs-bot `boast/run-001`**: the agent added present/human-check/cleanup
  nodes itself, wrote a Japanese page (hero, "play now" link, screenshots,
  capability cards, controls) and put a playable copy of `game/` in
  `boast/play/`. Accepted (answered as Omni Agent for the Developer);
  integrated fast-forward `c0cb45f`.
- **playprey `boast/run-001`**: an English page of five onboarded cases with
  status badges, lessons and "Open delivery files" buttons that use the
  `resolve-shared` message. Accepted; integrated `578cd25`.
- Deployed inspection (Chromium and WebKit through `/wfe/`): opaque origin,
  no failed requests or errors, the RTS play page opens alone with its
  canvas. The playprey button resolved a **folder** reference and showed
  "not reachable" because the display URL was only given for files → fixed
  in `2dcd056`; the button now opens a browsable listing of the delivery.

## Delegation trials (Step 4.4)

**rts-vs-bot `improve-game/run-006` → `boast/run-002`** (request: an
elapsed-time clock on the HUD).

- implement → verify → present → human-check: accepted after viewing the
  presentation; the parent's agent recorded the decision from the answer.
- Dispatch d1 with an explicit request: branch
  `wfe/rts-vs-bot/improve-game/run-006`, commit `532d1ca`, tests, what was
  not verified, the person's acceptance, and "not yet integrated".
- The child was created by the intake from the request post, asked its
  confirmation **to the parent's agent**, which compared the embedded copy
  with the implementation and accepted; the child completed and integrated
  (fast-forward `8401b6f`); the parent took up the result, disposed
  `satisfied`, cleaned up, completed and integrated by merge `0554425`.
- Order observed: **child before parent**. The decision recorded inside the
  attempt did not queue integration (only completion did), so the parent's
  result could not reach main while it waited. The child did not substitute
  old main: it built `boast/play/` from the named commit, so what the page
  advertised was reachable at the child's integration; `game/` on main
  followed about a minute later. Fixed in `a1bfedf`: an acceptance recorded
  during an attempt queues integration when the attempt ends (regression
  test added). Live proof of the fixed order is not available (below).
- Final main: `boast/play/main.js` equals `game/main.js`; the page has the
  clock card; the parent's merge kept the child's `boast/` changes.

**playprey `onboard/run-005`** (re-deliver eagle_beakfit): the agent
dropped every node it did not need **including `boast`** ("profile
unchanged"), published `o1` (1548 files, 55.4 MiB, verified) and completed.
No delegation — the workflow leaves that choice to the agent.

**playprey `onboard/run-006` → `boast/run-002`** (request: let the Boast
page point to the newest delivery): the agent reused run-005's published
`o1` and kept only `boast`. The child changed the eagle_beakfit card to
`shared://agshare/runs/playprey/onboard/run-005/o1/`, asked the parent's
agent, was accepted, completed and integrated (`810ab2a`); the parent
disposed `satisfied`, completed and merged (`504ad1d`) with `boast/`
preserved. The advertised output is a durable share output, available
before any integration. In the disposition the parent's agent wrote
"confirmed by Developer"; the child's confirmation was answered by the
parent's agent (the Developer's confirmation was run-005's).

No held historical run was restarted; the three stopped runs were not
touched. All changed workflow graphs validated before use.

## Not verified live

- The fixed order (parent result integrated while it waits, then the
  child): no trial after `a1bfedf` recorded an acceptance and then waited
  on a delegation in a later attempt. Covered by the unit test only.
- A real Git conflict between a parent and a child: none occurred.
- The manual copy dialog on the person's own browser: both engines copied
  through `execCommand` in the checks.

## Follow-up after the Developer's review (2026-10-18)

The Developer found that the playprey page did not say where the Godot
project can actually be tried (which PC, which path). My Step 4 check had
only confirmed that the page's buttons worked, not that the page answered
the braindump's playprey example. Three `boast` runs followed, each from
the Developer's words or a reviewed finding:

| Run | Result | Review |
| --- | --- | --- |
| `boast/run-003` p1 | "Try it live": machine, checkout path, `Launch.command` steps | sent back: it said wolf/cheetah/horse_combined play from the checkout, but the checkout has no runtime assets (`model_profiles.py check` fails); no copy action |
| `boast/run-003` p2 | the checkout is marked "needs assets first" with the fetch command; delivery cards resolve `shared://` in the panel to machine, path and launch commands; copy buttons | accepted; in the deployed panel the cards resolved and copy reported "Copied to the clipboard". After accepting, I saw the launch steps imported **in place on the share**, against `knowledge/delivery.md` |
| `boast/run-004` | copy off the share, verify, import twice, launch | the steps were run on a copy of run-005 `o1`: verify passed, import succeeded, Godot loaded `eagle_beakfit` headless; the share's output gained no `.godot/` |

Pages on main: playprey `5f4af48`.

**Model.** The executor's attempts had run Claude Sonnet 5.0
(`claude-sonnet-5`): `agautolab/agents.toml` pinned it, and the CLI glob
pointed at a stale `~/.vscode-server` copy of Claude Code (2.1.283). The
`sonnet` profile now uses the Claude Code alias `anthropic/sonnet`, and the
ignored overlay's glob points at `~/.vscode/extensions` (newest by mtime,
2.1.301). The alias resolves to `claude-sonnet-5-5` there; after restarting
`com.agdev.agautolab-wfexec`, the next attempt's log shows
`claude-sonnet-5-5` (agautolab `a73491a`, pj-agdev pin `da09897`). Front,
Observer, forge and cagent still pin `claude-sonnet-5` in their own
`agents.toml` files.
