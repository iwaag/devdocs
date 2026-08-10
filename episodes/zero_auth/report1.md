# zero_auth — report, Step 1 (open the gateway)

AI-generated (Omni Agent). Backend: Claude Code / claude-fable-5.
Date: 2026-08-10.

## What was done

Removed all authentication from `pj-agdev/agautolab/agent/gateway.py`
(commit in the agautolab repo, submodule bumped in pj-agdev):

- Deleted `TOKEN_FILE`, `read_token()`, `authorized()`, the `authorized()`
  call inside the `/mission` handler, and the boot refusal that exited when
  `.local/agent/gateway_token` was missing.
- Rewrote the module docstring: no route carries authentication; missions
  start via the open `POST /mission` (interim wording — Step 5 rewrites it
  again when the window gains mission-starting power).
- Rewrote `WINDOW_PROMPT`: the old "you hold no token" sentence is gone;
  interim wording says missions start via the open `POST /mission`.
- Kept the cost/concurrency guards untouched (`window_lock`, summarize
  one-at-a-time + per-iteration cache, evidence path containment) and
  reworded their comments away from "because unauthenticated" — they are
  cost guards now, not auth compensation.

Local state cleanup: `.local/agent/gateway_token` on agstudio was moved
aside to `gateway_token.retired` (used for the no-token boot verification;
final deletion belongs to Step 2's optional cleanup).

## Verification

- `python3 -m py_compile agent/gateway.py` — passes.
- No residue: `grep read_token|TOKEN_FILE|authorized` — zero hits.
- Gateway restarted on agstudio **with no token file present** — boots and
  serves `GET /healthz` → `{"ok": true}`.
- `curl -X POST :8791/mission` with no Authorization header and an empty
  body → `400 body must be {"mission": "..."}` — the request reaches the
  body parser instead of dying at 401, proving the auth gate is gone. A
  deliberately invalid body was used so no real drive launches; the 202
  happy path is exercised in Step 8's end-to-end check.
- As predicted by the plan, no tests broke (no test covered auth).

## Notes

- The restarted gateway currently runs as a child of the Omni Agent's
  session (the harness denied a detached `nohup` start). If it disappears
  after this session, restart with the documented command:
  `cd agautolab && nohup python3 agent/gateway.py > .local/agent/gateway/serve.log 2>&1 &`
- Did "restart local gateway" for agent agautolab/agstudio-node — handoff
  candidate (Deus Ex Machina note per README_DEV.md).
