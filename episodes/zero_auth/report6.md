# zero_auth — report, Step 6 (abolish POST /mission)

AI-generated (Omni Agent). Backend: Claude Code / claude-fable-5.
Date: 2026-08-10.

## What was done

Grepped `/mission` across pj-agdev and pj-clusterintent and retired every
live caller/reference, then deleted the route from `do_POST`:

- `agautolab/agent/gateway.py` — `/mission` branch removed from `do_POST`
  (unknown POST paths now 404); docstring route list, "one mission at a
  time" prose, `start_mission` and `run_row` docstrings updated. The
  internal `start_mission()` stays — the window's mission block is its
  caller, which is exactly the plan's "keep an internal function, not the
  HTTP route".
- `agautolab/agent/GUIDE.md` — Doors: the window is the only entrance;
  the `/mission` line is gone.
- `agautolab/agent/README.md` — the route list leads with `/window` (with
  the mission block), `/mission` entry deleted; "refuses to start without
  a token" and the "every GET is unauthenticated / auth designed
  system-wide later" passages deleted (overlaps with Step 7's W7 table,
  done here where the lines were already in hand).
- `agautolab/agent/CHARTER.md` — the "only authenticated route"
  safety-device item dropped (three → two; the two irreversible-harm
  guards stay).
- `agautolab/README.md:61` — gateway described as monitor + window (the
  entrance that starts missions).
- `agautolab/agent/monitor/monitor.js` — empty-state text now says "ask
  the window to start one" (monitor stays GET-only).
- `agdevworld/assistant/server.mjs` ROLE_PROMPT + `assistant/GUIDE.md` —
  the Step-4 interim `/api/autolab/<node>/mission` lines removed; the
  window line now says asking the window for work is how a mission starts.

Historical episode docs (`devdocs/ent-episodes/autolab_fix/report.md`
etc.) left untouched — dated records, per the analysis.

Nothing was found that genuinely needs a deterministic non-conversational
trigger: the monitor is GET-only and the assistant goes through the
window; `start_mission()` remains as the internal seam should one appear.

## Verification

- `py_compile` passes; grep for `POST /mission`, `"/mission"` route, and
  the assistant mission path over live code/docs → no residue.
- Gateway restarted on agstudio: `GET /healthz` ok,
  `POST /mission {"mission":"x"}` → **404 unknown route** (exit condition
  on this node; agautolab1 gets the same check after the Step 8 redeploy).
