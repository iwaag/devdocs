# project_migration p7 — one PC execution node: report

Steps:
[report1](report1.md) inspection and contract ·
[report2](report2.md) authoritative state and assignment ·
[report3](report3.md) worker lifecycle ·
[report4](report4.md) central review ·
[report5](report5.md) deployment ·
[report6](report6.md) real trial.

Physical facts, raw evidence and rollback commands are in pj-agdev's ignored
`.local/project-migration-p7/notes.md` and `.local/devenv.md`.

## Finding

**Adding this PC is a successful, repeatable case.**
- The GPU PC is a persistent, observed WFE execution node.
- Three real playprey runs went through the ordinary request path, each with no hand repair in
  between:
  - development on the PC's own disk and real desktop (Godot windows, captures, the browser Mouth
    Editor connected to Godot);
  - a question answered centrally and resumed on the same PC;
  - outputs on the share, and integration into `main`.
- The existing worker ran other work at the same time.
- A worker outage and restart left honest state and did not duplicate work.

What is not yet a case: any other host (the VM host is deliberately left for a later decision),
and the Developer's own experience of the result.

## Delivered behaviour

- **One coordinator, several nodes.** Every run is recorded by the coordinator; each node has one
  slot.
  - A run is assigned at creation (request, workflow, project, area default) and keeps its node.
  - A job waits for its own node, with the reason, instead of being dropped or sent elsewhere.
    `wfe run assign` is the only move.
  - Reports are fenced by worker and attempt, and repeatable. Claims carry receipts.
- **The PC works locally, the record stays central.**
  - The PC materializes the run from its published branches and stages inputs from its own share.
  - Its `wfe` forwards the agent's run operations to the coordinator, ships its log, and publishes
    its copy.
  - The coordinator's record copy follows the published copy, and integration runs there.
- **Unified review.**
  - The run view names the node, its contact and the remote attempt; Home has one chip per node.
  - Detailed IO, Zulip, questions, presentations and outputs work as for any run.
  - Observer judges a remote attempt by its shipped log and its node's contact.
- **Restarts.** The launch gate records each agent's end. A restarted executor adopts its live agent;
  lost contact is never read as a lost process.
- **Deployment.** The `wfe_worker` profile and role in the existing backend install a PC as a node
  from pinned commits, with its own tokens. A recipe for the next PC is in pj-clusterintent
  `nctl/docs/add-a-wfe-worker.md`. The contract is in agdevworld `workflow_editor/docs/workers.md`.

## Deployment and migration

- Coordinator WFE `ea09c14` (via `14e2b42`). The exec store migrated to schema v3 (301 historical
  attempts → `studio`) after a backup.
- The one unfinished run (jevlab `experiment/run-001`) is assigned explicitly. It was not resumed.
- The PC was provisioned by cagent `change/run-008`: share export and mount, worker placement,
  reconcile, verification. That run surfaced a role defect, which was fixed. Omni accepted the result
  and it is integrated.
- The PC's executor and WFE pins were later updated through desired state and the setup playbook.
- Rollback: activate `ebfe8a5` *and* restore the exec store backup, because old code cannot open a
  v3 store.

Commits (all pushed):

| Repository | Commits |
| --- | --- |
| agdevworld | `58e5c29`, `7745db5`, `14e2b42`, `ea09c14`, `cd4ea20` |
| agautolab | `84ae825`, `0a813b0`, `e975c9b` |
| pj-agdev | `c15bb7b` (Observer), pins `f7d46a0`, `1025271`, `dca6bb8` |
| ansible_agdev | `8616056`, `2ae175d` |
| nctl | gate, recipe |
| pj-clusterintent | `f047e98`, `0995e58`, `b53b6ca` |
| devdocs | the step reports, README_DEV |

## Trial evidence

From report6:
- playprey `freework/run-001` (a question, then a resume on the PC), `run-002` (the browser request
  route, repeat) and `run-003` (requested while the PC was down; its attempt crossed an executor
  stop and start) all ran on gpupc and integrated (journals 74–76);
- cagent `consult/run-005` ran on studio during run-001;
- outputs are `shared://agshare/runs/playprey/freework/run-00N/o1/`;
- during the first capture Godot used 877 MiB VRAM next to an idle ComfyUI, with no contention.

Tests:
- WFE 403/403 (release checks) plus the workers fixture with the AppleDouble regression;
- agautolab 43 passed, 1 skipped; all 13 inherited failures are corrected;
- Observer 115;
- nctl 1358.

## Defects found by real use, and fixed

1. **A lost claim** left a worker checkpoint pending forever. Found by the fixture, before
   deployment.
2. **macOS `killpg` EPERM** on a zombie-only group. Found by the fixture.
3. **The `wfe_worker` role** did not create the executor checkout's `.local/`. Its existence check was
   too early an artifact. Found by cagent's first reconcile.
4. **macOS tar's AppleDouble `._*` entries** were committed by the Linux worker. Found in run-001's
   commits. Fixed in WFE, then cleaned up by run-002 through the normal path.

## Interventions

Omni:
- answered run-001's question and cagent's confirmation on the Developer's behalf;
- resumed cagent's run after the role fix, since its external wait has no waking mechanism;
- made the four fixes and their rollouts;
- stopped and started the PC worker for the availability trial;
- prepared the review launcher.

No record was edited by hand and no agent was started by hand.

## Review entry points

- Central run views: `http://agstudio.local:8093/wfe/#/ws/playprey/run/freework/run-001` (and
  run-002, run-003). Detail shows the node and attempts.
- The outputs' `?inline=1` links (report6).
- On the PC desktop, the icon "playprey review (p7)" opens `Launch.sh` (play / Mouth Editor).

Experiential feedback is pending; technical completion does not depend on it.

## Limitations

- A worker offers every profile in `agents.toml`; on the PC only `sonnet` (claude_code) is installed.
  Codex is the old 0.114, so `luna` would fail at launch.
- A run waiting on an external result is woken only by a person's or Omni's `wfe run resume`
  (pre-existing in WFE; seen in cagent change/run-008).
- nctl compares the `wfe_worker` placement by existence. A pin or config change needs the setup
  playbook, as with NFS.
- Live, unpublished edits of a remote attempt are not visible centrally until its attempt ends. The
  run view says so.
- macOS behaviour of playprey's changed scripts is unchecked. The Mouth Editor's editing was not
  exercised on Linux.

Not part of p7: one worker per host, no automatic placement, relocation, failover or coordinator
high availability.

## Next decision

Whether to add the always-on VM host (CPU-only long tasks) or another PC is the Developer's decision
after looking at this case. The recipe is ready; nothing was deployed beyond the PC.
