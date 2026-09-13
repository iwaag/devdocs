# observer p2 — step 2: preparing and requesting the YuE2 study

## The workflow this went through, unchanged

mediagen is the study-pattern project for media-generation know-how
(`main/subjects/INDEX.md`, one row per investigated subject). Its normal
entrance is autolab's: a `workplan-…` topic in the project's own `pj-mediagen`
channel, which plans only, and execution surfaces autolab opens itself. No part
of that was adjusted for this episode.

- Request: `pj-mediagen › workplan-yue2-music-study`, message **6768**, posted
  by the Omni Agent 2026-09-13 07:25 UTC.
- Plan: message **6771**, mission **m6770**, 07:27 UTC — about two minutes.
- Execution surfaces: `#work-m6770` with `workrun-task1-m6770` … `task4`.
- Task 1 started by posting in its run topic, message **6790**.

## Where the model would run, and what that cost to establish

YuE2 needs a Linux NVIDIA GPU with BF16 and 24 GB. This realm has exactly one
candidate — **agpc**, `Quadro RTX 8000`, 47.26 GiB, which already runs the
ComfyUI that mediagen's image and video work uses. The executor runs on
agstudio (a Mac, no CUDA), so the study had to reach the card the way mediagen
always has: over HTTP.

Two things were checked before writing the request, and one of them changed the
shape of the episode:

- **ComfyUI-Manager is not installed on agpc** (`/api/manager/*` → 404), and no
  agent has shell access there. So a custom node could not be installed by the
  executor, by Front, or by me.
- The installed pack `comfyui-vrgamedevgirl` had gained YuE2 nodes upstream on
  2026-09-11, but agpc's copy predated that.

That made the node install a **human step**, and it was put to the developer as
one. The developer instead installed a different, newer YuE2 pack. After it,
agpc serves **1740 nodes including nine YuE2 ones** — `YuE2GenerateSong`,
`YuE2WriteSong`, `YuE2Options`, `YuE2Plan`, `YuE2PlanBatch`, `YuE2SelectPlan`,
`YuE2RenderPlan`, `YuE2DecodeLatents`, and a separate Chinese-language
prompt-enhancer that calls an external LLM API — and ComfyUI is now 0.35.0.

Facts handed to the executor as measurements rather than advice: the URL and
versions, 46.39 GiB free of 47.26, both queues empty, the shared card, and
**the weights are not on disk** — no YuE2 file in the checkpoint, UNET or VAE
lists, so the first run downloads ~7.3 GB. The Turing/BF16 mismatch was stated
as a possible real result ("if it fails, that is a result… do not spend the
whole budget forcing it").

*Deus Ex Machina note: the Omni Agent measured the ComfyUI environment and the
developer installed the node pack — work an in-system agent could not do
today, because nothing in the realm can install on the GPU node. Handoff
candidate.*

## How Observer was asked for, and how it was not

The braindump is explicit that the *use* of Observer must be given before the
request, not inside it. So the request says only this:

> There is an agent on the board whose whole job is waiting; read the
> introductions and use it, for the download and for the generations both. Hand
> it the wait, end your serving, and pick the work up when it tells you the wait
> is over. If it turns out not to fit some wait, say so plainly in your report
> and tell me what you did instead.

No channel name, no topic shape, no field list, no command — those live in
Observer's introduction, which step 1 made reachable with `agentchat intro`
and pointed at from the shared help.

**It worked at the planning stage.** autolab's plan named
`agobserver-agstudio1` and described the mechanism correctly — *"open a
`watch-...` topic in its channel per wait, end the serving, and let it notify
the task's own run topic"* — from the board alone. It also carried forward the
permission to report a bad fit rather than hide it. Whether the **executing**
run does the same is step 3's evidence; a planner reading the board is not yet
an executor using it.

## The bounded budget the executor chose

Asked to choose and state one before starting, autolab wrote it into the plan:

- reproducibility (task 2): **2** generations, one baseline and one identical
  re-run;
- exploration (task 3): up to **3** conditions × **2** seeds = up to 6;
- **8 generations total** for this pass, and — its own addition — task 2 stops
  and reports before task 3 if the Turing/BF16 mismatch makes the baseline
  unreliable or very slow.

Its four tasks are recon/download/first generation, the reproducible baseline,
bounded exploration with honest usable/total judging, and the `subjects/yue2/`
write-up with its INDEX row in one commit.

## What was deliberately left to the executor

Installation of the model weights and how to drive the pack; the graph; style,
lyrics, seed and option values; the order of trials; which waits go to Observer;
and how the music is judged. The request asked for the *records* (pack and
ComfyUI versions, the submitted graph, seed, non-default options, output path,
wall time, `vram_free` before and after) and for who judged the music, not for
how any of it should be done.

## Cost

One autolab planning run (`sonnet`) and one task-start post. The environment
measurement and the node-pack question cost no agent runs.
