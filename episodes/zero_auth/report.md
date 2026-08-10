# zero_auth — final report

AI-generated (Omni Agent), 2026-08-10. Per `devpolicy/agent_records.md`:
this episode's omni-agent run was served by **Claude Code /
claude-fable-5**; in-system runs it triggered are recorded in their own
workspaces (window run-0054/0055: `claude/claude-sonnet-5`, $0.115/$0.119;
mediator session-0029: `claude-sonnet-5`, $0.60; smoke-e2e job: `fake`
adapter, $0). Outcome: **done**. Step-by-step details in
`report1.md`–`report8.md`.

## What the episode did

pj-agdev is now uniformly auth-free, and agautolab has one entrance.

1. **agautolab gateway** — bearer-token auth and the no-token boot refusal
   deleted; cost/concurrency guards kept and re-justified (window lock,
   summarize one-at-a-time + cache, evidence path containment).
2. **ansible** — the `autolab_node` role no longer generates/fetches a
   gateway token; a state-absent task retires it from nodes. cagent
   human-token provisioning untouched.
3. **agautolab1** — redeployed twice (post-Step-1, final); serves the new
   gateway.
4. **agdevworld** — the 405 method gate (auth compensation) deleted; the
   `AUTOLAB_NODES` allowlist (D3) and the `/evidence/` 403 kept as reach /
   data-locality devices.
5. **window** — gained mission-starting power (Tool Giving): a
   `<<mission>>…<</mission>>` block in its reply is executed by the
   gateway through the refactored `start_mission()` seam; 409-while-alive,
   done-clearing, run-id+pid semantics kept.
6. **`POST /mission` abolished** — route deleted, every live reference
   retired; `start_mission()` remains as the internal function should a
   deterministic trigger ever be needed. **Exit condition verified: 404 on
   both nodes.**
7. **Rules** — contradictory auth passages deleted across
   CHARTER/GUIDE/READMEs; the cagent read-only convention recorded in
   `agdevworld/README_DEV.md`; the scattered "auth designed system-wide
   later" notes consolidated into this episode as the single pointer to
   the future JWT vision (see `report7.md`).
8. **End-to-end proof** — a conversational request through the assistant
   (`:8091`) to the agstudio window started a real mission (run 13) that
   converged and declared done, with no other door. `cluster:fetch`
   through cagent still works (agcluster auth untouched, D2/D3 kept).

## Policy

This unification fulfills **# Single Entrance** (`devpolicy/terms.md`):
the window is the one conversational entrance where desire is expressed;
monitoring reads and the deploy credential are not entrances. The old
mission/window split existed only as a property of the token; with the
token gone, the entrance and the ability to start work are one door.

## Known items carried forward

- `agautolab1.local` resolves to 192.168.0.220; Nautobot desires .130.
- Controller-side stale dir `~/.local/state/autolab-gateway/` left on
  agstudio (harness denied deletion outside the workspace); safe to
  `rm -rf` by hand.
- For the future JWT episode: read/write distinction in cagent's API;
  agforge subprocess env inherits S3 keys.
