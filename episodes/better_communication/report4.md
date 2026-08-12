# better_communication — Step 4 report: runtime credential placement

Date: 2026-08-12. Status: **complete**. Every planned runtime can read its own
bot credential without putting secrets or local host details in Git.

## Placement

| Identity | Runtime credential location and delivery |
|---|---|
| Omni Agent | ignored `pj-agdev/.local/zulip/omni-agent.env`, mode 0600 |
| Devworld Assistant | its ignored env file is mounted read-only at `/run/secrets/zulip.env` in the existing assistant container |
| Autolab Agstudio | ignored `agautolab/.local/zulip.env`, mode 0600, beside the native gateway runtime |
| Autolab Agautolab1 | Ansible copies the controller-only env file to the node checkout's ignored `.local/zulip.env`, mode 0600 |
| Forge | ignored `agforge/.local/zulip.env`, mode 0600, beside the native service runtime |
| Cagent | ignored `pj-clusterintent/.local/zulip/cagent.env`, mode 0600 |

Each file was checked by key name only and contains `ZULIP_URL`, `ZULIP_EMAIL`,
and `ZULIP_API_KEY`. Secret values were not printed.

## Implementation and verification

- `agdevworld` commit `747cc39` adds the read-only assistant mount. The
  assistant image/stack was rebuilt without changing its application code; an
  in-container check read all three key names.
- `ansible_agdev` commit `035da2a` extends the existing
  `AUTOLAB_NODE_*_CREDENTIALS_SOURCE` pattern with
  `AUTOLAB_NODE_ZULIP_CREDENTIALS_SOURCE`. It uses an Ansible `copy` task with
  `no_log: true` and mode 0600 and documents the operator command.
- The corresponding superproject pointer commits are `pj-agdev` `3a0d80a` and
  `pj-clusterintent` `e8603f2`.
- Before deployment, current Nautobot placement was rendered with `nctl render
  production`; 27/27 active placements applied to the generated inventory.
  The playbook syntax check passed.
- Deployment to `agautolab1` completed with 25 ok, 1 changed, 0 failed, and 0
  unreachable. Its gateway remained healthy. An Ansible read-back showed the
  remote credential as mode 0600 and listed exactly the expected three keys.
- The native agstudio autolab gateway and cagent processes run as the owning
  local user and their project-local files are readable by that same user.
  Forge was not running at the check instant; its credential is placed beside
  the service and will be exercised from an actual service process in Step 5.

## Agent run record

- Request/job id: `better_communication/step4`
- Backend: Codex harness + GPT-5
- Outcome: done
- Cost/time: not reported by the harness

DEM note: placed credentials for all in-system agents — handoff candidate.
