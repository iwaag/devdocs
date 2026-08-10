# zero_auth — report, Step 2 (stop provisioning the token)

AI-generated (Omni Agent). Backend: Claude Code / claude-fable-5.
Date: 2026-08-10.

## What was done

In `pj-clusterintent/ansible_agdev/roles/autolab_node/`:

- Deleted the four token tasks (generate / slurp / controller dir /
  controller-side copy, formerly `tasks/main.yml:50-77`) and replaced them
  with one cleanup task: `Retire the gateway bearer token (zero_auth
  episode)` — `file: state=absent` on `.local/agent/gateway_token`, so the
  next playbook run deletes the stale token from every node.
- Deleted the now-dead `autolab_node_token_fetch_dir` default
  (`defaults/main.yml`).
- **Kept** the cagent human-token install task untouched — that is
  agcluster auth, retained by decision.

Local cleanup:

- agstudio node-side `.local/agent/gateway_token` (moved aside in Step 1)
  is now deleted.
- Controller-side `~/.local/state/autolab-gateway/` could **not** be
  deleted: the harness permission classifier denied `rm -rf` outside the
  workspace. Left in place — stale, harmless (nothing reads it anymore);
  the user may remove it by hand: `rm -rf ~/.local/state/autolab-gateway`.

## Verification

- `grep gateway_token|token_fetch` across `ansible_agdev`: only the new
  state-absent cleanup task remains.
- `ansible-playbook -i inventories/agautolab.yml
  playbooks/agent/setup_autolab_node.yml --syntax-check` — passes.
- Live run against agautolab1 happens in Step 3.
