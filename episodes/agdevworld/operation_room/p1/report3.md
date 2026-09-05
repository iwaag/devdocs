# Step C — attributing backend activity to a task

Investigated item **C** of `plan.md`: can ComfyUI's queue be tied back to a
work item, and what does ollama actually report. Probe: `opsprobe.stepc`.
Read-only against Zulip and ComfyUI; the ollama half is described below.

## ComfyUI — the join key exists, and it is live-only

The plan is right that `prompt_id` is the one path from a backend job to a
task. Both ends of it were confirmed to exist:

- **task side, on disk.** `.local/agentws/<work id>/generator/pending.json`
  carries `{prompt_id, note}` and is renamed to `watching.json` with a
  `trigger` field when the listener tickets it
  (`agforge/assetrun_topic.py:342`). `<work id>` is the Plane Sub-Work id, so
  this file attributes a backend job to a *task*, not just to a topic. 18
  work directories exist on agstudio; **0 job files right now**, because
  nothing is generating.
- **task side, in chat.** The ticket is a post —
  `@**Comfy Notifier** watch <prompt_id>` — so the prompt_id is also readable
  from Zulip with no filesystem access at all. 12 such tickets found in
  history, in `agforge-agstudio1/assetrun-…`, `work-m-51/workrun-…`,
  `front/front-comfy-command-relay` and `general/…`.

**But none of the 12 resolves in ComfyUI today: 0/12 in `/history`.** The
server holds 600 history entries spanning **2026-09-04 03:02 to 2026-09-05
14:50 — about 36 hours** — and evicts the rest. The join is therefore usable
for a job that is queued, running, or minutes old, and useless for anything
historical. An operation room can say *what is generating now, and for whom*;
it cannot reconstruct yesterday from ComfyUI. The durable record of a past
generation is the Zulip ticket and the forge workspace, not the backend.

**And the queue is not forge's.** Of those 600 entries, **599 ended
`execution_interrupted` and exactly one succeeded.** That traffic is SwarmUI's
— the image path goes through SwarmUI onto this same ComfyUI process, and it
interrupts constantly. Two consequences for the screen:

- a "ComfyUI queue depth" tile mostly shows activity that belongs to no
  agent and no task, at a ratio of about 599 : 1;
- **the image path has no prompt_id at all.** SwarmUI returns none (README_DEV,
  `agent_standardize`-era note), which is why image generation stayed
  synchronous. Image generations are structurally unattributable.

`/system_stats` is worth having beside `/queue`: it reports the device and
`vram_free` / `vram_total` (currently 49.6 of 50.7 GiB free on a Quadro RTX
8000). Queue depth says whether work is *waiting*; VRAM says whether the card
is *occupied*, including by a job this system did not submit and by ollama's
resident models. Neither is attributable; both are honest.

## ollama — `/api/ps` reports residency, not sessions

The braindump asks for "the number of running local ollama sessions".
**That number does not exist.** `/api/ps` was hit while idle (`{"models": []}`)
and then while a model was resident. One small model (`gemma3:latest`, 3.3 GB)
was loaded deliberately with `keep_alive: 45s` to see a non-empty answer, on
the host that runs ollama — which is **not** the ComfyUI host, so nothing was
taken from the generation GPU; it unloaded itself within 30 s, as watched.

A loaded entry is:

```
name, model, size, digest, details{family, parameter_size, quantization_level},
expires_at, size_vram, context_length
```

That is the whole of it. **No request count, no concurrency, no client
identity, no caller, no start time.** A model is listed from its first request
until `keep_alive` expires, so "listed" means *resident*, not *busy*: an idle
model with a live keep-alive is indistinguishable from one mid-generation.

So ollama gives exactly two honest facts: which models are resident and how
much VRAM they hold. Which agent, which role, which task — not derivable, and
not derivable in principle, because the API has no field that could carry it.
It goes on the cannot list.

The one real use is negative and still worth a tile: **a resident model is
VRAM that a generation cannot have.** That is the thing the developer
actually gets bitten by (`comfy_video.py:95`, the 45 GiB workflow that has
failed twice on resident models), and `size_vram` states it exactly.

## An operational trap found by accident: `.local` names and macOS

Halfway through, every ComfyUI call started failing with
`OSError 65 EHOSTUNREACH` while the host was up and answering the same URL
from another shell. The cause is not the network: the backends are mDNS
`.local` names, and **macOS grants Local Network access per binary**. The
probe's uv-managed CPython had never been granted it; agforge's own venv
interpreter had. Rebuilding the probe's venv on the framework interpreter
fixed it instantly, with no network change.

For p2 this is a deployment constraint, not a curiosity: an operation room
process that polls LAN backends must run under an interpreter that has been
granted Local Network access, or **every backend will read as down while
being perfectly healthy** — the worst possible failure for a status screen.
Related and separate: `.local` lookups on this network also stall on
unanswered AAAA records. Treat an unreachable backend as **unknown**, never as
**down**, and show the difference.

## Summary of what attributes to what

| backend fact | source | attributable to | freshness |
|---|---|---|---|
| a video/music job is queued or running | ComfyUI `/queue` | task, via `prompt_id` ↔ `pending.json`/`watching.json` (Plane Work id) or ↔ the notifier ticket post (channel/topic) | live only |
| a job finished, and its outputs | ComfyUI `/history/<id>` | same | ~36 h, 600 entries, evicting |
| an image was generated | — | **nothing** (SwarmUI returns no prompt_id) | — |
| GPU occupancy | ComfyUI `/system_stats` | nobody | live |
| resident models and their VRAM | ollama `/api/ps` | nobody | live, with `keep_alive` lag |
| number of ollama sessions | **does not exist** | — | — |
