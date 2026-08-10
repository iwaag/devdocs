# zero_auth — report, Step 8 (verify and report)

AI-generated (Omni Agent). Backend: Claude Code / claude-fable-5.
Date: 2026-08-10.

## Final deploy

- agautolab pushed to the agstudio gitea (`3357498..9989615`) and
  agautolab1 redeployed via the playbook (`ok=13 changed=2 failed=0`).
- Assistant container rebuilt (`docker compose up --build -d assistant`)
  so the Step-6 ROLE_PROMPT/GUIDE state is what serves.

## Checklist results

- **Gateway boots with no token file on both nodes** — agstudio: verified
  directly (no `gateway_token` under `.local/agent/`, gateway serving);
  agautolab1: the role's cleanup task deleted the token (`changed` in the
  Step-3 run), gateway restarted under systemd, `/healthz` ok. Grep over
  the deploy path (`ansible_agdev`): no `gateway_token` residue beyond
  the state-absent cleanup task itself.
- **End-to-end unified entrance** — through the assistant service
  (`:8091`, the same container endpoints the browser chat drives):
  `POST /api/autolab/agstudio/window` with a conversational request for a
  zero-cost smoke test → the window (claude/claude-sonnet-5, run-0055,
  $0.119) answered with a `<<mission>>` block → gateway executed it:
  **202, run 13, pid 38388** → drive ran mediator session-0029
  (claude-sonnet-5, 22 turns, $0.60, 91 s) → job `smoke-e2e` on the
  `fake` adapter **converged on iteration 1** → agent wrote `done`,
  drive stopped, exit 0. A conversational request became a running,
  converging mission with no other door involved.
- **`POST /mission` → 404 on both nodes** — agstudio and agautolab1 both
  answer `{"error": "unknown route"}` with a valid mission body. **Exit
  condition met.**
- **`npm run cluster:fetch` still works** — all three snapshots
  (state/workspaces/actual) fetched through cagent `:8789` with the human
  token; agcluster untouched (and `GET`, not HEAD, used for probing).

## Notes

- Verified through the container endpoints with curl; no manual browser
  click was performed — the browser drives exactly these endpoints.
- agautolab1 IP mismatch (`.local` resolves 192.168.0.220, Nautobot
  desires .130) — unchanged, carried in report3.
- Deus Ex Machina notes: restarted the agstudio gateway, rebuilt the
  assistant container, and ran the deploy push+playbook for the autolab
  nodes — handoff candidates. The mission itself was started and carried
  out by the in-system agents (window + mediator), which is the point.
