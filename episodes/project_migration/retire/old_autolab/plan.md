# Retire old autolab

## Objective

Remove the old `workplan-*` / `workrun-*` development system. Keep autolab as the common executor for WFE-managed workflows. Follow `localrule.md`: delete obsolete behavior and its callers, without compatibility aliases, redirects, disabled fallback implementations or migration adapters. This document is a plan; retirement has not been executed.

The new path is WFE request → queue → `agautolab.wfexec` → a workflow-selected agent in a run-owned copy. Workflow definitions and run state belong to WFE and the managed projects. Keep `wfexec.py`, `wfintake.py`, `wfnotify.py`, the `wfrun` contract, required harness profiles and credentials, and their actual shared dependencies.

## Step 1 — identify and stop the old entrances

- Check current service/placement/agent records through Nautobot or nctl, and inspect the loaded processes. The previous investigation found both the old Zulip listener and the new workflow executor running; recheck at execution time.
- Identify any active old worker and its unfinished branch before stopping it. Record its location and disposition in ignored local notes; do not automatically replay old missions as workflow runs or discard unfinished work.
- Stop and remove the old Zulip listener's launchd/systemd registration and any old worker processes. Remove the legacy gateway if its remaining purpose is the old conversational entrance; first trace any health-probe or deployment dependency so it is removed or replaced together.
- Retire only the legacy entrance/agent records. Keep the autolab service/placement and bot identity needed by the workflow executor and Zulip transport. Do not mark the whole autolab service retired.

## Step 2 — delete the old implementation and its callers

- Remove legacy listener/routing, mission/task records, mission working copies machinery, acceptance/close-out/state-correction tools, and the old `autolab` CLI commands. Candidate modules include `listener.py`, `zulip_listener.py`, `anchor.py`, `worklog.py`, `missionspace.py`, `mission_done.py`, and `correct_state.py`; remove their obsolete tests and fixtures too.
- Trace the old entrance's other functions (`bmining`, project/study setup, archive, argue and gateway roles). Delete those that exist only for the retired system, and update any surviving callers to the existing managed-project path. Do not retain the old listener for these functions or create a replacement service just to preserve them.
- Remove obsolete guides, scripts, role definitions, packaging/workspace entries, configuration fields and local overlays. Simplify `instance.py` to the facts and configuration the retained executor/transports actually need; they currently import its `SPEC`, paths and environment setup. Keep shared functionality only where a remaining caller uses it.
- Search callers in Front, argue, shared `pyagag` tools, WFE, Observer, provisioning and Ansible. Remove instructions and routing that send work to old topics, legacy CLI calls, probes and deployment settings. Remove old-listener service templates and switches so redeployment cannot bring it back. Delete shared helpers only when their remaining consumers are gone.
- Keep historical conversation records, reports and unfinished Git work as history/data, outside active discovery where necessary. Remove obsolete executable local state/configuration after recording unfinished-work locations. Do not rewrite historical episode reports or erase project repositories to eliminate old vocabulary.

## Step 3 — deploy and update the current description

- Deploy the reduced autolab package and changed callers/deployment definitions. Restart the workflow executor only if its running code changed, after the current attempt ends; do not interrupt an unrelated managed run.
- Rewrite autolab's README, current agent introductions, active guides, `devdocs/README_DEV.md` and local environment notes around its executor/transport responsibilities. Remove the old mission instructions from current documentation. Where a legacy introduction must be retired, use the existing board retirement mechanism and preserve any introduction still needed for the executor.
- Record commits, deployment outcome and any remaining historical-only references in a short `report.md` beside this plan. Update affected submodule pins. Keep host details and raw operational evidence in ignored local files.

## Minimum verification

1. Search active source, guides and deployment definitions for the removed modules, commands and topic prefixes. Resolve live callers; historical records are allowed. Confirm the old listener/gateway registrations selected for retirement are absent and the desired deployment no longer recreates them.
2. Run the retained executor/intake/notifier tests and import/configuration checks relevant to the edits. Remove tests for deleted behavior; add no compatibility tests or broad new regression suite.
3. Run one small real request through an existing managed workflow: intake → executor launch → recorded result → notification. Read executor status afterward and confirm it is available. No separate model matrix, failure-injection campaign or legacy mission trial is required.
4. Recheck the affected Nautobot/nctl records once: the old entrance is retired, while the workflow execution service remains active. Distinguish recorded observation age from direct process checks.

Done when the old system cannot receive, route, execute or be redeployed for work, and one managed workflow still completes through the retained executor.
