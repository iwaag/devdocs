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
