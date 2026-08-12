# Phase 2 Plan — A lightweight report path from in-system agents to cagent

Source: [braindump.md](../braindump.md) phase 2. Policy under strengthening
(devdocs/README_DEV.md): *"In-System workflow should be designed so that cagent
receives a report when cagent's explanation of the cluster is found invalid."*

Goal: every in-system agent that discovers a supposedly-running service
unreachable has a way to tell cagent, is told (briefly) to use it, and cagent
verifies + records what arrives. Deliberately lightweight: no mandatory
pre-flight cluster checks, no report schema, no new report API.

Ground rules (deliberately few):

- Experimental, non-production environment. No backward compatibility owed;
  restarting services and editing charters freely is fine.
- No secrets/tokens in tracked files (existing token locations:
  `~/.local/state/cagent/human_token`, `pj-clusterintent/.local/secrets`).
- Reporting must stay optional-shaped for agents: a clause in their
  instructions, not code that auto-fires reports or blocks work until a report
  succeeds (Tool Giving, not Tool Implantation).
- Everything else — wording, exact transport, code shape — is implementer's
  discretion. Prefer acting and reporting over asking.

Write `report1.md` … `report4.md` (one per step) in this folder.

## Design decisions already made (with the user, 2026-08-12)

- **Channel = the existing conversational entrances, free text.** A report is
  just a message like "report: agforge at :8092 should be running but is
  unreachable" sent to `POST /requests`. No new endpoint, no format. Single
  Entrance holds; every request is already persisted by cagent's
  `EvidenceWriter` for free.
- **cagent records and verifies; it does not remediate (yet).** Phase 1 gave
  keepers auto-restart, so immediate repair adds little. Accumulate evidence
  first; escalate to action later only if the evidence says so.

## Baseline findings (2026-08-12 survey)

- cagent API routes live in `pj-clusterintent/cagent/src/cagent_api/server.py`
  (route regexes ~line 16, dispatch ~112). Node entrance :8788 (mTLS), human
  entrance :8789 (bearer token). Only inbound content channel:
  `POST /requests {"message": …}`. Capability card
  `src/cagent_api/static/llms.txt` is re-read per request — editable without
  restart. Agent behavior lives in `cagent/opencode/AGENTS.md` — needs an
  OpenCode restart (`launchctl kickstart` the `com.clusterintent.cagent-opencode`
  agent) to take effect.
- Existing cagent clients: node agents get a `cagent ask/status` CLI + an
  instruction section injected into their AGENTS.md by the ansible role
  `cagent_client` (`ansible_agdev/roles/cagent_client/`, section text in
  `files/cagent_agents_section.md`); autolab nodes get `autolab-cagent`
  (`roles/autolab_node/templates/autolab-cagent.py.j2`, human entrance).
- **agautolab**: already has `autolab-cagent ask` and cites it in
  `agautolab/agent/CHARTER.md` (~line 45). Failure records already exist
  (`.local/agent/NOTES.md`, window run records). Only a clause is missing.
- **agdevworld assistant**: has NO cagent route. Its passthroughs already
  detect exactly our trigger: `assistant/server.mjs` returns
  `forge_offline` (~line 227) and `node_offline` (~line 301). Instructions =
  role prompt literal (~line 55) + `assistant/GUIDE.md` (re-read per chat
  request). It also has `POST /api/note` (assistant.note.v1, ~line 345) for
  its own local record.
- **agforge**: has NO cagent client. Runs natively on agstudio, so the
  node-installed `cagent` CLI (or a curl to :8789 with the human token) is
  likely already reachable from the request agent — verify rather than build.
  Instructions = `service/charter.md` (re-read per request, templated) +
  `service/GUIDE.md`. Failure inbox `.local/problems/` already exists.
- nctl already models this symptom: binding state `unreachable` =
  "endpoints match, probe failed" (`nctl_core/drift/binding_evaluation.py`),
  surfaced by `nctl status` / `nctl drift` — cagent's verification tool.

## Step 1 — Close the tooling gaps (agdevworld, agforge)

- agdevworld assistant: add a minimal cagent passthrough in
  `assistant/server.mjs`, same shape as the existing forge/autolab
  passthroughs — submit message, poll, return answer text. Human entrance
  `https://host.docker.internal:8789` from the container; token via env from
  `~/.local/state/cagent/human_token` (compose env wiring, token stays out of
  Git). Self-signed TLS: the fetch script precedent
  (`scripts/fetch-cluster-state.mjs`) already accepts it.
- agforge: from an actual request-agent run (or its shell context), test
  `cagent ask 'ping'`. If it works, Step 2 wording is all agforge needs; if
  not, give it the smallest working transport (curl + token file) documented
  in its GUIDE.
- Quick check both: one round-trip question each, answer text comes back.

Deliverable: `report1.md` — what was added vs already worked, round-trip proof.

## Step 2 — One reporting clause per agent

Add a short clause (1–3 lines, same spirit everywhere, wording free) to:

- `agautolab/agent/CHARTER.md` and/or `agent/GUIDE.md`
- `agdevworld/assistant/GUIDE.md` (+ role prompt route list if a passthrough
  route was added in Step 1)
- `agforge/service/charter.md` (the Tools section next to the problems-dir
  bullet is a natural home) and `service/GUIDE.md`
- `ansible_agdev/roles/cagent_client/files/cagent_agents_section.md` for node
  agents — then redeploy the touched nodes (playbook run, see README_DEV
  ansible commands) so the section lands.

Gist to convey: *"If a service that is supposed to be running is unreachable,
send cagent a one-line report saying what you tried and what failed, then
continue or fail as you normally would. Don't pre-check the cluster; report
only what you actually ran into. If the unreachable thing is cagent itself,
record it in your local failure notes instead."* Include usage info (which
command/route, one example) — a tool without usage info is an Unexplained
Chainsaw.

Deliverable: `report2.md` — diff summary of every touched instruction file.

## Step 3 — cagent's receiving behavior: verify + record only

- `cagent/opencode/AGENTS.md`: on receiving an unreachable-service report,
  (1) cross-check with `nctl status` / `nctl drift` — was cagent's picture of
  the cluster actually wrong, or is the reporter's network/config the problem?
  (2) append one line to a running log of received reports, with verdict
  (confirmed / not-reproduced / reporter-side), (3) answer the reporter with
  the verdict. Explicitly: do NOT restart or reconcile services in response —
  record only. Restart OpenCode afterward.
- Pick the accumulation surface — implementer's choice, smallest thing that
  can be listed later wins. Candidates: a single append-only file under
  cagent's state dir (e.g. `~/.local/state/cagent/reports.md`), or `nctl ops`
  records. Session evidence already persists automatically; this log is only
  the cross-request index of it.
- `static/llms.txt`: add a line that reports of unreachable services are
  welcome and what happens to them (no restart needed to take effect).

Deliverable: `report3.md` — the AGENTS.md/llms.txt additions and the chosen
log surface.

## Step 4 — Fault-injection check, then report

Break something real, watch the whole path, restore:

1. Stop agforge (`launchctl bootout` the `com.agdev.agforge` LaunchAgent —
   KeepAlive would undo a plain kill). Ask the agdevworld assistant for an
   asset. Expect: `forge_offline` surfaces, assistant reports to cagent,
   cagent verifies (drift should show the placement unreachable while it's
   down) and logs a confirmed verdict.
2. One autolab-side case: e.g. stop Plane (`env -u DEBUG ./setup.sh stop` in
   its selfhost dir) during a small autolab job, or any dependency the
   implementer prefers. Expect a report via `autolab-cagent` and a log entry.
3. Negative path sanity: a report about a service that is actually fine
   should come back "not-reproduced", not get blindly logged as an outage.
4. Restore everything; `nctl status` back to the phase-1 baseline
   (all keepers converged).

Deliverable: `report4.md` — transcript excerpts of each path, cagent's log
lines, final drift summary.

## Close

`report.md`: what the path looks like end-to-end now, what reports were
collected during testing, and the open question intentionally left for later —
when accumulated reports justify letting cagent act instead of only record.
