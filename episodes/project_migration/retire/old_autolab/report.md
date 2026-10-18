# Retire old autolab — report

Steps: [report1](report1.md) (entrances), [report2](report2.md) (deletion), [report3](report3.md) (deployment and documentation).

## Outcome

The old `workplan-*`/`workrun-*` system no longer receives, routes or executes work, and nothing can redeploy it:
- its listener and both gateways are stopped and unregistered;
- its introduction is retired on the board;
- its code, CLI, guides, roles, launchd templates and Ansible role are deleted;
- the shared pyagag parts that only it used are deleted, along with agentroom's Project Room.

autolab remains as the WFE executor with its Zulip transports. One managed workflow run, `cagent consult/run-004`, completed through the restarted executor and was delivered to Zulip.

**One part is pending:** the Nautobot desired-state commit (report1). The auto-mode classifier refused `nctl desired apply --yes`. Until it is applied:
- Nautobot still declares the `agautolab-agautolab1` placement and the `autolab-agautolab1` agent;
- both placements still declare the deleted `autolab_node` profile, so `nctl render production` and `nctl reconcile` will reject them.

The batch files are `pj-clusterintent/.local/retire-old-autolab-pass{1,2}.yaml`. The pass 1 preview was clean (4 deletes, 1 update, 0 conflicts).

## Commits

| repo | commits |
|---|---|
| agautolab | `688e404` deletion, `8440d65`/`1e90c27` README |
| agdevworld | `e5ce398` |
| pyagag | `03a2a32` |
| agfront | `a6a0370` |
| archsage | `4dac08d` |
| pj-agdev | `9576f7e` devenv/Observer, `782fe71` pins |
| ansible_agdev | `f86e590` |
| nctl | `d5659a1` |
| pj-clusterintent | `0280b2c` pins |
| devdocs | this episode's reports, README_DEV |

## Minimum verification

1. **Search**. No live caller of the removed modules, CLI commands, routes or prefixes remains. The remaining matches are history (fixture boards and recorded JSON, trace/progress docstrings, retired agforge, episode reports, regenerated scan JSON) and WFE's executor identity `autolab`. The listener and gateway registrations are absent on agstudio and on agautolab1, and no template or role recreates them.
2. **Tests**:
   - agautolab: 30 passed, 1 skipped, 13 failed in `test_wfnotify`/`test_wfconversation`. The same 13 fail on the previous HEAD against the current WFE (event-number expectations); this predates the retirement and was not fixed.
   - Observer 114, nctl 1358, agentroom 268, pyagag 1174, agfront 185 and archsage 41 passed.
   - The frontend `tsc` and build are clean.
   - The retained modules import, and the executor resolves its profiles.
3. **Real run**. `consult/run-004`: queued, launched, exit 0, `answer.md` recorded, notified (message 19775). Afterwards the executor reported "available" and intake "Transport: ok". The request came through `wfe request`, not a Zulip post under the Developer's account.
4. **Nautobot recheck: not done**, because it waits for the commit above.
   - Direct process checks (2026-10-19): only `com.agdev.agautolab-wfexec` runs among the autolab labels, and agautolab1 has no autolab unit.
   - Recorded observation: `nctl relations` before the change showed both placements `satisfied`, an observation from before the stop. Its age was not re-read after the stop.

## Remaining historical-only references

- The old `pj-<slug>`, `work-m<id>` and `autolab-agstudio1` channels and their topics. Nothing serves them.
- The project clone sets in agautolab `.local/projects*`. Every `autolab/m*` branch is integrated.
- The `.local/topics` and `.local/executions` records.
- The checkout on agautolab1.

## Follow-ups

- Apply the Nautobot batch, then run `nctl reconcile agstudio --refresh-observation --max-rounds 1 --yes` and `nctl agents observe`, and recheck relations and drift (verification 4).
- `agag.trace`/`agag.progress`/`holds` still read missions and acceptance from history. They can go once the panel no longer needs to show those conversations. pyagag's unused scripted fixture responder can go as well.
- The 13 pre-existing notifier/conversation test failures against the current WFE.
- The intros `intro-archsage-agstudio1` and `intro-agforge-agstudio1` are still live on the board, although both agents are retired. They were outside this episode.

## Mistake

Clearing the removed `agent/front` workspace member also deleted its ignored `.local/topics/` chatlogs. Those were trial mission records from 2026-08-13. They were untracked and are lost.
