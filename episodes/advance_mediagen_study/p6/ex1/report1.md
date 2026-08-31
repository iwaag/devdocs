# advance_mediagen_study p6 ex1 — step report 1

Plan: `plan.md`. This file covers **Step 0, the Omni Agent preflight**, and
carries **the fire** when it goes out.

Host literals (address, ports, model filenames) are deliberately absent here,
per the plan; they live in the workspace files the plan names.

## Step 0 — preflight, 2026-08-31

All four items done, including the optional one. **Nothing in the environment
blocks the fire**, and item 3 found and fixed a real defect before it could
silently spoil the exercise.

### 1. Is a second Omni Agent session driving?

`agentchat read front front-routine-mediagen` — the topic ends at
**2026-08-31 11:26 UTC** with Front's close of mission m-39: both repositories
pushed, all four tasks done, *"nothing further pending from autolab on this
fire."* The last Developer post (11:25) is p6's own go-ahead, written by this
session's predecessor on this fire.

**No Developer post that this fire did not write, and nothing live in the
project.** This session takes the fire.

### 2. Host state

| check | reading |
|---|---|
| `GET /queue` | running 0, pending 0 |
| `GET /system_stats` | ComfyUI 0.33.1, 123.4 GiB system RAM |
| card | Quadro RTX 8000, **46.46 GiB free of 47.26** — cold |
| host disk | 1.3 TiB free of 1.8 TiB |
| `launchctl list` | `com.agdev.comfy-notifier` running |
| notifier log tail | three `success posted` lines from the notifier episode's own steps, no errors |

The card is above the plan's 40 GiB floor with the whole card idle, and the
queues are empty. One ComfyUI process, as always.

The log's older `post_failed [Errno 2] No such file or directory: 'agentchat'`
lines are from the notifier episode's own step 2, before `agentchat` was put
on the daemon's PATH. They are history, not a live fault.

### 3. The 124-output post — the preflight's finding

The plan asked whether the notifier truncates a 125-output `success` record or
whether the post fails. **Neither. It is the third possibility, and it is the
worst one: Zulip accepts the message, truncates it silently, and every party
in the chain reports success.**

Probed against p6 task 3's video job, still in `/history`, into a scratch
topic. Two facts, one measured offline and one live:

| | before the fix |
|---|---|
| outputs in the record | **125** |
| rendered message | **27 333 characters** against a ~10 000-character cap |
| Zulip's response | **accepted**, truncated, `[message truncated]` appended |
| daemon log | `success posted` — no error anywhere |
| what reached the topic | 46 of 125 output entries, cut mid-token |
| `prompt_id` in the body | **lost** |
| `state`, `wall_s`, `vram_free` | **lost** |
| JSON block | unparseable — no closing brace, no closing fence |

The record's keys are serialised sorted, so `outputs` precedes `prompt_id`,
`state`, `vram_free` and `wall_s` — the four fields a callback exists to
deliver are exactly the four the truncation eats. Only the headline survives
intact (`comfy success 3065fca5 in 1s`), carrying the state, the wall time and
**eight characters** of the prompt id.

**Why this is worse than a refusal.** A refused post leaves a `post_failed`
line, keeps the ticket, and retries — noisy, but the ticket is never lost. A
truncated post archives the ticket and writes `posted`. The receiving agent is
resumed on schedule by a callback that looks healthy and cannot be parsed, and
nothing in the daemon, the log or the topic says so.

**Decision: (a), implemented, plus (b) in the fire text.** The plan offered a
choice; the two are not exclusive and the first one is four lines.

- **(a) — done.** `comfynotify` now caps the `outputs` list at
  `COMFYNOTIFY_MAX_OUTPUTS` (default **6**) and carries the true count as
  `outputs_total`. A callback names a finished job; it is not a manifest. One
  regression test asserts the capped post is under the limit, is parseable,
  and still carries `prompt_id`. Daemon reloaded, and the *same job* re-probed
  live:

  | | before | after |
  |---|---|---|
  | message | 27 333 chars, unparseable | **1 758 chars, parses** |
  | `prompt_id` | lost | **full id present** |
  | `state` / `wall_s` / `vram_free` | lost | present |
  | output list | 46 truncated entries | 6 entries + `outputs_total: 125` |

- **(b) — goes in the fire.** The run's `finish` step re-reads
  `/history/<prompt_id>` itself and never depends on the callback's list. This
  needs no notifier change and it is the right shape regardless: the callback
  carries the *id*, the run fetches the *bytes*.

Worth recording for its own sake: **with 125 outputs the full prompt id never
reached the topic at all.** Option (b) alone, as the plan phrased it — *"the
callback only needs the id"* — would have had to reconstruct the id from an
8-character prefix matched against the run's own `pending.json`. That works,
but it was not what anyone thought they were relying on.

Committed to `pj-agdev` as `5107252`, pushed.

### 4. Optional — the cat still by hand

Three seeds at the p6 still literals (1024², 25 steps, cfg 4, `euler`/`normal`)
with the pose clause replaced per the plan: *neutral standing side view, all
four paws on the ground, idle stance*. **~9.1 s each, all three.**

`ex1_preflight_cat_candidates.png` beside this file, opened and judged by eye.

| seed | verdict |
|---|---|
| 12345 | **side view**, whole body, four paws down, tail raised — a clean sprite, but the cat fills the frame nearly to the top edge |
| 777 | **rejected — three-quarter, facing the viewer.** Not side-on |
| 20260831 | **side view**, whole body, four paws down, generous headroom above the cat |

**The prompt reads.** Two of three seeds gave the wanted pose, which is the
only thing this item had to establish, and the third is the exact failure the
plan predicted — that is why the theme says render ~8 and pick by eye.

**One observation the fire should carry, and the run should still own the
choice.** The action datasets are a *jump* and an *idle*. A jump needs room
above the character inside a fixed 640×640 frame; seed 12345's cat has almost
none and seed 20260831's has plenty. So "side-on, whole-body, centred" is not
the whole of the selection rule here — **headroom is part of it for dataset
A**, and that is not something p6 ever had to think about with a walk cycle.
The seed choice stays with the run.

Both known inherited defects are present and unchanged: the background is
lavender/pale grey rather than white, and a cast shadow sits under the paws
sharing the outline's palette family. Neither blocks anything. Seed 20260831
also puts a small green ground patch under the paws — one more thing the key
will not remove.

### Preflight verdict

**Fire.** The card is cold and free, the queues are empty, the notifier is
running and its one blocking defect is fixed and re-proved live, and the
character prompt produces the wanted still in about nine seconds.

## The fire, 2026-08-31 12:2x UTC

Posted from the Developer account into `#front › front-routine-mediagen` as
**four numbered posts** (message ids 4304, 4306, 4307, 4308), each under
5 000 characters.

**Four posts, deliberately.** The previous fire was one long payload and
arrived in Front's chatlog truncated mid-sentence; Front noticed and asked for
the tail, which was the right behaviour but cost a round trip. This time the
payload is pre-split under the limit, every post is numbered *"N of 4"*, and
post 4 ends with a sentinel line — so a truncated relay is detectable by the
receiver rather than by luck.

| post | carries |
|---|---|
| 1 of 4 | what the exercise is; **the suspension of the block-in-run rule, in the plan's own words**; what the notifier episode did *not* prove; **Front's one new instruction — go quiet until the bot posts**; mission shape and the new repository |
| 2 of 4 | base URL and the five model filenames verbatim; `pipeline.py` at `422eecc` as the graph source; the pinned settings; the prompt-cache trap; task 1 — the `submit`/`finish` split and the three extraction modes |
| 3 of 4 | the theme (cat, jump, idle), the still prompt, **the headroom finding from preflight**, the video prompt shape, the inherited defects, the dataset format |
| 4 of 4 | tasks 2, 3 and 4; the exact `comfynotify watch` line; **the 124-output decision as already made**; the optional two-ticket variant; the four MUST NOTs; the sentinel |

Three things in the fire came out of Step 0 rather than the plan:

- **The suspension sentence is quoted twice** — once in post 1 with an
  explanation of *why* the previous rule does not reach the notifier (a daemon
  is not a child process), and once in post 4 at the point of use. The plan
  warned that a relay would otherwise "correct" it back, and Front's own guide
  still says block.
- **The 124-output item is stated as a decision already taken**, not a choice
  to make, because the preflight fixed it. The fire also tells the run *why*
  its `finish` step must re-read `/history` anyway, so the belt and the braces
  are both explained rather than merely both required.
- **Headroom** is given as a selection criterion for the still, with the
  reason — a jump needs room above the character inside a fixed frame — and
  the seed choice explicitly left with the run.

Front was also told, in post 1, that a missing sentinel means it should ask
for the tail rather than relay a partial fire.
