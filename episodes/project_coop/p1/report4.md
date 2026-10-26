# project_coop p1 — Step 4 report: environment facts in actual run workspaces

Date: 2026-10-26.

## Where each fact comes from

| Fact a run needs | Owner and update mechanism | How it reaches a run |
| --- | --- | --- |
| Other projects' contracts (entries, delivery docs, tools) | each producer's Git, published `main` | `wfe projects read|show|search` (no checkout); a producer's own run gets its guide through `project.yaml` `guides` |
| Where `shared://agshare/…` is on this host | the WFE area's `shares.json` per host (deployed by cagent's `wfe_worker` profile; agstudio: the backing folder, agpc: the NFS mount) | `wfe share resolve` inside the tools (`assets.py`, protoprey's `deliveries.py`) |
| The same outside WFE | the person running the copy | `PROTOPREY_SHARE_ROOTS=agshare=<path>` or an explicit package (`deliveries.py pack` / `install --from`) |
| `wfe` itself | the executor | prepended to the agent's PATH (`agautolab/wfexec.py`) on both workers |
| Godot and Python | host installation (agstudio Homebrew, agpc `~/.local/bin`) | the executor's service environment (launchd PATH includes `/opt/homebrew/bin`; agpc worker PATH includes `~/.local/bin`) |
| Generation-service endpoint (ComfyUI) | the environment: `COMFY`; described by cagent's resource context | gimmickimg's tools now refuse to start without `COMFY` instead of assuming a host name (gimmickimg `826fdea`) |
| Resource overview per project | cagent, `wf-cagent.context` → `cagent-resources` blocks | delivered in every run's scoped context |
| A consumer's need for generation endpoints | none: protoprey delegates generation and receives a delivery | — |

No WFE or executor change was needed; the existing mechanisms cover the case.

## Checks

- **agpc (second worker), 11:39 UTC**, as the worker's environment sees it: `wfe`, `godot` and `python3` on PATH;
  `wfe share resolve` of a published playprey output answered `ok` at the agpc mount path; `wfe projects read` returned
  gimmickimg `docs/delivery.md` at `2e0a52f` and protoprey `docs/cooperation.md` at `91821dc`, i.e. the Step 2/3
  commits, without any local checkout.
- **agstudio**: the same commands from the WFE area; the executor's launchd PATH includes Godot.
- **Fresh copies**: a run's copy has no ignored files; playprey runs fetch profile assets with `assets.py fetch`
  (share through `wfe share resolve`), protoprey runs obtain adopted deliveries with `deliveries.py install`. Neither
  relies on a file Omni Agent placed.

## cagent's resource context (existing mechanism, exercised)

Request "project-coop-p1-cagent-context" → cagent `context/run-008` (studio, 2 min, completed). It published a
`cagent-resources` block for protoprey, gimmickimg and playprey, with evidence (cluster collection 01:42 UTC; its own
probes at 11:40 UTC: Godot 4.7.2 on its runner, ComfyUI API answering with 24 checkpoints, share listing).

Observed shortcomings, recorded and not hand-edited (the blocks are cagent's):

- The three blocks have the same text. cagent's guide says a project block adds what matters for that project; the
  protoprey block talks about gimmickimg's ComfyUI client.
- It probed only from its own runner and says so; it did not check agpc's Godot or renderer, which the request asked
  for. The facts from the Omni check above (agpc Godot 4.7.2, NVIDIA RTX-class GPU, share mounted) are therefore in
  this report and the ignored notes, not in the context.
- It stated the ComfyUI endpoint by quoting gimmickimg's tracked code default. After `826fdea` that sentence is out of
  date ("defaults to"); the endpoint itself is unchanged. A refresh is due after the trial (Step 6).

## What changed

- gimmickimg `826fdea`: `experiments/window-loop/tools/comfy.py` and the two Wan scripts read `COMFY` and stop with a
  message naming the run context when it is absent.

The trial (Step 5) was queued at the end of this step: protoprey `freework/run-001` on studio, request receipt
`project-coop-p1-trial-1`.
