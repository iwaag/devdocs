# p11 Step 5 — migration and deployment

Done on 2026-10-18 (JST).

## Existing state

- No execution attempt was open and no job or integration was queued.
- Three runs were unfinished, all held by an explicit stop the Developer
  asked for: jevlab `experiment/run-001`, playprey `compare/run-001` and
  `onboard/run-002`. They stay stopped; recovery cannot lift a person's hold.
- Approvals, decisions, answered questions and frozen definitions are facts
  in the records and needed no rewrite: the new code reads them as records
  and gates nothing on them.
- The ignored `.local/cagent.toml` no longer holds `approvers`/`playbooks`
  (Step 4). Run copies receive project guides from the copy itself at claim
  time; the three held runs keep their frozen copies until someone resumes
  them, when the copy's newer commits are named in the brief.

## Record migration

The new release checks each stored `change` projection against what the
operation now does. A dry check of every `run.json` under the managed
workspaces (406 files, run copies included) found 117 failing. Of those, the
records WFE actually reads were 24 workspace files and 8 records in the
run's own copy (25 distinct runs, all completed). The rest were other runs'
base snapshots inside copy trees and frozen presentation bundles (whose
bytes are pinned by their manifest hashes); neither is read as a run's
record, so they were left alone.

`workflow_editor/scripts/migrate-process-state.ts` reprojected the 32
records: all `completed → completed`; history, actors, decisions and
definitions unchanged; original bytes in ignored
`pj-agdev/.local/workflow-editor-p11/migration-backup/`. A second check
passed 32/32. Commits per repository ("p11 step 5: reproject run records"):
agsfx, cagent, forge, jevlab, playprey, ppt, rts-vs-bot on `main`, and the
eight copies on their `wfe/<project>/<workflow>/<run>` branches. Those eight
run branches had been removed from Gitea after integration; the push
re-created them there.

## Deployment

| Item | Value |
| --- | --- |
| Source | agdevworld `a129662`, agautolab `47f7f7c`, pinned in pj-agdev `7f5e296` |
| Release build | `scripts/wfe-release.sh deploy`: `npm ci` + `npm run check` 380/380 in the release folder |
| Activation | `service/current` → `releases/a129662…` (was `e19fa04`), `com.agdev.wfe-service` restarted |
| Health | service `GET api/version` and `bin/wfe version` both report `a129662`; the editor answers from the LAN name (HTTP 200) |
| Executor | `com.agdev.agautolab-wfexec` runs `bin/wfe` and reads `agent/guides/wfrun/guide.md` at each launch, so new attempts get the Step 4 guide without a restart |
| Cluster backend | nctl `status --json` ok before deployment; nctl/backend changes from Step 3 and the ansible_agdev README are on the live checkout |

The release script's one guard, refusing activation while an attempt is
open, was kept: it reports the open attempt so it can be stopped, and
prevents replacing the CLI under a live process. It asks no one's approval.

## Verification

- `wfe observe` under the new release: 0 problems; the three runs report
  `stopped` with their original holders.
- `wfe status` for every workspace: 75 runs (71 completed, 1 cancelled, 3
  in progress and held), 0 read problems, 0 issues.
- Installed `wfe run help ack` shows the new text ("no transition waits for
  it").

By the Developer's instruction no fresh agent attempt was launched and
Step 6 is not run.
