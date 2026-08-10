# zero_auth — report, Step 3 (redeploy agautolab1)

AI-generated (Omni Agent). Backend: Claude Code / claude-fable-5.
Date: 2026-08-10.

## What was done

- Pushed agautolab `main` (Step 1 commit `3357498`) to the agstudio gitea
  deploy source (`8c0c9f7..3357498`).
- Ran `ansible-playbook -i inventories/agautolab.yml
  playbooks/agent/setup_autolab_node.yml` from
  `pj-clusterintent/ansible_agdev` (the only controller channel,
  `~/.ssh/ansible_key`). Recap: `ok=13 changed=3 failed=0` — checkout
  updated, the new `Retire the gateway bearer token` task deleted the
  node-side token file (`changed`), gateway restarted via handler and
  passed its health probe.

## Verification (from agstudio)

- `GET http://agautolab1.local:8791/healthz` → `{"ok": true}`.
- `GET /jobs` → 200 with the jobs document — previously this demanded a
  bearer token on the stale checkout; now open.
- `POST /mission` with no Authorization header and an empty body →
  `400 body must be {"mission": "..."}` — the request reaches the body
  parser (not 401), so the mission route is open. Invalid body used on
  purpose so no real drive starts on the remote node.

## Notes

- Known separate issue, carried along as instructed, not blocking:
  `agautolab1.local` resolves to **192.168.0.220** while Nautobot desires
  **192.168.0.130**. Not touched in this episode.
- Ansible warning (pre-existing, benign): discovered Python interpreter
  `/usr/bin/python3.12` on the node.
