# p7 Step 6 — the PC worker doing real work

The trial ran on 2026-10-19, between 03:16 and 03:38 JST. Coordinator WFE was `14e2b42` for the
first run and `ea09c14` from the second on; executor `0a813b0`. Raw evidence: pj-agdev
`.local/project-migration-p7/step6/` (observations, worker states, resource samples, screenshots).

## Runs

| Run | Node | Attempts | What happened |
| --- | --- | --- | --- |
| playprey `freework/run-001` | gpupc | 2 (exited, exited) | Linux support for the launch/capture tools; a question; answered; capture recorded |
| cagent `consult/run-005` | studio | 1 (exited) | What runs on the PC; ran while run-001's first attempt ran on the PC |
| playprey `freework/run-002` | gpupc | 1 (exited) | Repeat request through the browser's HTTP route: clean-up and a readable capture |
| playprey `freework/run-003` | gpupc | 1 (exited, across an executor stop/start) | Mouth Editor on Linux, asset fetch fix |

## Evidence per requirement

**Ordinary request.**
- run-001 came from `wfe request freework --node gpupc`.
- run-002 came from the service's request route (`POST api/workspaces/playprey/runs`, the browser's
  entrance) with `node` and `predecessor`.
- Each request created one run, queued for gpupc. The PC's executor claimed it. Nobody started an
  agent by SSH.

**Local development.**
- The PC built each run's copy from the published run branch (playprey `.git` about 354 MB, cloned
  over the LAN).
- The agent fetched the runtime assets from the share with the project's own
  `assets.py fetch` (through `wfe share resolve` on the PC).
- It edited `godotw`/`run.sh` (`caffeinate` only on Darwin) and added `Launch.sh`,
  `tools/capture_default.sh`, a capture-only `--exposure` option and `tools/x11_arrange.py`.
- It launched Godot 4.7.2 (Vulkan, Quadro RTX 8000) in a window on the PC desktop and captured it:
  - run-001: a 1280×720 screenshot and a 150-frame Movie Maker movie;
  - run-002: brighter, mean luma 38 → 106;
  - run-003: the whole desktop with the Godot preview connected to the browser Mouth Editor
    ("Godot connected — revision 0").
- Resource sample during the first capture: Godot used about 97 % of one core and 877 MiB VRAM.
  ComfyUI stayed idle at 260 MiB. Load average was 0.5, and 105 GB of memory was available. No
  contention, so no restriction was added.

**Question and resume.**
- run-001's first attempt asked q1 (which model and view should be the default capture, with
  options) and ended.
- The question was recorded on the coordinator and delivered to the run's Zulip topic like any run's.
- Omni answered for the Developer: "horse_combined / exit". The answer queued a resume *for gpupc*.
  The second attempt ran on the PC in the same copy (`copies/playprey/freework/run-001/tree`) and
  took the answer up.
- The policy is `fresh`, so it was a new session from the records, as designed.

**Parallel work.**
- The PC's run-001 attempt and the existing worker's consult/run-005 executed at the same time.
- `/api/workers` showed both slots running (studio: consult/run-005; gpupc: freework/run-001), and
  `wfe observe` showed both runs `executing` with their own node.
- Neither took the other's job.

**Delivery and integration.**
- Each run published its output on the share (`o1` of each run: shot, movie, desktop screenshot,
  `info.json`), shown in the central run view. The images open in the browser through the share
  route's `?inline=1`.
- Each end of attempt was a worker checkpoint, published by the PC, after which the coordinator's
  record copy followed.
- All three runs integrated into playprey `main` by fast-forward (journals 74, 75, 76) with no
  manual step. The central run view says "Integrated into main".

**Availability and recovery.**
1. With the PC executor stopped, run-003 was requested. It waited, honestly: `gpupc unavailable`
   (49 s), job queued *for gpupc*, position 1. The existing worker did not take it.
2. Started again, the PC claimed it.
3. 22 s into that attempt the PC executor was stopped again. The agent kept running
   (`KillMode=process`).
4. The coordinator showed the attempt still `executing` and the worker `unavailable`, with last
   contact 48 s. Observer's classifier said quiet `worker-down`, with no incident.
5. On restart the executor logged `adopted pid … left running before the restart`.
6. The attempt ended once (exit 0): one attempt, no duplicate, same run identity, integrated.

**Repeatability.**
- run-002 and run-003 used the deployed PC with no repair, file copy or installation in between.
- The one change between run-001 and run-002 was a defect fix, rolled out by the same deployment
  path (below).

## Defect found and fixed in the trial

run-001's commits included six AppleDouble files (`._run-001`, `._braindump.md`, …).
- **Cause.** The coordinator's macOS `tar` adds `._*` entries for extended attributes when it sends
  the record copy's run folder. The PC's GNU tar extracted them as ordinary files, and the worker
  committed them. They reached `main` with run-001's integration.
- **Fix.** WFE `ea09c14` runs tar with `COPYFILE_DISABLE=1` on both sides. A fixture asserts that the
  record copy's tar carries no `._` entry; the fixture fails without the fix.
- **Rollout.** Coordinator release, then the PC's `wfe_version` updated through desired state and
  the setup playbook.
- **Cleanup.** run-002 removed the files from the repository through the normal run and integration.

## Omni's interventions

- Answered run-001's q1 on the Developer's behalf, as authorized.
- Fixed the AppleDouble defect and rolled it out (WFE release; PC pin through desired state and the
  setup playbook).
- Stopped and started the PC's `wfe-worker.service` for the availability trial.
- Prepared the review launcher (below). No run's state file was edited and no attempt was started by
  hand.

## For the Developer to look at (from the LAN or the PC)

- **Run views** (central, Detail shows "Execution node: gpupc … last contact …", attempts "on
  gpupc"):
  - `http://agstudio.local:8093/wfe/#/ws/playprey/run/freework/run-001`
  - `…/run-002`
  - `…/run-003`
- **Outputs:** `http://agstudio.local:8093/wfe/api/shares/agshare/runs/playprey/freework/run-00N/o1/?inline=1`.
- **On the PC desktop:** the icon "playprey review (p7)" (`~/wfe-worker/review/launch-playprey.sh`)
  opens the menu of `Launch.sh` (play / Mouth Editor) in run-003's copy, which has the assets and
  the built editor. The integrated `main` holds the same files.
- Experiential feedback is still pending. Technically the round trip is complete.

## Limits observed

- macOS behaviour of the changed scripts is unchecked (the agent says so in every report).
- The capture exposure (2.5) was judged by eye on one view.
- The Mouth Editor's editing (Apply/Save) was not exercised, only its connection and display.
