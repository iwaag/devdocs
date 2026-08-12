# Plan: Group chat (Zulip) launch and participation check

## Goal

Stand up a self-hosted Zulip on agstudio as the future hub of agent communication,
give every in-system agent (and the humans/Omni Agent) an account, and verify that
each agent's runtime environment can send and receive messages. Replacing the
string-telephone routes and wiring chat into real workflows is a **separate episode**.

## Non-goals

- No workflow integration (no mission dispatch, no report routing through chat yet).
- Plane stays exactly as it is (the failed write-back path is deliberately parked).
- No removal of existing REST routes (`AUTOLAB_NODES`, passthroughs, `/window`).
- No hardening beyond what Zulip gives by default. This is a scratch/experimental
  environment: self-signed TLS, LAN-only exposure, and pragmatic shortcuts are fine.

## Constraints (keep these, everything else is implementer's discretion)

1. No secrets or local host/IP details in git-tracked files (repo rule). Credentials
   go under `.local/`, mode 0600, following the `plane-credentials.env` pattern.
2. Don't stop or reconfigure the running Plane stack and other live services.
3. Record which backend/model served any agentic run, per Agent ≠ Model policy
   (only relevant if you use an agent to do the work).

## Useful context discovered during planning

- **Precedent to copy**: Plane CE self-host lives at `pj-agdev/.local/plane-selfhost`,
  managed by its own compose stack, LAN URL `http://agstudio.local:8290`. Mirror this
  shape: `pj-agdev/.local/zulip-selfhost/`.
- **Ports already taken on agstudio**: 3000 (gitea), 5173, 8000 (nautobot), 8090,
  8091, 8092, 8290, 8490, 8789, 8791, 9100 (MinIO), 5432, 6379. Pick free ones
  (e.g. 8380 HTTP / 8543 HTTPS) and note them in `.local/devenv.md`.
- **docker-zulip** (github.com/zulip/docker-zulip) is the supported deployment. Key
  settings: `EXTERNAL_HOST` (use `agstudio.local:<port>`), `SSL_CERTIFICATE_GENERATION=self-signed`,
  disable outgoing email (`DISABLE_EMAIL_NOTIFICATIONS`/noop SMTP is acceptable here),
  and turn off mobile push notification registration (external telemetry — same
  policy as Plane's disabled telemetry). It bundles its own Postgres/Redis/RabbitMQ/
  memcached; keep them internal to its compose network, do NOT reuse the host's
  `my_postgres_db`/`service_scripts-redis-1`.
- **Memory**: Zulip wants ~2 GB. agstudio already carries Plane + nautobot + services;
  check headroom first (`docker stats --no-stream`, Activity Monitor). If it doesn't
  fit, fall back to a Proxmox guest via the pj-clusterintent add-LXC recipe — but try
  agstudio co-location first.
- **This shell exports `DEBUG=release`**, which broke Plane's setup script. Run any
  third-party setup with `env -u DEBUG ...` if it misbehaves.
- **Zulip API in one breath** (works with curl / Python stdlib / Node fetch — no SDK):
  - Auth: HTTP Basic, `bot_email:api_key`.
  - Send: `POST /api/v1/messages` with `type=stream&to=<stream>&topic=<topic>&content=<text>`.
  - Read (poll): `GET /api/v1/messages?anchor=newest&num_before=20&num_after=0&narrow=[{"operator":"stream","operand":"sandbox"}]`.
  - Read (push-ish): `POST /api/v1/register` → queue_id, then long-poll `GET /api/v1/events`.
  - Every message carries `sender_email`/`sender_full_name` — the sender-identity
    problem from the braindump is solved by the platform; nothing to build.
- **Reachability quirks**: the assistant container reaches the host via
  `host.docker.internal`; `agautolab1` reaches agstudio as `agstudio.local` (its
  gitea/MinIO already resolve it). A bare `node` process on agstudio may get
  `EHOSTUNREACH` to LAN addresses (macOS local-network privacy) — test from inside
  containers where relevant, and use `curl -k` for the self-signed cert.
- **cagent runs on agstudio** (pj-clusterintent), so its send/receive check can be
  done from its local environment; no cluster-side deployment needed this episode.
- **Future linkage hint** (don't implement now): reserve the convention
  "topic name = Plane issue ID / mission ID" so the two-channel model
  (chat = origin, Plane = durable task state) drops in later without renaming.

## Steps

### Step 1 — Preflight and placement decision
Check agstudio memory/disk headroom and confirm chosen host ports are free.
Decide agstudio vs Proxmox-guest placement (default: agstudio).
**Done when**: placement and ports are written down in `pj-agdev/.local/devenv.md`.
Produce `report1.md`.

### Step 2 — Deploy Zulip
Set up docker-zulip under `pj-agdev/.local/zulip-selfhost/`. Self-signed TLS,
email/push disabled, data on named volumes. Verify the web UI loads from a browser
at `https://agstudio.local:<port>` and survives a `docker compose restart`.
**Done when**: login page reachable from the LAN; restart-safe. Produce `report2.md`.

### Step 3 — Organization, accounts, streams
Create the realm and accounts:
- Humans: one admin (developer), optionally a read-mostly viewer.
- Omni Agent: its own bot (or full account — implementer's choice, note the reasoning).
- Bots: `devworld-assistant`, `autolab-agstudio`, `autolab-agautolab1`, `forge`, `cagent`.
Streams: `#ops` (everyone), `#sandbox` (for this episode's checks). Keep it minimal;
more streams are a later-episode concern.
Collect each bot's email + API key into per-agent env files under the owning
project's `.local/` (e.g. `pj-agdev/.local/zulip/<bot>.env`, cagent's under
`pj-clusterintent/.local/`).
**Done when**: all accounts exist, credentials stored 0600, a manual browser post
in `#sandbox` works. Produce `report3.md`.

### Step 4 — Credential placement per runtime
Put each bot's credentials where its runtime can read them:
- assistant container: mount or env-inject (compose already wires `.env` values).
- agforge service, autolab gateway (agstudio), cagent: local `.local/` file each.
- agautolab1: reuse the Ansible mechanism that ships `plane.env`
  (`AUTOLAB_NODE_PLANE_CREDENTIALS_SOURCE` pattern) or a one-off manual copy —
  implementer's choice; the Ansible route pays off later, the manual route is faster.
**Done when**: each runtime environment has readable credentials for its own bot.
Produce `report4.md`.

### Step 5 — Participation check (send and receive from every runtime)
From each agent's actual runtime environment (inside the assistant container, on
agautolab1 via its shell, etc.), using plain curl or a throwaway script:
1. Send: post "hello from <bot>" to `#sandbox`, topic `participation-check`.
2. Receive: fetch recent `#sandbox` messages and confirm the other bots' hellos,
   with correct sender identity visible.
No integration into agent code — this validates credentials, network paths, and TLS
from each vantage point only.
**Done when**: all six identities have posted and read back messages; a screenshot
or message-list capture of `#sandbox` shows every sender. Produce `report5.md`.

### Step 6 — Human monitoring check and wrap-up
Confirm the web UI gives the "casual monitoring" experience: `#sandbox` history,
sender names, topic list, from a LAN browser. Update `pj-agdev/.local/devenv.md`
(ports, start/stop commands, credential locations) and write the episode's final
`report.md`, including anything that should seed the next episode (workflow
integration, string-telephone removal).
**Done when**: report.md exists and devenv.md reflects reality.

## Notes for the implementer

- You have wide latitude on tooling and order within a step; the step boundaries
  exist so each produces a verifiable report, not to constrain method.
- If docker-zulip fights you on macOS (it is Linux-first), acceptable fallbacks in
  order: tune compose settings → run it in a Linux VM/guest → escalate to the human
  with the failure evidence. Don't burn more than a couple of hours silently;
  failure reports are assets (Failure Farming).
- `zulip-send` and the `zulip` Python package exist and are fine to use for checks,
  but stdlib/curl is enough and matches the agents' dependency style.
