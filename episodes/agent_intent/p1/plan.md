# agent_intent — Phase 1 plan: agents as desired/actual state

AI-generated (Omni Agent, 2026-08-17). Plan for `../braindump.md`.
Destructive phase — no backward compatibility required. Keep implementer
discretion high; the rules below are the minimum, everything else is advice.

## Goal

An agag agent (repo + workspace + running listener + Zulip/Plane identity)
becomes a first-class desired/actual pair in cluster intent:

- **Registered** (deterministic, drift gap): Zulip account + subscriptions
  exist, Plane account + membership exist, per-agent credentials placed.
- **Alive** (informational, NOT a gap in p1): listener polled Zulip events
  recently — freshness of a node-local status file.

Roll-call, reconcile actions for liveness, and Zulip presence are all out of
scope for p1 (presence is an optional bonus, see step 6).

## Target shape

```
nintent/nautobot_intent_catalog/models.py   + DesiredAgent (thin identity)
nctl/src/nctl_core/drift/agent_evaluation.py  registered gaps + freshness class
nctl (collector)                            Zulip/Plane API read → ingest
nauto/jobs/                                 + agag registration ingest Job
pyagag (github: iwaag/pyagag)               listener writes agag-status.json
.local/plane/<agent>.env                    per-agent Plane API keys (0600)
```

## Steps

1. **DesiredAgent model** (nintent). Deliberately thin, DesiredService
   style: `name`, `slug`, `lifecycle` (reuse the existing enum), optional FK
   `desired_workspace`, optional FK `desired_service_placement`,
   `zulip_user_id` (int), `plane_user_id` (string, blank until step 2),
   `desired_zulip_channels` (JSON list of channel names). Identity keys are
   numeric/opaque ids, never emails — the Zulip realm hides emails from
   events (agforge precedent: keys on user ids; known ids today: developer 8,
   devworld assistant 10, agforge 13, cagent 14).
   - Deploy path reminder: the Nautobot container installs nintent from
     GitHub (`pip install git+…/iwaag/nprojects.git`), so the loop is
     commit → ask the developer to push → `docker compose build` → restart →
     `nautobot-server migrate` in the container. Scratch DB; migrate freely,
     `--keepdb` for test iteration.
   - Precedent worth reading before modeling: the retired `node_agent`
     profile and `PROFILE_BINDING_NAMES` (models.py ~line 647–659). If you
     want to bind an agent to its LLM provider, revive that binding
     mechanism rather than inventing a new edge — but that is optional in p1.

2. **Per-agent Plane accounts.** Today all agents share one `PLANE_API_KEY`
   (see `pj-agdev/.local/plane-credentials.env`; `agag.plane` loads it).
   Create one Plane CE member account per agent (cagent, agforge, autolab,
   devworld assistant), each with its own personal API token.
   - Email delivery does not exist in this environment; the admin/viewer
     accounts were created by pulling codes from logs/console. Do the first
     agent account manually, write the ritual down in this episode dir; if
     it stays manual, that is acceptable — observe/drift remain automatic.
   - Store per-agent tokens as `pj-agdev/.local/plane/<agent>.env`, one file
     per agent, mode 0600, mirroring `.local/zulip/<bot>.env`.
   - `agag.plane` reads a single `PLANE_API_KEY`, so no code change is
     needed for attribution: distribute a different env file per agent.
     For autolab nodes the distribution hook already exists —
     `AUTOLAB_NODE_PLANE_CREDENTIALS_SOURCE` on
     `playbooks/agent/setup_autolab_node.yml`; point it at the agent's file.
   - Record each account's Plane user id into its DesiredAgent.

3. **Registration collector + ingest.** A small collector (nctl side, next
   to `observation.py`'s pattern: collect → validate → ingest via a nauto
   Job) that reads, with admin credentials:
   - Zulip: `GET /users` (+ per-bot channel subscriptions) from
     `https://agstudio.local:8543` — self-signed TLS, and beware the
     `agstudio.local` multi-address DNS trap if anything runs inside Docker
     (`ZULIP_LAN_HOST` note in pj-agdev devenv.md).
   - Plane: workspace members list from `http://agstudio.local:8290`
     (admin key in `.local/plane-credentials.env`).
   Ingest as actual-state records keyed by DesiredAgent slug. Shape of the
   actual model/table is implementer's choice — a minimal "AgentRegistration
   observation" is enough; don't over-model.

4. **Liveness status file.** In pyagag (the shared listener code): after
   every *successful* event-poll cycle, atomically write
   `<workspace>/.local/agag-status.json` with `last_poll_ok` (server
   timestamp), `queue_id`, `last_error` (nullable). Write it only on
   success — that is the whole anti-lying rule. Then extend
   `nodeutils_collect.py` (it already walks workspaces for git status) to
   pick the file up when present, and let the existing ingest carry it.
   Node clocks vs Nautobot clock: compare against the collection timestamp,
   not local wall-clock claims, to dodge skew.

5. **Drift evaluation.** New `agent_evaluation.py` beside
   `workspace_evaluation.py`, same philosophy:
   - Gaps (real drift): DesiredAgent has no matching Zulip user / missing
     channel subscription / no Plane membership / per-agent env file absent
     from the placement.
   - Informational `liveness_class` (NOT a gap): `polling` (status fresh
     within 3× the poll interval), `stale`, `unobserved` (no status file).
     This mirrors `activity_class` exactly; resist promoting staleness to a
     gap until false-positive data exists.
   - Surface both in `nctl drift` output; a `nctl agents` convenience view
     is optional.

6. **Optional: Zulip presence.** One line in the listener — POST
   `/users/me/presence` after a successful poll — gives a free realtime
   complement readable via the Zulip API. Nice, not required; the status
   file is the source of record because it also works when Zulip is down.

## Enrollment order

Start with **agforge** (simplest listener, known-paid DM behavior is
irrelevant here) and **autolab on agautolab1** (exercises the ansible
distribution path), then cagent and the devworld assistant. Note while
enrolling: `agautolab1.local` has a known stale-checkout/IP quirk (resolves
.220, Nautobot desires .130) — enrolling it will surface real drift, which
is the point, not a blocker.

## Rules (minimum)

- Secrets stay in `.local/`, 0600, never committed; don't print token values.
- Deploy sources stay on GitHub — never repoint anything at the old gitea.
- Everything else — schema details, collector transport, file formats — is
  implementer's discretion.

## Acceptance to demonstrate live

- `nctl drift` shows an enrolled agent as registered-converged, with a
  `liveness_class` of `polling`.
- Stop one listener (`launchctl` / kill): registration stays converged,
  `liveness_class` degrades to `stale` after the threshold — and no gap
  appears.
- Delete a test agent's Zulip channel subscription (or use a deliberately
  wrong desired channel): a registration gap appears.
- A Plane issue created by an enrolled agent shows that agent's own account
  as actor (per-agent attribution works end to end).
