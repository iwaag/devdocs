# Step 3 — the generation-test repository and one real matrix

Mission `M-4`, four tasks, both repositories pushed. 27 generations across
three matrices produced six evidence-backed tips and one clean null result.
The most valuable thing the step produced is not a tip: it is a live
reproduction of p7's backgrounded-work failure, caught and survived because
the runner the agent built one task earlier happened to be idempotent.

## Mission text (Developer, `#pj-mediagen › workplan-pixelart-matrix`, message 3096)

The asset requirement, stated by the Developer so that "usable" is judged
against something rather than against taste:

> A **64×64 side-view walking sprite of a four-legged animal**, for a 2D
> game. Single subject, centred, whole body inside the frame with a little
> margin. Flat, uniform background that keys out to transparency cleanly.
> Readable silhouette at 64×64. At most 32 colours after quantisation.
> Consistent outline.

The text carried the backend contract (`GetNewSession` → `GenerateText2Image`,
`model` required, `imageformat: PNG` because JPEG ringing survives a
nearest-neighbour downscale), the measured timings below, and two
instructions that turned out to matter:

> **One instruction for planning, not for the work.** When you split this
> into tasks, **copy the backend address, the exact model filename and the
> asset requirement into each task description verbatim.** The last mission's
> planning paraphrased "the backend is at `http://…`" down to "use the
> backend's IP address directly" — with no address in it — and the task then
> burned about twenty tool calls hunting the host down. A paraphrase drops
> literals. A generation matrix without an endpoint is not slower, it is
> nothing.

> **And one from the knowledge you already hold.** `summary.md` records a
> straight disagreement between two sources: the checkpoint's own model page
> says put `pixel art` early in the prompt, while the `pixel-art-xl` LoRA's
> card reports that including those words *hurts* results. Neither is
> evidence.

**The planning instruction worked.** Verified by grep before approving: all
four task files carry the literal address, the literal model filename and the
requirement. The superdirector's reply says "per your instruction", and it
added a constraint nobody asked for — task1's description tells itself to
stay to "a couple of images, not a full matrix — that is task 2". One
sentence in a mission text closed Step 2's friction 1.

## Developer-side measurement that sized the matrix

Before writing the mission the Developer probed the backend directly (the
permission-classifier stop this required is recorded at the end):

| | |
|---|---|
| first image, cold checkpoint | **27.2 s** |
| each subsequent image, 1024², 25 steps | **~8 s** |
| `imageformat: "PNG"` | accepted; default is 512² JPEG |

So a 30-cell matrix is ~4 minutes of GPU against a 20-minute task budget.
The plan's worry about matrix size does not bind, and the mission said so
explicitly — "you do not need to trim the matrix to fit, and you should not
pad it either". The agent's own smoke test reproduced the figures at
**27.45 s** cold / **9.23 s** warm.

## The repository

`gentest-pixelArtDiffusionXL/`, created with `autolab project init-repo
gentest-pixelArtDiffusionXL` plus a hand-written yaml — the pattern doc's
second route, chosen by the agent, recorded in `README_PROJECT.md` along
with the correction of the splice this episode introduced in Step 1
(`gentest-` folder "created by `init-localtest`", which was never true of
either route).

```
.gitignore              .local/ only
localtest.yaml          subject / backend (generic) / model / state
runner.py               230 lines
README.md               usage
report.md               raw run log
matrix.smoke.json       2 cells
matrix.sweep.json       20 cells
matrix.negprompt.json   10 cells (5 reused baselines + 5 new)
results.csv            27 rows
contact_sheet.png              4×5, labelled
contact_sheet_negprompt.png    5×2, baseline beside negshadow
.local/images, .local/post     ignored; raw 1024² PNGs and 64×64 outputs
```

**Runner usage**: `uv run --with pillow --with requests python runner.py
<matrix.json> [--base-url URL] [--force]`. The spec is JSON — a `defaults`
object merged into each `cells` entry, each cell a set of
`GenerateText2Image` params plus a `name`. The post step is inside the
runner, because "usable" is judged after it: nearest-neighbour downscale to
64×64, quantisation to ≤32 colours, and `bg_flatness` = mean per-channel
standard deviation of the border pixels.

**Unasked-for and decisive: the runner is idempotent.** A cell whose raw
image already exists is skipped, `results.csv` is upserted by name, a
human-filled `verdict` survives a re-run, and `--force` overrides. Nothing
in the mission asked for this. It is what made the task2 accident cost
almost nothing.

Three commits, all pushed:

```
f214abf Follow up negative-prompt axis: names the shadow, doesn't remove it
a7942e4 Run and judge the 20-cell sweep: subject dominates, CFG/style effects are subject-conditioned
80162c8 Add matrix runner for pixelArtDiffusionXL_spriteShaper generation test
```

## The matrices

**Backend used: the SwarmUI HTTP API directly**, neither ComfyUI surface.
The agent said so unprompted. No GPU contention with agforge, no timeouts,
no cell that failed to generate or post-process.

| matrix | cells | new generations |
|---|---|---|
| smoke | 2 | 2 |
| sweep — 5 animals × {tag, sentence} × CFG {4, 9}, steps 25, seed 12345, 1024² | 20 | 20 |
| negprompt — 5 animals × {baseline, +shadow/ground terms}, tag, CFG 4 | 10 | 5 |

The agent chose 5 animals rather than the plan's 3, reasoning that it wanted
more per-animal signal, and stayed inside budget doing it.

## What the matrices found

Verdicts, judged by the agent reading its own contact sheets, spot-checked by
the Developer against the same images: **19/20 usable with edits, 1/20 not
usable** in the sweep; the negprompt axis changed no verdict at all.

Two systemic defects, present in every one of the 20 cells regardless of
style or CFG: a **cast ground contact** (a shadow bar, or for deer a grass
plane), and **no white background anywhere** — hues vary between cells, stay
uniform within a cell, and are never the requested white.

The three axis answers, which only exist because the first judging pass was
sent back (see frictions):

- **CFG is subject-conditioned, not universal.** Dramatic on fox (CFG 4 thin
  shadow bar → CFG 9 full-width ground strip, both prompt styles), nil on
  dog, cat and horse.
- **Prompt style matters only for the horse** — sentence-style gives a
  thinner shadow and crisper outline. For fox, dog and cat it is a genuine
  negative result, which saves the next run an entire axis.
- **The subject dominates both swept axes.** All four deer cells carry a
  categorically different defect at every style and CFG, and the only
  outright failure is a deer (`deer_sentence_cfg9`, a full forest scene with
  two flanking trees). Pick the subject before tuning anything.

**The follow-up axis was a clean null**, and the agent chose it by the
Developer's own leverage argument — the ground contact is the one defect
blocking all 20 cells, so removing it would convert the whole matrix. Adding
`shadow, cast shadow, drop shadow, ground contact shadow, grass, ground,
floor, dirt, terrain` to the negative prompt removed nothing in any of the
five animals, and `bg_flatness` moved slightly *worse* in all five pairs.

The Developer corrected the wording of that result, and the correction is
the point. The agent first wrote "no visible change from baseline". The
contact sheet says otherwise: every pair is a different image — the fox's
background goes grey to lavender and its pose shifts, the horse's stance
changes, the cat is redrawn, the deer's grass strip narrows. So the negative
prompt was **not ignored**; it re-conditioned the generation and the shadow
came back anyway. "Negative prompting cannot remove it" and "negative
prompting had no effect" send the next run in opposite directions — the
first says stop asking the prompt and go post-process or change checkpoint,
the second invites someone to retry with better words. The tip carries the
first, plus the caveat that a changed negative prompt at a fixed seed is an
A/B on conditioning, not a single-variable comparison.

## `tips.md` as landed

`main/pixelArtDiffusionXL/tips.md`, commit `65c1c71`, pushed. Seven bullets
plus a "Still open" section; every tip carries a one-line evidence citation
(cell name plus the settings that mattered) and `2026-08-30`; **no score, no
grade, no level anywhere**, which is what the braindump asked for and what
the pattern doc now requires. The six substantive tips are the three axis
answers above, the negative-prompt null, the white-background finding, and:

- **The instrument has a blind spot.** `bg_flatness` ranked
  `deer_sentence_cfg9` — the forest scene, the matrix's one failure — as the
  **flattest of all 20 cells** (0.85, the lowest), because the outer pixel
  ring it samples happened to fall on a uniform patch. The agent found this
  in its own metric and wrote it as a tip. A finding about the tool belongs
  beside findings about the model, and a later run is told not to use
  `bg_flatness` as a sort key.

**The agent refused one of the Developer's suggested tips, correctly.** The
Developer's task text listed the `pixel art`-placement disagreement among
things to write. The agent checked, found that every prompt in both matrices
already put `pixel art` first or folded it into the opening clause, and that
no cell tested removing or moving it — so it moved the item to "Still open"
instead of writing a tip the evidence does not support, and said so. That is
harder than compliance and it is the behaviour that makes `tips.md` a
findings file rather than a summary.

Still open, as landed: the `pixel art` placement question; the
`pixel-art-xl` LoRA compared against this dedicated checkpoint; palette
size, sampler and step count (all held fixed, none swept); the unverified
`cat_sentence_cfg9` timing; and whether the ground contact responds to
anything other than negative-prompt wording.

`main/INDEX.md`'s `tips` column for this subject reads `2026-08-30`. Two
small labelled figures are committed in `main/pixelArtDiffusionXL/` —
`subject_dominates.png` puts the usable fox beside the failed forest deer,
and the finding is legible without reading a word.

**Publish-condition check** (Developer, independent):
`grep -rniE "agstudio|agpc|home\.arpa|/Users|localhost|<ipv4>|tailscale|7801|8188|autodev|gitea|gentest|README_PROJECT|localtest"` over `main/` — **one hit**, `INDEX.md`'s line
saying a `gentest-<subject>/` repository is created once testing starts. No
host fact and not this repository's real name, so nothing leaks; but it is
internal-workflow residue in a file `publish/` will copy to strangers who
have no idea what a gentest repository is. Raised for the first publish
review rather than fixed here.

## Frictions

1. **A task backgrounded its own work and ended — p7's failure, reproduced
   in a different agent and a different role.** task2 launched the 20-cell
   generation in the background and ended its run with "Waiting for the
   background matrix generation run to complete (20 cells, ~3 min GPU
   expected)". A finished run cannot wait, and the process died with it.
   **15 of 20 raw images survived on disk; `results.csv` still held only the
   two smoke rows**, so no timing or flatness was recorded for any of them.
   What saved it was the idempotence the agent had built into the runner one
   task earlier, unasked: re-running the same spec cost only the 5 missing
   cells and no GPU for the other 15. The Developer's re-serve said plainly
   that the sweep fits in the foreground with room to spare — 5 cells is
   under a minute, the full 20 from cold about three, against a 20-minute
   budget — so there was never a reason to background it. p7 fixed this for
   agfront with two guide lines; nothing carried the lesson to autolab's
   supercoder, and the guide it reads does not say it.
2. **The lost rows were reconstructed honestly.** Told not to invent wall
   times, the agent rebuilt the 14 missing rows from the spec and the saved
   images, computed `bg_flatness` from the post-processed PNGs it still had,
   and marked `wall_time_s` as `unmeasured` — **only 6 of 20 cells have a
   real measured time**, and the report says so. It also flagged
   `cat_sentence_cfg9`'s 0.58 s as too fast to be a real render, likely a
   server-side cache, and recorded it as unverified rather than as a fact.
3. **The verdict scale collapsed on the first judging pass and had to be
   sent back.** Nineteen cells reading `usable with edits` is not a result:
   the matrix existed to separate two axes and the uniform verdict hid both.
   The re-judge was cheap — no regeneration, no GPU, one run — and produced
   the three axis answers that are now the best content in `tips.md`. Worth
   generalising: **a verdict scale that every cell lands in the middle of is
   a sign the judging was done per-cell instead of per-axis.** A matrix
   mission should ask for the axis answers explicitly, not just the cell
   verdicts, or it gets a table nobody can act on.
4. **The Developer's acceptance of task4 never reached it, and created a
   phantom topic.** task4 finished, reported, marked `M-8` Done and resolved
   its own topic — all before the acceptance was posted. The post went to
   the literal name `workrun-task4-m-4`, which Zulip no longer has, so it
   **created a new topic of that name** and got the deterministic "not bound
   to any task" refusal. No agent ran and nothing cost money, and `work-m-4`
   now carries a phantom `workrun-task4-m-4` beside the real
   `✔ workrun-task4-m-4`. The cause is Developer-side tooling: the Omni
   Agent posts with `ZulipClient.send_to_channel`, which does **not** follow
   Zulip's `✔ ` resolve rename, while `agentchat send` does. The devenv notes
   say resolve-rename blindness is what lost p9's own completion report for
   26 minutes; this is the same hazard on the human's side of the wire. The
   Omni Agent should post through `agentchat` or a resolve-aware send.
5. **A close-out is a paid run per task.** Four tasks meant four acceptance
   runs on top of four work runs plus planning and approval. It is not large
   per run ($0.26–$0.31), but a four-task mission is eleven runs, not four.
6. **No GPU contention, no timeouts, no permission-classifier stops on any
   in-system run.** One classifier stop happened on the Omni Agent's own
   side; see below.

## Costs (run records — missions cannot quote their own)

| run | what | turns | duration | cost |
|---|---|---|---|---|
| `superdirector/run-0114` | planning (4 tasks) | 10 | 100.6 s | $0.2587 |
| `superdirector/run-0115` | approval turn | 5 | 20.2 s | $0.1317 |
| `supercoder/run-0121` | task1 repo + runner + smoke | 32 | 215.3 s | $0.5757 |
| `supercoder/run-0122` | task1 close-out (+ contact-sheet fix) | 17 | 88.4 s | $0.3117 |
| `supercoder/run-0123` | task2, backgrounded and lost | 21 | 214.1 s | $0.3370 |
| `supercoder/run-0124` | task2 retry, foreground | 41 | 270.6 s | $0.6296 |
| `supercoder/run-0125` | task2 re-judge (the axis answers) | 41 | 201.4 s | $0.6208 |
| `supercoder/run-0126` | task3 negative-prompt axis | 40 | 235.9 s | $0.6815 |
| `supercoder/run-0127` | task3 close-out | 16 | 66.4 s | $0.2628 |
| `supercoder/run-0128` | task4 tips.md + push | 33 | 177.1 s | $0.6521 |

**Step 3 total ≈ $4.46**, ≈ 26.5 min of model time, wall clock 19:36 →
20:10Z. All `claude_code` / `anthropic/claude-sonnet-5`, all `"outcome":
"done"`.

**GPU actually consumed: 27 generations, ≈ 4.6 minutes** — 2 smoke, 20
sweep, 5 negprompt. The two runs that were thrown away cost no GPU at all,
because of the runner's cell-skip.

## Deus Ex Machina interventions

- **Measured the backend's cold and warm generation times by hand** (three
  direct `GenerateText2Image` calls) to size the matrix before writing the
  mission — did the sizing measurement for agent autolab; **handoff
  candidate**: a first task could measure its own budget and say so, which
  would also have caught the JPEG default without the Developer knowing to
  look.
- **Permission-classifier stop, on the Omni Agent's own probe.** The first
  attempt at that measurement was refused by the classifier. Per
  `localrule.md` the work stopped and was reported rather than routed
  around; the user granted permission and it was re-run. No in-system agent
  was blocked at any point, and the classifier never touched the generation
  work itself, which runs inside autolab's own harness.
- Everything else — the repository, the runner, the post step, all three
  matrices, the judging and `tips.md` — was written by autolab runs. The
  Developer stated the requirement, started and accepted each task, sent one
  judging pass back, and corrected one tip's wording.
