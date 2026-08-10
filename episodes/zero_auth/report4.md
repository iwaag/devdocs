# zero_auth — report, Step 4 (drop the auth-compensation gate)

AI-generated (Omni Agent). Backend: Claude Code / claude-fable-5.
Date: 2026-08-10.

## What was done

`pj-agdev/agdevworld` (commit in the agdevworld repo, submodule bumped):

- `assistant/server.mjs`: deleted the method gate (`isOpenPost` + the 405
  refusal) from the autolab passthrough. Kept, per decision D3, the finite
  `AUTOLAB_NODES` allowlist; kept the `/evidence/` 403 at my discretion —
  it is a data-locality device (raw evidence stays on the producing node),
  unrelated to auth. Rewrote the passthrough's "two safety devices"
  comment accordingly.
- `ROLE_PROMPT`: added the `/api/autolab/<node>/mission` path (interim —
  Step 6 removes it again when the route is abolished).
- `assistant/GUIDE.md`: added the mission path line; rewrote "Safety
  devices" — the "writes refused because this passthrough carries no
  identity" sentence is gone.
- `README_DEV.md` "Safety devices": three guards → two; deleted the 405
  bullet ("`POST /mission` is token-gated … introduce identity and this
  can go").
- `src/autolabState.ts`: comment no longer says an older gateway "still
  demands a token".
- The plan also listed the server.mjs header comment (:14-26); in the
  current file that range holds backend-selection/records prose with
  nothing auth-related — the auth prose lived in the passthrough comments,
  which were rewritten. Nothing to change there.

## Verification (through the Docker container, per the plan's hint)

Rebuilt with `docker compose up --build -d assistant`, then from agstudio:

- `GET localhost:8091/api/autolab/nodes` → both nodes reachable
  (`agstudio`, `agautolab1`, status 200 each).
- `POST localhost:8091/api/autolab/agstudio/mission` (empty body) → the
  node's own `400 body must be {"mission": "..."}` comes back verbatim —
  formerly this died at the assistant's 405. Same result via
  `agautolab1`. Empty body on purpose: proves the gate is gone on both
  nodes without launching a real drive.
- `/api/autolab/agstudio/jobs/<job>/evidence/...` → still
  `403 evidence_not_proxied` (kept device works).

Interim state reached: the assistant can start missions through the open
`/mission`.
