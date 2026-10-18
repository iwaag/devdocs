# p7 Step 1 — the PC, the execution contract and the trial

Physical names, addresses, paths and raw evidence are in the ignored notes of `pj-agdev`
(`.local/project-migration-p7/notes.md`). Here: **coordinator** (the WFE service and its area),
**existing worker** (`studio`), **PC worker** (`gpupc`). The worker ids are logical names, not host
names; they appear in run records.

## Starting point

| Item | State |
| --- | --- |
| Installed WFE release | `ebfe8a5` (agdevworld HEAD `e5ce398` changes nothing under `workflow_editor/`) |
| Executor | agautolab `1e90c27`, one slot, heartbeat available, idle |
| Unrelated working changes | none in agdevworld/agautolab; untracked braindumps in devdocs only |
| Non-terminal runs | jevlab `experiment/run-001` (stopped hold), playprey `onboard/run-002` (stopped hold, the Developer's decision in p6) |
| Inherited failures | agautolab `test_wfnotify`/`test_wfconversation`: 13 fail, 30 pass. Cause: hard-coded history numbers (`#4:question` expected, `#5:question` written) since WFE records `attempt.context`. Stale expectations, not a delivery defect; corrected in Step 4. |

Nautobot/nctl: a fresh `nctl observe` of the PC succeeded and drift is converged. The PC carries
two image-generation placements (ComfyUI, SwarmUI) and no executor placement. The only executor
placement is the existing worker's (`manual_toolchain`, process `agautolab\.wfexec`).

## The PC as it is

- Linux (Ubuntu 24.04), x86_64, 32 threads, 123 GB RAM, 1.9 TB free, an NVIDIA workstation GPU with
  48 GB (Vulkan ICD present). Idle at inspection; ComfyUI holds a few hundred MiB of VRAM.
- A real interactive desktop: a GNOME Xorg session on the physical seat, `DISPLAY=:0`, lingering
  user services. `xwd` and `ffmpeg` exist, so a window can be captured.
- Node 25, uv, git, Python 3.12, Blender 5.2 (snap), codex 0.114 (old; `luna` needs ≥ 0.163).
  Claude Code exists only inside the VS Code extension; there is no CLI on PATH. **No Godot.**
- It reaches the coordinator's WFE and Gitea over the LAN. It has no Gitea credential and **no
  shared-area mount** (the share is exported to one other client only).
- Reachable for the agents through the existing Ansible inventory (the inventory's key and host-key
  alias). The direct shell route of this session is refused by the session's permission classifier;
  the Developer granted manual approval.

**Chosen execution environment:** the user's own session on the PC, supervised by a systemd *user*
service (lingering is on), launching agents in run copies on the local disk, with `DISPLAY=:0` for
graphical work. No container: the agent must drive the real desktop and GPU, and the project
tooling is host Python/Node/Godot/Blender. Godot is installed per user (pinned release binary,
same minor as the project's `4.7`).

## What relies on the owning working tree today

| Operation | Today | With a remote worker |
| --- | --- | --- |
| Request, allocation, receipts | `createRunInCopy` on the coordinator: number under the workspace lock, copy cloned, receipt in `copy.json`, job in `exec.sqlite` | unchanged: the coordinator allocates and keeps a **record copy** for every run |
| Run record writes (`wfe run …`, service, executor) | `runOp` on `run.json` of the local copy under its lock | always the record copy on the coordinator; the PC's `wfe` forwards |
| Claim / start / finish / stops / session / context | `wfe exec …` CLI against the local `exec.sqlite`; one global slot; `hostname()` per attempt | same `ExecStore` functions over HTTP; slot per worker; attempts carry `worker` |
| Launch, process group, stop, reconcile | local pid; `reconcile` declares a vanished local pid `unknown` | only the owning worker looks at its pids; the coordinator never interprets a remote pid |
| Brief, continuation, `sinceBase` | paths of the coordinator's copy and the person's workspace | rendered for the claiming worker: its copy root, no foreign paths, host facts it offered |
| Context resolution | `resolveContext(…, host: executor)` | unchanged, resolved at claim on the coordinator, host = the worker |
| Run documents, artifacts, presentations | files of the local copy | the PC copy is where they are written; the forwarding client uploads the run folder and named paths before each command; the coordinator reads its record copy |
| Attempt logs (run view, IO channel, compaction check) | `<area>/.local/exec/logs/<attempt>.jsonl` | the PC worker ships log increments to the same path |
| Shares | the host's `shares.json`; `run input` stages into the copy | coordinator records and hashes as now; the PC stages its own copy from its own mount and verifies against the recorded hashes; `run publish` uploads the folder to the record copy first |
| Checkpoint / publication | coordinator commits and pushes the copy's run branches | for a remote run the worker commits and pushes its copy (with the record snapshot); the coordinator then aligns its record copy with the pushed commits |
| Integration | coordinator, from the copy | unchanged (record copy aligned with the worker's commits) |
| Intake, notification, Observer | central | central; observation gains per-worker facts |

Project access (`wfe projects`), definitions, approvals, context store, delegation, intake and
notification have no dependency on the executing host beyond the above.

## Protocol

One coordinator. Every worker uses the same `ExecStore` operations; the existing worker calls them
through the local CLI (as now), the PC worker through HTTP routes on the coordinator
(`/api/workers/…`). There is one implementation of each transition.

| Operation | Contract |
| --- | --- |
| `register` (in every heartbeat) | worker id, host label, offered profiles and session capabilities, contract hash, host facts (OS, tools, display), copies root, software revision |
| `heartbeat` | per worker; detail and current attempt. The coordinator stores it per worker id |
| `claim` | worker id + offer → the oldest queued job **assigned to that worker** when *its* slot is free. One open attempt per run (the reducer) and per worker slot (the store). A job the worker cannot serve (profile not offered) stays queued with the reason in its note; it is never dropped for that and never handed to another worker |
| `prepare` | on the worker, before `started`: materialize/advance the local copy from Gitea at the recorded branch commits; a missing commit is reported as a preparation failure (`finished --interrupted "preparation: …"`) |
| `started` / `finished` / `session` / `context` | as today, bound to (worker, attempt): a report for an attempt of another worker, or an ended attempt, is refused or answered with the recorded outcome (idempotent) |
| `stops` | per worker; a stop stays *pending* in the run view until the owning worker reports the end |
| `files` | `PUT` a tar of paths relative to a run's copy (run folder, named paths, publish folders, logs at an offset); `GET` a tar of the record copy's run folder (and staged inputs) |
| `cli` | the worker's `wfe` forwards an agent's command: argv, the copy-relative cwd, `WFE_ATTEMPT`, stdin; the coordinator runs its own `wfe` in the record copy and returns exit code, stdout, stderr with the record-copy root rewritten to the worker's |
| `checkpoint` | after `finished`, a remote run's checkpoint row is `mode: worker`: the worker pulls the record snapshot, commits, pushes the run branches and reports the steps; the coordinator then aligns the record copy (`fetch`, move the run branch, restore `run.json` under the run lock) |

Retries: every report carries its attempt id; `finished` on an ended attempt returns the recorded
outcome; checkpoint reports carry the checkpoint id; forwarded commands keep the run ops' own
`receipt`s. The worker persists its bookkeeping (attempt, pid, pending reports, log offset, exit
file) so a restart reconnects without relaunching.

## Run-state cutover

- **Authority:** `run.json` of the record copy on the coordinator. The PC copy's `run.json` is the
  snapshot last pulled; it is overwritten on every pull and never written by the PC.
- **What is read where:** the frozen definition, adjusted revisions, the node catalog, context
  snapshots and the documents the agent wrote are read from the record copy (the coordinator); the
  PC copy carries the same run-folder files after each sync. Authored guides are read at claim from
  the revisions context resolution already records. Nothing is read from the coordinator's unrelated
  project checkout for a run that has a copy.
- **Git:** a remote run's commits are made on the PC (code + run folder + record snapshot). Between
  attempts the coordinator may commit record-only changes (a decision, an integration record) in the
  record copy and push them; the PC fast-forwards at preparation. Commits therefore stay linear;
  a divergence is a preparation problem, never a forced reset.
- **The final integration result** is a record event after the last publication; it is in the
  record copy and published by the coordinator's own after-decision checkpoint, as today.

## Node selection

Precedence, highest first: the request's `--node` (recorded in the run's `run.create` as
`executor.node`, with `via`), the workflow's `execution.node`, the project's `execution.node`
(`project.yaml`), the area default (`<area>/workers.json` `default`). A run keeps its node for all
attempts; a change is an explicit `run.assign` between attempts, recorded with who and why — never
automatic. Unknown node ids are refused at request time. Existing records without a node are
migrated explicitly to the existing worker (`run.assign`, `via: migration`).

## PC requirements

- WFE release (same commit as the coordinator) for the forwarding `wfe` and worker-side Git
  operations; agautolab + pyagag for the executor and harness argv; a standalone Claude Code CLI;
  codex ≥ 0.163 for `luna` (or not offered).
- Godot 4.7.x Linux binary on PATH; `ffmpeg`/`xwd` for capture; Node for the browser editor.
- A Gitea credential for the worker (a dedicated token); GitHub access only if a project needs it
  (playprey does not: one repository, no submodules).
- The shared area mounted read/write, recorded in the worker's `shares.json`.
- A systemd user unit, worker config (`worker.toml`: coordinator URL, worker id, copies root),
  installed through the deployment backend; observed through Nautobot/nctl as an executor placement.

## Trial scope

- **PC task (playprey, `freework`):** make the Godot runtime's launch and capture tools work on
  Linux (`tools/run.sh`, `record.sh`, `godotw`, the launcher's Godot/Node discovery) and record the
  portable requirements separately from the host facts. Evidence: the default profile launched in a
  window on the PC desktop and captured (screenshot and a short movie), published as a run output.
  It asks the person which profile/view to capture by default (question → resume on the PC), and
  the script/README changes integrate. Inputs: the runtime assets on the share (`assets.py fetch`).
- **Concurrent existing-worker task:** a cagent `consult` on what runs on the PC now (small,
  read-only), run while the PC attempt executes.

Routine design choices from here on are made during implementation and recorded in the step
reports.
