# scheduled_routine p9 — phase report

`mediagen` exists: four surfaces, a knowledge layer of five documents, a
generation-test repository with a reusable runner, three matrices, 27
generations, and a `tips.md` carrying six evidence-backed findings and no
score of any kind. The braindump's bet — that the `studyarxiv` framework
transfers to media generation if you drop its level scale — held.

## 1. Does the study framework fit media generation?

**Yes, and it needed one deletion rather than a redesign.**

**Transferred as-is, untouched:**

- The `study` pattern itself. `main/` publish-ready, `publish/` a reviewed
  copy the agent never pushes — no adaptation needed, and the Developer's
  independent grep of `main/` found no host facts at any step.
- The repository-backed test folder: one bounded, resumable folder per
  subject on the ordinary `init-repo` naming path, its raw run log in
  `report.md`, its large artifacts in an ignored `.local/`. This is the part
  the braindump hoped would carry, and it carried without argument.
- `summary.md` / `INDEX.md` as the knowledge layer, including the discipline
  that made the arXiv summaries useful: **say what you could not find out.**
  The pixel-art summary's four-item "could not find out" list is what handed
  the matrix its questions.
- The requester-acceptance contract, the `workplan-`/`workrun-` topic
  machinery, the sequential sub-work gate, the devlog record. All worked
  unmodified on a project whose subject-matter is images.

**Renamed or bent — one thing, and only in naming:**

- `init-localtest` is paper-id-shaped. The agent chose the documented
  alternative (`init-repo gentest-<subject>` plus a hand-written yaml) and
  recorded the choice, exactly as the extended pattern doc says to. The
  hand-written yaml still calls itself `localtest.yaml` and still uses the
  local-test state vocabulary (`prepared` … `verified`), which fitted
  without strain. **Nothing was renamed in code and nothing needed to be.**
  The one place the two routes got spliced — `README_PROJECT.md` claiming a
  `gentest-` folder made by `init-localtest` — was written by the bootstrap
  run in Step 1 and corrected in Step 3, which is the pattern doc's own
  "record the choice" rule doing its job late rather than a defect in the
  design.

**Did `tips.md` hold up as the result format? Yes, on all three tests:**

- **Did the tips stay evidence-backed?** Every one carries a cell name or
  seed plus the settings that mattered, and a date. The strongest evidence
  that the format has teeth is that the agent **refused a tip the Developer
  asked for**: told to write up the `pixel art`-placement disagreement, it
  checked, found no cell had tested it, and moved the item to "Still open"
  rather than write an unsupported bullet. A format that makes an agent say
  "the evidence does not support this" is doing more than organising text.
- **Would a second run know where to append?** Yes — and this is no longer
  hypothetical. A later `routine-mediagen` fire (outside this phase, see
  below) read `tips.md`'s open questions and built a mission on them.
- **Do negatives land as tips?** Yes, and they are the most useful entries.
  "Sentence-style prompts do nothing for fox, dog or cat" and "the ground
  contact cannot be negative-prompted away" each close an axis. The
  free-form, append-only shape is what let a null result be written as a
  finding instead of a failure.

**The one thing the braindump did not anticipate: a tip about the
instrument.** `bg_flatness`, the runner's own automated proxy, ranked the
matrix's single failure — a full forest scene — as the flattest of all 20
cells, because it samples only the outer pixel ring. The agent found this in
its own metric and wrote it as a tip. Findings about the measuring tool sit
naturally beside findings about the model in this format, and would have had
nowhere to go in a level scale.

**No level scale was imported, and nothing missed it.** The pattern doc now
says so explicitly, next to the `L1`–`L4` table it does not apply, and the
`INDEX.md` column carries a date or `no` rather than a grade.

## 2. Did the matrix find conditions for asset-usable pixel art?

**Partly — and the honest headline is that it found the conditions under
which the checkpoint *cannot* meet the requirement.**

Against the stated requirement (64×64 side-view walking quadruped, single
centred subject, flat key-able background, ≤32 colours), **no cell was
drop-in usable**: 19 of 20 needed an edit and one failed outright. Two
defects are systemic and unavoidable through the axes swept — a cast ground
contact in all 20 cells, and no white background in any of them at any style
or CFG.

What it did establish positively:

- **Subject choice dominates prompt wording and CFG.** Deer carries a
  categorically different defect at every setting and produced the only
  failure; dog, cat and horse are stable across both swept axes. For asset
  work this is the actionable one: choose the subject first, then tune.
- **A flat, key-able background is reachable; a *specified colour* is not.**
  Backgrounds are uniform within a cell and never the requested white. Ask
  for flatness, key and recolour afterwards.
- **CFG's effect is subject-conditioned.** Dramatic on fox, nil on three of
  five animals. Any tip of the form "CFG 9 does X" without naming the
  subject would be wrong.

**The two findings agforge's image toolset would care about** — hand-off
material for a later phase, stated as information, not as a request:

1. **A pixel-art asset needs a post-processing stage that agforge does not
   have.** `agforge image generate` returns the raw SDXL render. For this
   asset class the render is not the asset: it needs nearest-neighbour
   downscale, palette quantisation, and background handling before anyone
   can judge whether it meets a requirement, let alone use it.
   `gentest-pixelArtDiffusionXL/runner.py` is 230 lines and already does
   generate → downscale → quantise → flatness-check → contact sheet, and
   `service/transform.py` is where that would live. The measurable claim
   behind this is that judging *before* the post step and judging *after* it
   give different answers.
2. **The ground contact is not a prompting problem, so no prompt-side
   toolset change fixes it.** Naming the shadow and the ground explicitly in
   the negative prompt re-conditioned the image — pose, background hue and
   the deer's grass width all moved — and the shadow returned in all five
   animals. A toolset that offers "add these negative terms" as the remedy
   would be offering something this evidence says does not work; the routes
   left are post-processing (crop or mask the ground band) or a different
   checkpoint/LoRA.

Not a finding, but worth recording: **the default output is 512² JPEG**, and
JPEG ringing survives a nearest-neighbour downscale. Any pixel-art path must
pass `imageformat: PNG` and explicit dimensions. That was learned by the
Developer probing the backend, not by an agent, which is itself a hand-off
note (below).

## 3. Costs, timings, GPU

| step | runs | model time | cost |
|---|---|---|---|
| 1 — pattern doc + bootstrap | 1 | 42.5 s | $0.19 |
| 2 — knowledge layer (`M-1`, 2 tasks) | 6 | 13.6 min | $2.29 |
| 3 — gentest repo + matrices (`M-4`, 4 tasks) | 10 | 26.5 min | $4.46 |
| **total** | **17** | **≈ 40.8 min** | **≈ $6.94** |

(Step 2's figure includes its final close-out run, which landed after
`report2.md` was written and is not in that report's $2.04 subtotal.)

**GPU actually consumed: 27 generations, ≈ 4.6 minutes** — 2 smoke, 20
sweep, 5 negative-prompt. Cold start ~27 s, warm ~8 s at 1024²/25 steps,
measured twice independently and agreeing. **The GPU was never the
constraint**: the model time above is 40 minutes against under 5 minutes of
generation. Two runs were thrown away entirely (a backgrounded task and a
collapsed judging pass) and cost **no GPU at all**, because the runner skips
cells whose images already exist.

The plan's concern that a matrix might not fit a 1200 s task was wrong by an
order of magnitude for images. It remains right for video, which is why no
video was generated.

## 4. Recommendations — partly overtaken by events

**A `routine-mediagen` standing text: it already exists, and p9's evidence
supports it.** The Developer wrote `#front › routine-mediagen` v1 and fired
it during the gap after Step 3, so this phase's recommendation is an
assessment rather than a proposal. What v1 does, and what p9 says about it:

- It reuses `gentest-pixelArtDiffusionXL/runner.py` explicitly rather than
  re-deriving it. p9 supports this strongly: the runner's idempotence is
  what absorbed the phase's worst accident, and rewriting it per subject
  would throw that away.
- It requires the asset requirement to be stated **before** generating. p9
  is the evidence for that rule — "usable" only became a judgeable word once
  the requirement existed.
- It says a fire finding only `blocked` questions should stop rather than
  invent work. Good; p9's own agent demonstrated the underlying virtue by
  refusing an unsupported tip.
- **What v1 should gain in v2, from this phase:** (a) *run generations in
  the foreground* — the single most expensive failure here was a task that
  backgrounded its own matrix and ended; (b) *report the axis answers, not
  just the cell verdicts* — the first judging pass put 19 of 20 cells in the
  same bucket and had to be sent back, and the axis answers are now the best
  content in `tips.md`; (c) *do not trust an automated proxy metric as a
  sort key* — p9's own metric ranked its worst cell best.

**Video and audio subjects in the next phase: not yet, and for a sharper
reason than the plan's.** Not because of budget — images used 5 minutes of
GPU against a 20-minute task budget — but because **the judging method has
not been shown to transfer.** Every finding here rests on a contact sheet of
still frames judged against a written requirement. There is no equivalent
artifact for a 5-second clip, and inventing one is a phase of its own. A
smoke generation to prove the API path is cheap and reasonable; a video
matrix without a judging method would produce files, not tips. `INDEX.md`
already catalogues the video and audio subjects, so nothing is lost by
waiting.

**A `publish/` review run: yes, and there is now exactly one small thing for
it to find.** `main/` is clean of host facts by independent grep, and
`INDEX.md` carries one line of internal-workflow residue — it tells the
reader a `gentest-<subject>/` repository is created once testing starts,
which means nothing to a stranger and names an internal arrangement. That is
a good first exercise for the gate: one real, non-leaking, genuinely
publish-blocking defect. The p8 question the gate raised then — whether
summary-only subjects may publish — arises here too, since four of five
subjects have `summary.md` and no `tips.md`, and mediagen's answer need not
match studyarxiv's.

**One thing p9 recommends that the plan did not ask about:** the mission text
instruction *"copy the literals into each task description verbatim"* should
become part of how missions are written, not a line the Developer remembers
to add. Step 2 lost ~20 tool calls because planning paraphrased an address
away; Step 3 added one sentence and all four task files carried the address,
the model filename and the requirement. That is a one-sentence fix with a
measured before and after, which is the cheapest kind of evidence-driven
guidance there is — it belongs in the planning guide.

## 5. What went wrong, and what it says

Three failures, all recovered, each worth more than a clean run would have
been:

1. **A task backgrounded its own work and ended** — p7's agfront failure,
   reproduced in autolab's supercoder. 15 of 20 images survived, all
   timings did not. p7 fixed this for agfront with two guide lines; nothing
   carried the lesson across, and the supercoder's guide does not say it.
   **The fix is a guide line in the same shape p7 used, not code.**
2. **The sequential sub-work gate cost 21 minutes of dead time** — not
   because the gate misbehaved (it answered in one line, immediately, naming
   what blocked it) but because the Developer watched the filesystem for a
   commit instead of the conversation for a reply. The lesson is the
   requester's: **acceptance is a scheduling dependency, not a courtesy.**
   Autolab's "a task is not closed until the requester agrees" contract is
   what holds the next task shut.
3. **The Developer's acceptance of the last task never reached it**, because
   the task had already resolved its own topic and the Omni Agent's posting
   helper does not follow Zulip's `✔ ` resolve rename — so it created a
   phantom topic instead. `agentchat send` follows the rename; the raw
   `ZulipClient.send_to_channel` the Omni Agent uses does not. The devenv
   notes record resolve-rename blindness losing a completion report for 26
   minutes on the agent side; this is the same hazard on the human's side.
   **The Omni Agent should post through `agentchat`.**

And one thing that went right and should be named: **the runner's
idempotence was not asked for.** The agent built cell-skipping and
verdict-preservation into task1 unprompted, and it is what made failure 1
cost almost nothing. Autonomy paid for itself within one task.

## 6. Deus Ex Machina interventions

- **Measured the backend's cold/warm generation times and the PNG parameter
  by hand**, before writing the Step 3 mission, to size the matrix — did the
  sizing measurement for agent autolab; **handoff candidate**: a first task
  could measure its own budget and report it, which would also have found
  the 512²-JPEG default without the Developer knowing to look for it.
- **One permission-classifier stop**, on that Omni-side probe. Work stopped
  and was reported rather than routed around, per `localrule.md` as it then
  stood; the user granted permission and it was re-run. **No in-system agent
  was ever blocked** — generation runs inside autolab's own harness, and the
  classifier never touched it.
- **Extended `agent/project_pattern.md`, created the Zulip channel, wrote the
  marker file, and posted every mission and acceptance as the Developer.**
  These are the requester's own role in the contract, not work taken off an
  agent, and are recorded here only because the plan asks for the full list.
- Everything inside `main/`, `publish/` and `gentest-pixelArtDiffusionXL/`
  was written by autolab runs. The Developer stated the asset requirement,
  sent one judging pass back, and corrected one tip's wording.

## 7. Scope note

Work on `mediagen` continued after Step 3 outside this phase — a
`routine-mediagen` standing request, a cross-subject `main/QUESTIONS.md`
queue, and a mission on skeletal versus sprite-sheet animation (`M-9`), all
driven through Front rather than by this phase. Section 4 assesses the
standing text because p9's evidence bears on it directly. Nothing else in
this report describes that work, and none of it is claimed here.
