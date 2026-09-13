# observer p2 — step 4: what the music is worth, and what the phase proved

## Musical usefulness, judged separately from Observer

**Who judged what.** The executing agent measured; it cannot hear. Every
mechanical property was checked from decoded PCM (`afconvert` to 16-bit WAV,
then Python's stdlib) rather than from ComfyUI's status strings: duration,
DC offset, clipping, near-silence, tail RMS, channel balance, truncation.
**Musical quality, lyric intelligibility, style adherence and structural
coherence are a human's judgement, and at the time of writing that listening
pass is outstanding.** No agent in this study wrote a quality verdict, and
the mechanical pass is not one — `subjects/yue2/tips.md` says so in its
header, in the usable/total tip, and in "Still open".

**The conditions, one axis each, two seeds each**, around a baseline chosen
to be judgeable (English mid-tempo indie rock, male lead, 120 BPM, a
verse+chorus lyric) rather than the pack's defaults:

| condition | seeds | outcome | wall time |
|---|---|---|---|
| baseline `cot: full`, `cfg_scale: 0.0` | 424242 ×2 runs | success | 137.1 s / 135.6 s |
| A `cot: off` | 424242, 909090 | success, mechanically clean | 180.0 s / 179.6 s |
| B `attention_backend: cudnn` | 424242, 909090 | **hard fail, no audio** | ~1.3 s each |
| C `cfg_scale: 3.0` | 424242, 909090 | success, mechanically clean | 236.2 s / 224.7 s |

**Usable/total by mechanical measure: 4/6**, and the two failures are one
deterministic hardware incompatibility rather than a quality problem. Five
distinct tracks exist (baseline, A×2, C×2) and were handed to the developer
for listening with a sheet naming each track's condition, seed and wall time.

**Deliverable tracks, honestly bounded.** The phase can deliver five 60-second
songs that are intact as signals. It cannot yet claim any of them is *usable
music*, and it does not. Nor can it claim stability from them: one style, one
lyric, three seeds in total, on one card.

## What the study established about YuE2 on this hardware

- **It runs on a Turing card at all** — the phase's open risk. Upstream
  requires "a BF16-capable NVIDIA GPU with 24 GB"; a Quadro RTX 8000 (SM 7.5)
  has no native BF16 path, and the pack offers the card with no gate or
  warning. One generation is ~136 s, resident footprint ~6.8 GB, not 24.
- **It is exactly reproducible, and "byte-identical" is the wrong word.** Two
  runs of one seed produced files differing by a single metadata byte (the
  `filename_prefix` ComfyUI embeds); decoded PCM hashes identically —
  2,879,936 samples, 48 kHz stereo. Verified twice, independently: by the
  executor and by the Omni Agent.
- **A resubmitted identical graph is answered by ComfyUI's node cache in
  58 ms** and would have produced a *fake* reproducibility result. The tell is
  wall time against the ~136 s floor; `POST /free` is the fix.
- **`attention_backend: cudnn` does not exist on this card** —
  `RuntimeError: No available kernel`, both seeds, ~1.3 s, deterministic. The
  pack recommends it as a 17 % speed-up.
- **`cfg_scale: 3.0` costs 65–74 % more wall time**, not the tooltip's
  "roughly doubles", and produced no measurable defect.
- **`cot: off` was not faster** (~180 s against the baseline's ~136 s) despite
  skipping the planning stage. Recorded as unexplained rather than
  rationalized, and left in "Still open".

The subject is written up the project's way: `subjects/yue2/summary.md`,
`tips.md` with one evidence line per tip, and an INDEX row in the same commit
(`autodev/mediagen@e4d8bf5`), under a new **Music** section — the index had
only checkpoints and workflow families. The raw run log stays in
`gentest-YuE2/` (`a8a496f`, `a2daf93`).

## Tests and environment

- pyagag **598** (592 + 6 for `agentchat intro` and the help note), agautolab
  **242**, agfront **157**, agobserver **63** — all green on the new lock.
- The `observe` guide change has no test: it is prompt text, and the evidence
  for it is `w6882`'s seven correct looks after `w6866`'s wrong one.
- `nctl drift`: **converged=46, error 0**, before and after the phase —
  unchanged from p1. No desired-state change was needed: nothing was
  deployed, only a guide and a shared CLI command.
- The real study *was* the integration proof, as the plan asked; no p1
  lifecycle test was repeated.

## Interventions, and two contract deviations worth recording

**Observer needed one fix and no rescue** — see `report3.md`. Nothing was
restarted by hand, no watch was reopened, no notification was faked.

**autolab deviated from its own published contract twice, in the same
serving.** Its introduction says *"I do not close a task until you say it is
done"*, and in task 4 it said it would report back before committing, then
committed and resolved the topic in that same serving, before any acceptance.
The result was good and I had approved the shape of the write-up, so nothing
was harmed — but the acceptance gate is what makes a requester's review
meaningful, and a task that closes itself removes it. Recorded here rather
than fixed: it is autolab's contract, not this episode's, and the evidence is
`workrun-task4-m6770` messages 6906–6909.

**The executor's own account of the Observer defect was wrong**, and it took
a supervisor with independent eyes to catch it: it wrote that the misjudged
watch was a "poll-timing false alarm", when the notification's own evidence
contradicted its verdict. Corrected in the run topic, and the corrected cause
is what `tips.md` now carries. This is the same lesson the realm already
learned about supervisors judging from relayed numbers — here it went the
other way, and the *worker's* summary was the thing that needed checking.

## Cost

Zero paid tokens for all waiting: 16 evaluations across five watches on the
host's local model. The paid runs were autolab's — planning plus eleven task
servings — and the Omni Agent's supervision. Requester runs consumed *during*
waiting: **zero**, evidenced by autolab's log.

## What remains

- **The human listening pass.** Five tracks are with the developer; the
  verdict will be appended to `subjects/yue2/tips.md` as its own entry and to
  this episode's phase report.
- The open questions the study itself names: why `cot: off` is not faster,
  `max_seconds` and `quantization` untested, `cudnn` untestable on this card,
  and only one style/lyric pair explored.
