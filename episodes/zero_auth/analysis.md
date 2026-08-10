# zero_auth — analysis

Analysis for `braindump.md`: make pj-agdev consistently auth-free (single-user,
dedicated experimental cluster), unify agautolab's entrance into the window,
keep agcluster (cagent) auth with agdevworld restricted to reads, and delete
contradictory development rules. AI-generated (Omni Agent), based on code
investigation on 2026-08-10.

## 1. Current state (verified)

Cluster health at investigation time: Nautobot reachable/authenticated, one
Celery worker running, no pending jobs (`nctl status`). Services live on
agstudio: nginx web :8090, assistant :8091, agforge :8092, autolab gateway
:8791, cagent human entrance :8789. Vite :5173 stopped. `agautolab1.local:8791`
answers `/healthz` but runs a stale checkout that still demands a bearer token
on `/jobs`.

### Where auth actually exists today

The surprise of the investigation: **pj-agdev is already almost auth-free.**
The inventory of real authentication is short:

| # | Mechanism | Component | Direction | Verdict under braindump |
|---|---|---|---|---|
| 1 | Bearer token on `POST /mission` (`.local/agent/gateway_token`) | agautolab gateway :8791 | inbound | **Remove** — the only inbound auth in all of pj-agdev |
| 2 | Gateway refuses to boot without the token file (`gateway.py:1124-1125`) | agautolab | startup guard | **Remove** (prerequisite for #1) |
| 3 | cagent human token (`~/.local/state/cagent/human_token`) | agdevworld `cluster:fetch` → cagent :8789 | outbound | **Keep** — agcluster auth is retained |
| 4 | MinIO access/secret key + presigned URLs | agforge → MinIO :9100 | outbound + delivery | Keep (see decision D2) |
| 5 | Gitea API token (`.local/gitea/autolab-agent.token`) | agautolab deploy path | outbound | Keep — infra credential, not an agent gate |
| 6 | `ANTHROPIC_API_KEY` | agdevworld assistant | outbound (SaaS) | Keep — external provider, out of scope |

Everything else is already open:

- **agautolab** (`agent/gateway.py`): all `GET` routes, `POST /window`, and
  `POST /jobs/<job>/summarize/<iter>` are unauthenticated. `POST /mission` is
  the single authenticated route (`authorized()` at `gateway.py:838-847`,
  called at `:875`). The auth path has zero test coverage.
- **agdevworld** (`assistant/server.mjs` :8091): no auth on any endpoint.
  The passthrough to autolab nodes deliberately carries no token; its method
  gate (`server.mjs:525-540`) 405s `POST /mission` *because* it holds no
  token — the guard is a compensation for auth, not a policy of its own.
- **agforge** (`service/request_service.py` :8092): no auth at all.
  `POST /api/requests` can be driven by anyone who can reach the port; job
  ids (`uuid4`) are the only privacy. Already conforms to the braindump.
- **agcluster / cagent**: human entrance :8789 = static bearer token
  (`cagent/src/cagent_api/auth.py:87-107`), node entrance :8788 = mTLS +
  ledger + DesiredNode check. No no-auth mode exists; token has no scopes,
  expiry, or revocation. **Crucially, the API layer has no read/write
  distinction** — the same token that reads also permits `POST /requests`,
  which is what drives cluster mutation. The only "cannot destroy"
  enforcement is a command-string deny list inside the shared OpenCode
  process (`cagent/opencode/config.json.template:14-34`), explicitly
  documented as "not a capability boundary".

### The mission/window split

The split is deliberate design, not accident: the window's charter of
"cannot start work" is enforced by *the token's existence*, not by prompt
instruction (`gateway.py:21-35`). Removing the token dissolves the split's
rationale — which is exactly what the braindump asks for ("windowに統一").
Consequence: `WINDOW_PROMPT` (`gateway.py:582-583`) and `agent/GUIDE.md:17`
currently tell the model it holds no token and cannot start missions; after
removal these become lies that will actively mislead the window.

## 2. Work items

Ordered by dependency. Paths are repo-relative under `~/projects`.

### W1. agautolab — remove gateway auth, unify entrance into window

- Delete in `pj-agdev/agautolab/agent/gateway.py`: `TOKEN_FILE` (:73),
  `read_token()` (:98-102), `authorized()` (:838-847), the call at :875-876,
  and the boot refusal at :1124-1125.
- Unify the entrance: make mission-starting reachable from `POST /window`
  (the window decides to start a mission, or the window handler accepts a
  mission intent), and retire `POST /mission` as a separate public door — or
  minimally keep it as an open internal route the window uses. Design choice
  D1 below.
- Rewrite the module docstring (:21-44) and `WINDOW_PROMPT` (:582-583) so the
  window is told it *can* start work.
- Keep the non-auth guards that exist for cost/concurrency, with reworded
  comments: `window_lock` (one answer at a time), summarize
  one-at-a-time + per-iteration cache, evidence path containment. These are
  irreversible-harm/cost guards, not auth.
- No tests break (no test covers auth); add nothing — the removal is the test.

### W2. Ansible — stop provisioning the gateway token

In `pj-clusterintent/ansible_agdev/roles/autolab_node/`:
- Delete token generation/slurp/fetch tasks (`tasks/main.yml:50-77`) and the
  now-dead `autolab_node_token_fetch_dir` default (`defaults/main.yml:19-21`).
- **Keep** the cagent human-token copy (`tasks/main.yml:36-42`) — that is
  agcluster auth, retained by the braindump.
- Delete stale controller-side copies under `~/.local/state/autolab-gateway/`
  and node-side `.local/agent/gateway_token` files (cleanup, optional).

### W3. Redeploy agautolab1

The node runs a pre-open-read checkout. After W1 lands:
- push to agstudio gitea, then run
  `ansible-playbook -i inventories/agautolab.yml playbooks/agent/setup_autolab_node.yml`
  from `pj-clusterintent/ansible_agdev` (the only controller channel).
- Separate known issue to carry along: `agautolab1.local` resolves to
  192.168.0.220 while Nautobot desires 192.168.0.130 — reconcile or record.

### W4. agdevworld — drop the auth-compensation guards

In `pj-agdev/agdevworld/assistant/server.mjs`:
- Delete the method gate (`isOpenPost` + 405, :525-540) so the passthrough
  can reach mission-starting on nodes.
- Re-decide the remaining guards as *policy* rather than auth compensation:
  the finite `AUTOLAB_NODES` allowlist (:345-349) is worth keeping (it stops
  the assistant being an open LAN relay — a reach guard, not auth); the
  `/evidence/` 403 (:519-524) likewise stays or goes on its own merits.
- Update prose that encodes the old model: `ROLE_PROMPT` (:104-120), header
  comment (:14-26), `assistant/GUIDE.md:48-52` (re-read per request, no
  restart needed), `README_DEV.md:49-58` "Safety devices",
  `src/autolabState.ts:91-93` comment.

### W5. agforge — nothing to remove; record the decision on presigned URLs

Endpoints are already unauthenticated. The MinIO credential and presigned
URLs are agforge's *delivery mechanism* (SigV4 signs the reachable hostname;
the URL is the deliverable), not an agent gate — recommend keeping them
(decision D2). The agent tool allowlists (`agent_run.py:64-110`,
`opencode.json`) are authorization/blast-radius guards, not authentication;
keep the destroy-class denies per the safety-device audit rule
(irreversible-harm guards stay).

### W6. agcluster — keep auth; state the read-only convention for agdevworld

- No code change in cagent now. Today agdevworld's only cagent call is the
  read-side snapshot fetch (`scripts/fetch-cluster-state.mjs`), so the
  braindump's "read系だけ" is currently satisfied *by convention*.
- Record the convention explicitly (this episode + `agdevworld/README_DEV.md`):
  agdevworld may only issue read-shaped requests to cagent; write-shaped
  prompts (`desired apply`, `reconcile --yes`) are out of bounds.
- Known limitation to record, not fix now: the human token cannot enforce
  read-only — a scoped/second token or a GET-only listener would be new work
  in cagent (candidates noted in the investigation; also the pre-`c6adb2a`
  deny list `*nctl*reconcile*--yes*` etc. as a process-wide fallback).
  Defer to the future unified-auth (JWT) episode.
- "部分的に無認証にする必要があればそうする": no current need — `cluster:fetch`
  works with the token, and cagent has no no-auth mode; adding one would be
  new work against a component we are keeping intact. Skip.

### W7. Delete / rewrite contradictory development rules

Normative documents that mandate the old auth model (delete or rewrite):

| File | What contradicts |
|---|---|
| `pj-agdev/agautolab/agent/CHARTER.md:53` | "`POST /mission` is the only authenticated route" listed as a non-negotiable safety device — delete the item |
| `pj-agdev/agautolab/agent/GUIDE.md:17-18` | Doors list: `/mission` + bearer as a separate door — rewrite for single window entrance |
| `pj-agdev/agautolab/agent/README.md:40,43-44,56-57,62-64` | token boot requirement, "only authenticated route", "auth designed system-wide later" |
| `pj-agdev/agautolab/README.md:61` | prose encoding the mission/window split |
| `pj-agdev/agdevworld/README_DEV.md:55-56` | 405 write-block justified by the node's token gate |
| `pj-agdev/agdevworld/assistant/GUIDE.md:48-52` | "writes refused because this passthrough carries no identity" |

Keep as-is:
- All historical episode records mentioning tokens
  (`devdocs/episodes/unshackle_agent/**`, `agent_mindmap/**`,
  `pj-agdev/devdocs/episodes/**`) — dated records, not live rules.
- All cagent/agcluster auth docs (`pj-clusterintent/cagent/README.md`,
  `devdocs/vision/cluster_agent/p4/contract.md`, etc.).
- `devpolicy/terms.md` — "auth machinery" in the Entrance definition still
  holds for agcluster; `# Single Entrance` is the policy that *justifies*
  the window unification, cite it in the report.
- Gitea token rules (`CHARTER.md:25-27`, `AGENT_GUIDE.md:88-105`) — deploy
  credential, not an agent gate.

Consolidate the scattered "auth is designed system-wide later" /
"introduce identity and this can go" notes into this episode as the single
pointer to the future JWT vision (braindump line 6), instead of leaving them
as implied obligations in component docs.

### W8. Verification

- `curl -X POST :8791/mission` with no header → 202 (both agstudio and
  agautolab1 after W3); gateway boots with no `gateway_token` file.
- Window can start work end-to-end through the agdevworld assistant
  (browser → :8091 passthrough → node window → mission running).
- `npm run cluster:fetch` still succeeds with the human token (agcluster
  path untouched).
- cagent mTLS conformance gate untouched — no cagent code change, so no run
  required; if anything in cagent is touched after all,
  `uv run --project cagent pytest -q devtests/test_strategy/test_mtls_conformance.py`.

## 3. Decisions needed (D)

- **D1 — shape of the unified entrance.** (a) `/window` gains the power to
  start missions itself and `/mission` is deleted; or (b) `/mission` stays as
  an open route and the window is simply told it may call it. (a) is the
  literal braindump reading ("windowに統一") and honors Single Entrance;
  (b) is the smaller diff and keeps the deterministic mission API for
  scripts/monitor. Recommendation: (b) first — remove auth and update the
  prompts so the window can drive `/mission` — then fold the route into the
  window only if the two-route shape causes real friction. Both satisfy
  "認証を無くす" immediately.
- **D2 — agforge MinIO/presigned URLs.** Keep (recommended): they are the
  delivery mechanism and storage credential, not an agent gate; making the
  bucket anonymous changes the product surface (`generate.py:127-157`,
  TTL semantics in `service/GUIDE.md:25`, `charter.md:18-19`) for no
  braindump requirement. Only revisit if "無認証" is meant to include storage.
- **D3 — agdevworld node allowlist (`AUTOLAB_NODES`).** Keep (recommended):
  a reach guard against being an open LAN relay, i.e. an irreversible-harm
  style guard, not authentication. Delete only if the braindump's "他の
  エンドポイントも認証なし" is meant to cover reachability policy too.

## 4. Risk notes

- After removal, two *paid* POSTs are open on every autolab node
  (`/window` on the claude backend, `/summarize/`): protected only by
  one-at-a-time locks and the summarize cache. Acceptable on the isolated
  single-user cluster premise; the locks stay.
- agforge `POST /api/requests` was already an open paid endpoint — the
  braindump's premise (isolated cluster) is what makes both acceptable.
- The agent subprocess in agforge inherits the full process env including
  S3 keys (`agent_run.py:262-270`); unchanged by this episode, noted for the
  future auth episode.
