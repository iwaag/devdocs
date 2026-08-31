# advance_mediagen_study p6 ex1 — Plan

An exercise on top of p6, not a new phase. Two things are new:

- **A new theme**: a **new source character** (a fresh still, not the p1
  dog) and a **non-walk action**. Two sprite-frame animation datasets come
  out of it.
- **The flow**: the one `agent_notifier/report.md` confirmed — submit the
  clip, `comfynotify watch <prompt_id>`, **end the run**, get called back
  by the Comfy Notifier bot's post, finish the dataset on the second
  serving. p6 fire 1 established the opposite rule ("block in-run, never
  background a clip") after three orphanings; ex1 deliberately suspends it,
  because the notifier is what makes ending the run safe. **The fire text
  must say so in as many words** — the guide, the previous topic and the
  standing text all still say "block", and a relay will otherwise "correct"
  the instruction back.

Experimental, non-public environment; destructive phase, no backward
compatibility. **MUST NOT** lines are the only prohibitions; everything else
is advice the implementer may override with a stated reason.

## What is already true (do not re-derive)

- `gentest-videoLoopPipeline/pipeline.py` is the proven one-command chain:
  six-node SDXL still (1024², steps 25, cfg 4, `euler`/`normal`) → 64 NN
  thumb → pad to 640×640 → free → MiniMax H3 first+last (start = end,
  `length` 124, 6 turbo steps, no `ResolutionSelector`) → free → loop
  analysis (fixed period picker, `stride = round(period/8)`) → keyed 64×64
  8-frame sheet. Both stages are **byte-deterministic at a fixed seed** on
  this backend; frees cost ~10 s of ~450 s. Prompts, checkpoint and model
  filenames are constants at the top of `pipeline.py` / `onecell.py`; the
  base URL is `onecell.DEFAULT_BASE_URL`. Copy literals from there into the
  fire text verbatim; none of them belong in this repository.
- The still prompt is tag-style (`pixel art, a dog walking, side view, full
  body, game sprite, flat solid white background, 32 bit, …`); the video
  prompt is a sentence (`2D pixel art video game sprite, a dog running in
  place, side view, seamless walk cycle loop, flat solid background, no
  camera motion.`). Describe the **motion** in the video prompt, the
  **character** in the still prompt.
- Known inherited defects: background comes out lavender, not white; a cast
  shadow shares the outline's palette entry; faint background "ghost"
  structure survives the key at 640². None is fixed; none blocks this.
- `comfynotify watch <prompt_id> [--to ch/topic] [--mention name] [--note
  text] [--timeout s] [--comfyui url]` writes a ticket and returns; the
  daemon (`com.agdev.comfy-notifier`) posts once per terminal state
  (`success`/`error`/`timeout`/`lost`/`unreachable`) as the **Comfy
  Notifier** bot: headline `comfy <state> <id[:8]> in <wall>s` plus a
  fenced JSON record with `outputs[]` (`filename`, `subfolder`, `type`,
  `url`), `vram_free`, `note`. `--to` defaults to `AGENTCHAT_HOME`, i.e. the
  `workrun-` topic itself. It is on PATH in autolab's and agforge's runs;
  both guides carry the "submit, watch, record what is pending, finish"
  paragraph.
- **What the notifier episode did *not* prove**: a second autolab serving on
  a `workrun-` topic triggered by the bot's post. Step 4 hit the short-job
  race (still finished before the first run exited); step 5 used a scratch
  topic. ex1 is the first real test of that leg. A serving is resumed by
  nothing but a post into a topic the agent watches, and the bot's post is
  one — that is the mechanism, and it is untested end to end.
- Zulip caps a message around 10 000 characters. p6's video graph saves
  **124 PNG frames** through `SaveImage`; a `success` record listing 124
  outputs with URLs is ~18 KB. Whether the notifier truncates or the post
  fails is unknown — preflight item below.

## The theme

Defaults, so nobody is blocked; the run may change any of them with a
reason, and the Developer may override in the fire.

- **Character**: a **cat** (p1: cat/fox/dog/horse generate cleanly, deer
  does not). Still prompt in the p1 tag style with the pose clause replaced
  by a **neutral standing side view, all four paws on the ground** — the
  start = end frame of every action below. Stills cost ~10 s: render up to
  ~8 seeds, **open them**, keep the one that is side-on, whole-body, centred
  on a flat background, and record the seed. That is the "new source image"
  half of the exercise and it is meant to be judged by eye.
- **Dataset A — jump in place**: crouch → airborne → land → back to the
  stance. One-shot, non-periodic, returns to start by construction.
- **Dataset B — idle**: breathing, tail sway, a head turn; subtle,
  quasi-periodic. Deliberately the opposite motion profile from A.
- Alternatives if either fails to read: pounce/attack (A), sit-down-and-up
  (B). Two datasets, one character, one still: the comparison stays clean
  and the still is generated once.
- Video prompt shape: *"2D pixel art video game sprite, a cat jumping
  straight up in place and landing back in the same standing pose, side
  view, flat solid background, no camera motion."* Same for idle with the
  motion swapped. Keep `length` 124, 640×640, seed fixed per dataset.

### Extraction is different for a non-gait

p6's `stride = round(period/8)` assumes a repeating gait. A jump has no
period; an idle may have a weak one. Give `pipeline.py` an extraction mode:

- `cycle` — p6's rule, for gaits.
- `whole` — 8 frames at uniform stride over the whole clip, last frame
  dropped (it duplicates the first).
- `motion` — cumulative adjacent-frame distance over the clip, pick the
  8 frames at equal motion quantiles. For a jump this spends frames on the
  crouch, flight and landing instead of on 3 s of standing still.

Default `motion` for A, try `cycle` then fall back to `motion` for B, and
say which was used in the dataset's `meta.json`. As always: the picture
decides, not the closure ratio — that metric was built for loops and will
rank a static sheet well.

## Dataset format

Per dataset, in the test repository (small PNGs are fine to commit):

```
datasets/<character>_<action>/
  frames/00.png … 07.png     64×64 RGBA, one shared ≤32-colour palette
  sheet.png                  512×64 horizontal strip
  preview.gif                8 frames at 12 fps, looping (for humans)
  contact_4x.png             the sheet at 4× nearest-neighbour
  meta.json                  character, action, still seed, video seed,
                             both prompts, prompt_id, extraction mode,
                             source frame indices, palette size,
                             suggested fps, loop=true, closure numbers
```

Copy `contact_4x.png` and `preview.gif` of each into this folder for the
report.

## Step 0 — Omni Agent preflight

1. `agentchat read front front-routine-mediagen` — no Developer post you did
   not write. (p6's mission M-39 is closed, all four tasks resolved; nothing
   else is live in the project.)
2. Card ≥ 40 GiB free, queues empty; `launchctl list | grep comfy-notifier`
   running, tail of `comfynotify/.local/out/notifier.log` sane.
3. **The 124-output post.** Write a ticket by hand for p6's last successful
   video `prompt_id` (still in `/history` unless ComfyUI restarted) into a
   scratch topic and see what the bot posts. If it is truncated or refused,
   pick one before firing: (a) have the notifier cap `outputs` at N entries
   plus a count, or (b) tell the run to keep `SaveImage` but have its
   `finish` step re-read `/history/<prompt_id>` itself — the callback only
   needs the id. (b) needs no notifier change; say which in the fire.
4. Optional: render the cat still by hand once to check the prompt gives a
   side view. Ten seconds; if you do it, the seed goes into the fire and
   the run still owns the choice.

## Fire — one mission, ~4 tasks

Fire `routine-mediagen` from `#front › front-routine-mediagen` (context in
the firing post). Subject: **two sprite animation datasets — new character,
jump and idle — through the notifier callback flow**. New repository
(`gentest-videoLoopPipeline` is `verified`), e.g. `gentest-actionDatasets`;
copy `pipeline.py` and its helpers, do not import across repositories.

1. **Tooling, no GPU beyond stills.** Parametrise the copy: `--character`,
   `--action-prompt`, `--still <png>` (reuse a chosen still), `--label`,
   `--extract cycle|whole|motion`, and split the run into
   `pipeline.py submit …` (still if needed → pad → free → submit video →
   print `prompt_id`, write `pending.json`) and `pipeline.py finish
   <prompt_id>` (fetch frames → free → analyse → extract → dataset folder).
   Render the still candidates, pick one by eye, commit the seed.
2. **Dataset A (jump)** — one task, two servings: `submit`, then
   `comfynotify watch <prompt_id> --note "A jump"`, then a report saying
   *what is pending and that a post is required to resume*, then **end the
   run**. On the bot's callback: `finish`, open the sheet, write `meta.json`,
   commit. The first serving must not close the task in Plane.
3. **Dataset B (idle)** — same shape. If task 2's callback did **not**
   re-serve the topic within a few minutes, the Developer posts one line to
   resume it and that fact goes in the report; task 3 then runs the same way
   again rather than falling back to blocking — the point is to measure the
   mechanism twice.
4. **Improve and publish**: tips (evidence line + date) in
   `main/subjects/videoFrameExtraction/tips.md` — non-gait extraction,
   what the video model did with "jump" and "idle" from a single still,
   whether start = end still closed the loop for a one-shot action; INDEX
   row; `git grep --untracked` self-check; push `main` and the test
   repository.

Optional variant if tasks 2 and 3 both go cleanly: submit A and B back to
back with two tickets in one task (ComfyUI queues them; the second callback
lands ~7 min after the first; `--timeout` must cover ~15 min) to see two
callbacks into one topic.

Things to put in the fire text verbatim: the base URL and model filenames
from `onecell.py`; the still and video graphs (or "your copy of
`pipeline.py` as committed at `422eecc`, which already has `--positive-prompt`"); the character/action defaults
and the extraction rule above; the exact `comfynotify` line; **"for this
mission, end the run after `comfynotify watch` — the previous rule to block
in-run is suspended; the notifier's post in this topic is what resumes
you"**; and, unchanged, no `low_vram`, no CPU-CLIP, one clip per task.

## Front

Relay, do not verify. One new thing to ask of it: **after autolab's first
serving ends with "pending", stay quiet until the bot posts** — a Front
post into the `workrun-` topic before the callback would itself resume
autolab early and spoil the measurement. If the bot's post does not produce
a second serving, Front reports that; the Developer decides the nudge.

## Report

`report1.md` for the preflight and the fire; `report.md`: the two datasets
(contact sheets and GIFs beside this file, judged by eye first); for each
callback the timeline — job end → bot post → second serving start — and
whether the second serving happened unprompted; cost of the two servings
against p6's one blocking run (`agautolab/.local/agent/<role>/run-NNNN.json`,
`agfront/.local/agent/front/`); GPU minutes; what the motion model did with a
non-gait prompt from a standing still; extraction mode used per dataset;
the 124-output decision; Deus Ex Machina lines; and what the notifier
episode should change, if anything, now that a real workrun has used it.

**MUST NOT**: host literals, internal repository names or agent names into
`main/` or this repository; push `publish/`; re-apply `low_vram`/CPU-CLIP;
resume autolab by a human or Front post *before* the bot has had its chance
(a few minutes after job end) — the callback is the thing under test.
