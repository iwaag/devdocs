# project_migration p3 ex1 — step 3: retirement and release

Written 2026-10-14.

## Nautobot retirement of old agforge: not completed (blocked)

- **Current state** (`nctl desired export`, read before any change):
  - `desired_service agforge`: `lifecycle: active`;
  - `desired_agent agforge`: `lifecycle: active`;
  - `desired_service_placement agforge/agforge-agstudio`: `desired_state: active`.

  All three changes were still needed. `desired_workspace agforge` stays `active`. The checkout and its `.venv` serve comfynotify, and the batch does not touch them.
- **Batch re-read**: `pj-agdev/.local/project-migration-p3/retire-agforge.yaml`. It holds exactly these three upserts: service and agent `retired`, placement `disabled`.
- **Preview**: `nctl desired apply -f …retire-agforge.yaml` gave `update 3, create 0, delete 0, unchanged 0, conflict 0`. It was saved to `.local/project-migration-p3/ex1/retire-preview.txt`, with the pre-change export beside it.
- **Apply refused.** The exact action was `cd pj-clusterintent && uv run --project nctl nctl desired apply -f ../pj-agdev/.local/project-migration-p3/retire-agforge.yaml --yes`. Claude Code's auto-mode classifier refused it with reason `[Blind Apply]`, as in p3. It was not bypassed.
- **Remaining:** the Developer runs that one command. It is the sole remaining retirement task. Until then, `nctl drift` will report the agforge placement once the next node dump is collected.

## Editor release

- `agdevworld/scripts/wfe-release.sh deploy .local/agdev`. It built and checked release `e1e6be3` (`npm ci && npm run check` in the release: passed), switched `service/current` from `75cdae7`, restarted `com.agdev.wfe-service`, and the service and `bin/wfe` agree on `e1e6be3`.
- Before the release, the stores were backed up (`.local/agdev/.local/backups/pm-p3ex1-*`). No store migration was needed.
- The executor runs `bin/wfe exec reconcile` every minute, so the queue fix is live without restarting it.
- **Smoke check** on the deployed editor over the LAN (`http://agstudio.local:8093/wfe/`), read-only, in Simple:
  - forge `freework/run-001`: chip *Decision needed*; focus card "Completed — your decision is needed", with *Accept result* beside the status.
  - `provide/run-002` and `develop-capability/run-001` (the two that were accepted twice in p3): *Accepted · integrated*, with no Accept offered.
  - No page errors.

  The write paths (pending, failure, success, renewed acceptance, double click) were exercised on isolated data by `checks/approval.ts` (step 1). No real result was accepted again.

## Prepared for the Developer

The correction run is the directly accessible affected run: completed, no decision, and its acceptance is the Developer's own.
`http://agstudio.local:8093/wfe/#/ws/forge/run/freework/run-001`
