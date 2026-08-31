# advance_mediagen_study p6 ex1 — step report 2

Plan: `plan.md`. Preflight and the fire: `report1.md`. This file follows the
mission task by task as it runs.

Mission **M-46**, *"Two sprite-frame action datasets (cat: jump, idle) through
the notifier callback flow"*, four sub-works, opened 2026-08-31 12:23 UTC.

## The relay, and a race the fire created

The fire went out as four numbered posts inside about four seconds. Both
listeners in the chain snapshot their chatlog when a run *starts*, so both
agents were served a partial payload:

| | what it had | what it did |
|---|---|---|
| **Front**, 12:16:52 | post 1 only | checked for the sentinel, found none, **refused to relay**, asked for the tail |
| **autolab**, 12:20:59 | posts 1–3 | **held off writing `plan.md`** rather than guessing how tasks 2–4 divide, and said so |

**Both refusals are the right behaviour and both were unprompted at the point
of decision.** Front had been told what to do about a missing sentinel;
autolab had not been told anything of the kind, and reasoned it out from the
posts being numbered "N of 4".

**The race is the fire's fault, not theirs.** p6's fire was one long payload
and arrived truncated; the fix here — pre-split under the length cap, numbered
parts, an explicit end sentinel — removed that failure and introduced a new
one in the same place. The general rule, which is what should survive:

> **A multi-post payload must be posted tail-first, or announced only once
> every part is up.** A receiver that is woken by the first part will be
> served a snapshot that does not contain the rest.

One Developer post fixed it (*"all four are already in this topic"*), and
autolab's listener recovered on its own — its log shows `reprocessing …: human
posts arrived during the run` twice, which is exactly the mechanism that
should exist for this and did.

Cost of the race: one extra Front run and roughly six minutes. Cheap, and it
bought a rule.

## Planning

autolab wrote `plan.md` and `task1.md`–`task4.md`, created M-46 with sub-works
M-47…M-50, opened `#work-m-46` with four `workrun-` topics, and **did not
write `start.flag`** — it held the mission in planning because nothing in the
log had greenlit it. That is its own contract with the requester and it kept
it without being reminded.

**I read `plan.md` and `task2.md` off disk rather than trusting the relay's
summary of them.** Everything survived two hops intact: the suspension
sentence verbatim, the headroom clause, the three extraction modes with
`motion` defaulted for the jump, the `outputs_total` note, and all four MUST
NOTs. `task2.md` also carries the instruction that a missing callback is to be
*reported*, not worked around.

Greenlit at 12:26.

## Task 1 — tooling and the still

**Done, ~8 minutes, no GPU beyond stills.**

- `onecell.py`, `analyze_loop.py`, `extract_sheet.py` copied from
  `gentest-videoLoopPipeline@422eecc` and **proved unmodified by three empty
  diffs against that commit** — not asserted, shown.
- `pipeline.py` rewritten into `submit` / `finish`. `submit` renders or reuses
  the still, pads, frees, submits, prints the `prompt_id` and writes
  `.local/pending/<prompt_id>.json`; it does not wait. `finish <prompt_id>`
  reads that record **by prompt id** and re-reads `/history/<prompt_id>`
  itself.
- Three extraction modes added. `whole` and `motion` were checked against a
  synthetic 20-frame directory — 8 distinct, monotonic, well-spread indices
  from each. Each mode returns a *reason* alongside its frames, and `finish`
  writes `extract_mode`, `extract_mode_reason` and `extract_mode_detail` into
  `meta.json`. The plan asked for the mode to be recorded; the run recorded
  why as well.

### The still: 8 seeds, 7 rejected

| seed | verdict |
|---|---|
| 11111 | reject — three-quarter body despite "side view" in the prompt |
| 22222 | reject — more front-on; ~7 % headroom |
| 33333 | reject — torso facing viewer; ~8 % headroom |
| 44444 | reject — good side pose, but a green grass patch breaks the flat background |
| 55555 | reject — sitting, not standing |
| 66666 | reject — front-facing; head ~3 % from the top edge |
| 77777 | reject — sitting, front-facing, off-centre |
| **88888** | **kept** — side profile, flat ground, standing, ~10 % headroom |

**I opened the chosen still myself and would have picked the same one.**
Genuine side profile, all four paws down, whole body inside the frame, flat
grey ground, real space above the head.

Two things worth keeping from how the choice was made:

- **The rejection reasons are specific and varied** — pose, background,
  stance, framing. Four different failure modes across seven seeds at a fixed
  prompt. My own preflight hit the same rate (one of three facing the viewer),
  so "render several and open them" is load-bearing, not ceremony.
- **Headroom was measured as a cross-check on top of looking, not instead of
  it.** The run says the eye decided and the pixel measurement (topmost
  non-background row, corner-colour threshold) only separated the close calls,
  66666 against 88888. That is the right order, and it is the order p6 had to
  be told twice.

The still was re-checked **after** padding to 640×640, not only before — the
headroom that matters is the headroom in the clip's frame, which is not the
same number.

### One honest gap, stated by the run itself

**`cycle` was not exercised this task.** Its code path is unchanged from
`422eecc` and previously verified there, but it has not run in this
repository. The run said so plainly rather than letting "three modes added"
imply three modes tested. Task 3's idle is therefore the first real `cycle`
run here and its result must not be treated as pre-verified — relayed back to
autolab as an explicit instruction.

## Task 2 — dataset A (jump): the experiment, and it passed

**The callback leg works end to end, unprompted, in about one second.**

All three times are taken from the backend and the listener logs, not from
anybody's report of them:

| event | source | time (UTC) |
|---|---|---|
| video submitted | ComfyUI `execution_start` | 12:46:16.04 |
| serving one ended, "pending" | autolab's post | 12:46:34 |
| **job finished** | ComfyUI `execution_success` | **12:53:31.15** |
| **Comfy Notifier bot posts** | Zulip message 4392 | **12:53:32** |
| **autolab's second serving starts** | autolab listener log | **12:53:32** |

**No human post, no Front post, no nudge.** The clip ran 435 s; the run had
been over for seven minutes; the bot's post alone brought autolab back and it
finished the dataset.

That is the leg the notifier episode never proved — its step 4 hit a race and
its step 5 used a scratch topic. It is proved now, on a real `workrun-` topic,
inside a real mission.

Two details worth keeping:

- **The gap from job end to work resumed is ~1 s**, of which the daemon's own
  5-second poll interval is the only real latency budget. The mechanism is not
  merely functional, it is fast enough that "block in-run" buys nothing.
- **Front stayed silent**, as instructed, for the whole seven minutes. Asked
  to *not act* rather than to keep a run alive, it did, and it was woken by
  autolab's own reply. The p6 stall was a supervisor promising to watch and
  then ending its run; this is the same shape of moment with the opposite
  outcome, because nobody was asked to watch.

### What the model did with "jump" from a standing still

`cat_jump_contact_4x.png` beside this file. **I opened it before reading any
number.**

Frame 1 is the standing stance. Frame 2 is a crouch, haunches compressed.
Frames 3–6 rear up and leave the ground — body stretched vertical, all paws
off, the cast shadow detached below. Frames 7–8 land and settle, with a dust
puff the model invented on its own.

**A one-shot, non-periodic action came out of a single standing still and a
sentence.** The video model was never given a crouch or an airborne frame to
work from; `start = end` and a motion description were enough.

**`motion` extraction did what it was added to do.** Its chosen indices —
0, 24, 34, 40, 59, 72, 76, 89 of 124 — cluster where the movement is and skip
the standing seconds. Uniform stride over the same clip would have spent
several of eight frames on a stationary cat.

Two honest qualifications, neither of which the run hid:

- **`loop: true` rests on the clip's construction, not on the sheet.** The
  quoted closure ratio (0.2598) is measured over the **full 124-frame clip**,
  where start = end by construction. The delivered 8-frame sheet's own last
  frame is a settle, not the frame-1 stance. `meta.json` says exactly this,
  in its own words, and says the picture is what decided.
- **`suggested_fps: 10` is labelled a judgment call, not a measurement** — the
  8 frames are motion quantiles, not equal-time samples, so the source clip's
  24 fps does not carry over. The run said so rather than quoting 24 and
  hoping.

### A standing fact that turned out to be false

The fire told autolab *"`comfynotify` is on your PATH"*, on the authority of
the notifier episode's own report and the local environment notes. **It was
not.** `which comfynotify` returned nothing in the run's shell, and autolab
found the binary at its absolute path in `comfynotify/.venv/bin/` instead.

`AGFORGE_COMFYUI_URL` was also unset, so the first `watch` invocation — the
exact line the fire gave, with no `--comfyui` — **failed**. autolab retried
with the variable set inline and the ticket was written 11 s later.

`agautolab.instance.extra_environment` does put that directory on `PATH`, and
the directory exists and contains the binary, so the grant is written and the
value is lost somewhere between there and the agent's shell. **I am recording
the fact and not the mechanism** — p5's lesson was that a confident mechanism
invented at this point is what the next phase pays for.

The run recovered in two attempts and about eleven seconds, which is the
system working. But it recovered from **an Unexplained Chainsaw**: a tool it
was told it had, at a name that did not resolve. Handoff candidate.
