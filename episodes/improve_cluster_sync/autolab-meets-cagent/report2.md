# Step 2 report — cagent can write, but cannot destroy

Status: complete (2026-08-09).

cagent can now preview/apply desired-state batches and run ordinary
reconciliation. Its OpenCode permission boundary denies destroy-class command
surfaces, and its instructions retain the separate no-secret and
loopback-only OpenCode requirements.

## Changes

- Removed the hard denies for `nctl desired apply ... --yes` and ordinary
  `nctl reconcile ... --yes`.
- Added hard denies for `--allow-destroy`, `nctl prune`, braindump
  purge/review-delete, and `playbooks/proxmox/destroy_*`.
- Rewrote cagent's instructions around the new recoverable-write boundary and
  linked the partial-batch/basic-service recipes.
- Recorded the 2026-08-09 repeal of cluster-agent roadmap prohibition #1.
- Restarted the dedicated loopback OpenCode process; cagent-api did not need a
  restart. The rendered live config contains the same five deny patterns.

## Recovery safety net

Before the first cagent write, a canonical desired export and a fresh
PostgreSQL custom-format dump were retained in the ignored backup directory:

- `.local/backups/desired-autolab-meets-cagent-step2-20260809.yaml`
- `.local/backups/nautobot-autolab-meets-cagent-step2-20260809.dump`

The dump is 3,158,871 bytes and `pg_restore -l` inside the PostgreSQL
container listed 2,426 lines.

## Live evidence

Recoverable write request `req_9fdce08de7f949c68457b94736d99a32`
(session `ses_01bfa0724ffevXzBLwScp4bzKe`) completed through the human entrance.
cagent exported current desired state, previewed the exact file, and applied
it with `--yes`. Both results were no-ops: 43 unchanged, zero create/update/
delete/conflict; the apply transaction status was `committed`.

Hard-deny request `req_3962255e3059434091a04bf3c62f19b8`
(session `ses_01bf81d67ffePq6xHHhVMKJ1fC`) attempted the safe dry-plan command
`nctl reconcile agautolab1 --allow-destroy`. OpenCode logged the exact match:

```text
action.pattern=*--allow-destroy* action.action=deny
```

No command execution, retry, or substitute occurred. Durable transcripts are
under `~/.local/state/cagent/evidence/<request-id>/`; sanitized poll responses
are retained in `.local/cagent-step2-*-response.json`.

Validation: `uv run --project cagent pytest -q cagent/tests` — 92 passed.
