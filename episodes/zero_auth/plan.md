# zero_auth — plan

Implementation plan for `analysis.md`. Decisions taken: D1 — `/mission` may
survive mid-episode while callers still use it, but the episode's exit
condition is **`POST /mission` abolished**, window as the only entrance.
D2 — agforge MinIO/presigned kept. D3 — `AUTOLAB_NODES` allowlist kept.

Ground rules for the implementer: closed experimental cluster — no security
hardening, no backward compatibility. Prohibitions are the minimum three:
don't touch cagent/agcluster auth code, keep the destroy-class tool denies in
agforge, keep secrets out of Git. Everything else is your call.

## Step 1 — open the gateway (agautolab)

Remove auth from `pj-agdev/agautolab/agent/gateway.py`:
- Delete `TOKEN_FILE` (:73), `read_token()` (:98-102), `authorized()`
  (:838-847), its call (:875-876), and the boot refusal (:1124-1125).
- Fix prose that becomes false: module docstring (:21-44) and
  `WINDOW_PROMPT` (:582-583) — for now say missions start via open
  `POST /mission`; Step 5 rewrites this again.
- Keep `window_lock`, summarize one-at-a-time + cache, evidence path
  containment — cost/concurrency guards, reword their comments away from
  "because unauthenticated".

Hints: no test covers auth, so nothing breaks; the deleted boot guard is
what let the gateway run without any `.local/agent/gateway_token`. Verify:
start gateway with no token file, `curl -X POST :8791/mission` → 202/409.

## Step 2 — stop provisioning the token (ansible)

`pj-clusterintent/ansible_agdev/roles/autolab_node/`:
- Delete token generate/slurp/fetch tasks (`tasks/main.yml:50-77`) and
  `autolab_node_token_fetch_dir` (`defaults/main.yml:19-21`).
- **Keep** the cagent human-token copy (`tasks/main.yml:36-42`) — agcluster.
- Optional cleanup: `~/.local/state/autolab-gateway/` on the controller,
  `.local/agent/gateway_token` on nodes.

## Step 3 — redeploy agautolab1

The node runs a stale checkout (still demands a token on `/jobs`). Deploy is
two commands — gitea push then playbook (see `pj-agdev/.local/devenv.md`,
"Updating an autolab node"); ansible with `~/.ssh/ansible_key` is the only
controller channel, plain ssh is refused. Verify open `POST /mission` and
open `GET /jobs` from agstudio.

Known separate issue, don't block on it: `agautolab1.local` resolves to
192.168.0.220, Nautobot desires .130 — leave a note in the report.

## Step 4 — drop the auth-compensation gate (agdevworld)

`pj-agdev/agdevworld/assistant/server.mjs`:
- Delete the method gate (`isOpenPost` + 405, :525-540). Keep the
  `AUTOLAB_NODES` allowlist (D3) and keep-or-drop the `/evidence/` 403 at
  your discretion.
- Update prose: `ROLE_PROMPT` (:104-120), header comment (:14-26),
  `assistant/GUIDE.md:48-52` (re-read per request — no restart needed),
  `README_DEV.md:49-58`, `src/autolabState.ts:91-93`.

Interim state after this step: the assistant can start missions through the
open `/mission`. Hint: test through the Docker container (`localhost:8091`),
not a bare `node assistant/server.mjs` — macOS local-network privacy gives a
bare node process `EHOSTUNREACH` to LAN nodes while curl works.

## Step 5 — give the window mission-starting power

Make the window the real entrance: a conversational request for work must be
able to launch a mission (write `MISSION.md`, spawn `drive.sh`) without any
other door. Mechanism is implementer's discretion — per Tool Giving, prefer
giving the window model an explicit ability over hard-coded intent
detection. Reasonable shapes: a structured marker in the window reply that
the handler executes; or refactor the `/mission` handler body (:873-922)
into a `start_mission()` the window path calls. Keep the existing semantics
as concurrency guards: 409 while a drive is alive, `.local/agent/done`
handling, run id + pid in the response.

Rewrite `WINDOW_PROMPT` and `agent/GUIDE.md` "Doors" so the window knows it
can start work (the old text told it the opposite — that text was
load-bearing: it's served at `GET /guide` *and* injected into every window
prompt).

## Step 6 — abolish POST /mission (exit condition)

- Grep `/mission` across `pj-agdev` and `pj-clusterintent` and retire every
  caller/reference; then delete the route from `do_POST`. Known reference
  sites from the investigation: agdevworld `server.mjs` (gate deleted in
  Step 4 — make sure no residue), `agent/README.md`, `agent/GUIDE.md`,
  `agent/CHARTER.md`, `agautolab/README.md:61`, monitor is GET-only.
- If anything still genuinely needs a deterministic non-conversational
  trigger (scripts, future automation), that's a signal to keep an internal
  function — not the HTTP route.

## Step 7 — delete contradictory rules, record conventions

- Rewrite/delete per the analysis W7 table: `agent/CHARTER.md:53` (drop the
  safety-device item), `agent/GUIDE.md` doors, `agent/README.md` auth
  passages, `agautolab/README.md:61`, `agdevworld/README_DEV.md:55-56`,
  `assistant/GUIDE.md:48-52`.
- Leave historical episode docs and all cagent/agcluster docs untouched.
- Record in `agdevworld/README_DEV.md`: agdevworld sends only read-shaped
  requests to cagent (convention, not enforced — enforcement deferred to the
  future JWT episode).
- Consolidate the scattered "auth designed system-wide later" notes into
  this episode as the single pointer to the JWT vision.

## Step 8 — verify and report

Checklist:
- Gateway boots with no token file on both nodes; no `gateway_token`
  anywhere in the deploy path.
- Browser → assistant :8091 → node window → mission actually running:
  the end-to-end proof of the unified entrance.
- `POST /mission` → 404 on both nodes (exit condition).
- `npm run cluster:fetch` still works (agcluster untouched; cagent has no
  HEAD — use GET when curling :8789).
- Write `report.md` with backend/model per `devpolicy/agent_records.md`,
  note the agautolab1 IP mismatch, and cite `# Single Entrance`
  (`devpolicy/terms.md`) as the policy the unification fulfills.
