# Step 2 — knowledge: `summary.md` files and the index

One mission, `M-1`, two tasks. `main/` went from an unborn branch to
`0ab93b3` pushed: an index of fourteen curated subjects and four subject
summaries, the pixel-art one written to the depth Step 3 needs. Two real
frictions, both about information the mission text had and the task did not.

## Mission text (Developer, `#pj-mediagen › workplan-knowledge-layer`, message 3060)

> Mission: build the first knowledge layer in `main/`. One or two tasks, your call.
>
> Read `autolab doc patterns` first. Its "Repository-backed generation tests" section defines `main/<subject>/tips.md` and says there is **no level scale** for generation tests — the `L1`–`L4` table above it belongs to papers and does not apply here.
>
> The live backend to inventory is a SwarmUI instance at `http://<ipv4>:7801` (use the IP; the `.local` name stalls ~5 s on an unanswered AAAA lookup). Its API needs a session first: `POST /API/GetNewSession` with `{}` returns a `session_id`, then `POST /API/ListModels` with `{"session_id": "…", "path": "", "depth": 3, "subtype": "Stable-Diffusion"}` lists the checkpoints; `subtype` also takes `LoRA`. **Generate nothing in this mission** — that is the next one, and the GPU is shared with agforge's production asset runs.
>
> Do, in `main/`:
>
> 1. Write `main/INDEX.md`: one row per catalogued **subject** — a checkpoint, a LoRA set, or a workflow family — with its architecture, what it is for, and a `tips` column that says `no` or the date of the most recent tip in `main/<subject>/tips.md`. Everything is `no` today. Document that column in one line inside the file, so a reader who does not have the autolab checkout understands it. **Catalogue general-purpose subjects only**: that server's library also holds adult-oriented checkpoints and character LoRAs, so leave those out and say in one sentence that the index is a curated subset rather than the server's full listing. You decide the cut.
>
> 2. Write `main/pixelArtDiffusionXL/summary.md` first and thoroughly — it is the project's first subject, and the next mission runs a generation matrix against it. Cover: what the checkpoint is (origin, author, version, base architecture, where it is published) with the URLs you read cited; the `pixel-art-xl` LoRA as a sibling approach to the same problem; the sampler / steps / CFG / resolution / prompt vocabulary its model page and public SDXL pixel-art practice recommend; the post-processing pipeline people actually use to turn a 1024² SDXL image into a usable pixel-art asset (nearest-neighbour downscale to 32–128 px, palette quantisation, background removal) and why raw SDXL output needs it at all; and a closing list of the first tests worth running. Paraphrase — no long quotations. Say plainly what you could not find out.
>
> 3. If budget allows, brief summaries (a few paragraphs each) for two or three more general subjects, so the index is not a one-row table. `perfectdeliberate_XL`, `novaAnimeXL_ilV5b` and `flux1-kontext-dev` are reasonable picks. Optional — skip rather than pad.
>
> 4. Commit and push `main`. That clone is on an unborn branch, so the first push is `git push -u origin main`.
>
> Publish conditions apply from day one: `main/` is written publish-ready, so **no host facts** anywhere in it — no hostname, IP, port, filesystem path, or internal repository name. Write "a local SwarmUI instance" and name the model files; nothing more.
>
> Report in the task topic: `main/INDEX.md` as landed, the pixel-art `summary.md` in full, and what you could not find out.

(The address is redacted in this report only; the Zulip message carries the
literal IPv4. That distinction is the whole of friction 1 below.)

Per p8's finding, the "quote your own harness result JSON" line was **not**
included — a mission cannot satisfy it. The numbers below come from the run
records.

## Plan and runs

`superdirector` answered at 18:54 with a two-task split (`M-1`; sub-works
`M-2` inventory+index, `M-3` summaries+push), opened `work-m-1` and both
`workrun-` topics, and asked whether the split was right. Approved, and
`workrun-task1-m-1` posted into to start it.

| run | what | turns | duration | cost |
|---|---|---|---|---|
| `superdirector/run-0112` | planning | 11 | 70.3 s | $0.1739 |
| `superdirector/run-0113` | approval turn | 6 | 20.0 s | $0.1039 |
| `supercoder/run-0117` | task1 (inventory + INDEX) | 38 | 384.0 s | $0.7719 |
| `supercoder/run-0118` | task1 close-out re-serve | 6 | 41.9 s | $0.1491 |
| `supercoder/run-0119` | task2 (summaries + push) | 37 | 219.4 s | $0.8400 |

Mission total **≈ $2.04**, ≈ 12.3 min of model time. Wall clock 18:53 →
19:35Z, of which **21 minutes were dead time** — see friction 2. All five
runs `claude_code` / `anthropic/claude-sonnet-5`, all `"outcome": "done"`.

## `main/INDEX.md` as landed

Fourteen rows in three sections, plus a stated cut. Header:

> This indexes the generation subjects (checkpoints, LoRA sets, workflow
> families) catalogued for this project. Each row is one subject; a
> `gentest-<subject>/` repository is created for a subject only once
> generation testing on it actually starts — none exist yet.
>
> This is a **curated subset** of what the backend holds, not its full
> listing: adult-oriented checkpoints/LoRAs and character LoRAs are left out
> by design (see the note at the end of this file for the cut and why).
>
> The `tips` column says whether `main/<subject>/tips.md` has any findings
> yet: `no` means no generation has been run against that subject, otherwise
> it holds the date of the most recent tip.

**Checkpoints (9):** Pixel Art Diffusion XL (SDXL), PerfectDeliberate XL
(SDXL), FLUX.1 Kontext [dev] (Flux, instruction-guided editing), LTX-2,
Wan 2.2 T2V 14B, Wan 2.2 I2V 14B, OmniGen2, ACE-Step 1.5 (text-to-music),
MiniMax H3. **LoRA sets (5+1):** Pixel Art XL LoRA, LTX-2 distilled speed,
LTX-2 camera control, MiniMax H3 turbo, Wan 2.2 I2V and T2V LightX2V 4-step.
Every `tips` cell is `no`.

**Workflow families: none** — and the file says why, which is the honest
answer rather than an empty section. The run probed `ListComfyWorkflows`,
`ListWorkflows`, `ListSavedWorkflows` and `ListModels subtype=wildcards`;
all returned `bad_route` / `Invalid sub-type`. So this instance's API
exposes no workflow catalogue, and a workflow-family subject will have to be
described from the API-format JSON files rather than discovered from the
server.

**The cut**, in the file's own words: kept "general-purpose generation
engines or generic utility LoRAs (style, speed-up, camera control) with no
adult framing"; excluded the anime/furry Pony/Illustrious/NoobAI finetune
cluster (one explicitly NSFW-named, the rest in the same family and
"excluded for the same reason rather than judged file by file"), the
unlabeled Wan 2.2 I2V finetunes sitting beside an explicitly NSFW one, every
character LoRA (Digimon, Pokémon, dragons…), and the two LTX-2 VAE files as
supporting components rather than subjects. Of 32 checkpoints and 25 LoRAs
offered, 14 subjects were catalogued. The reasoning is stated per group, so
a later run can disagree with the cut without re-deriving it.

## `main/pixelArtDiffusionXL/summary.md` — 169 lines, the full text

Sections: *What the checkpoint is* / *Sibling approach: the `pixel-art-xl`
LoRA* / *Sampler, steps, CFG, resolution, prompt vocabulary* / *Post-processing*
/ *First tests worth running* / *What could not be found out*. Rather than
transcribe all 169 lines, what it establishes:

- **Identity.** Pixel Art Diffusion XL by the Civitai creator *Yamer*, built
  on SDXL 1.0, [Civitai model 277680], mirrored on CivArchive. The file this
  project holds is the **"Sprite Shaper"** version (published 2024-02-27) —
  as against the line's other version, "Pixel World" — described as more
  detailed and blockier, closer to a game-sprite look. **VAE is baked in**,
  so no external VAE file. License CreativeML Open RAIL++-M with an addendum
  requiring written consent to redistribute or to sell generated images.
- **The sibling.** `nerijs/pixel-art-xl` v1.1 (HuggingFace + Civitai,
  CreativeML OpenRAIL-M, published 2023-08-07), which puts the same style on
  a stock SDXL checkpoint instead of replacing it. The summary states the
  trade plainly — checkpoint bakes the style in, LoRA generalises across
  checkpoints at the cost of a component to load and a strength to tune —
  and says to test both against the same prompts.
- **Settings the model page gives**: 1024×1024 or other SDXL-friendly
  ratios; **30–50 steps**; **CFG 4–12**; optional upscale
  (4x_foolhardy_Remacri / 4xUltraSharp) at ≤0.25 denoise. Prompt vocabulary:
  `pixel art` early, optional `16 bit` / `32 bit` / `64 bit`, short prompts,
  no `realistic`/`photography` near the start.
- **The tension worth the whole document.** The checkpoint's page says put
  `pixel art` early in the prompt. The LoRA's own card says including the
  words "pixel art" **is reported to hurt results**. Both are recorded, as
  hearsay, unresolved. This is precisely a `tips.md` question and Step 3's
  first prompt-wording axis is now a real disagreement to settle rather than
  a guess.
- **Post-processing**, three stages with the reason each exists: raw SDXL
  "only *looks* like pixel art at a glance — it has soft anti-aliased edges,
  far more distinct colors than any real pixel-art palette, and no true 1:1
  pixel grid". (1) nearest-neighbour downscale, never bilinear/bicubic;
  ~8× as a starting divisor from the LoRA's card, 32–64 px common for
  sprites; `pixeldetector` / ComfyUI-PixelArt-Detector detect the grid the
  model actually drew rather than assuming a divisor. (2) palette
  quantisation — ~8 colours for strict retro, 16–32 for typical game art,
  "beyond ~32 colors the pixel-art look starts to disappear", optional
  dithering. (3) background removal via `rembg` or BiRefNet for a
  transparency-ready PNG.
- **Six first tests**, including the two that Step 3 should actually run:
  trigger-phrase on/off, and checkpoint vs LoRA on identical prompts.

**What it says it could not find out** — four items, all specific:

1. No sampler name (Euler a vs DPM++ 2M Karras…) is stated on the model page.
2. No non-LCM CFG/step recommendation for `pixel-art-xl`; the card's only
   numeric recipe (8 steps, CFG 1.5) is explicitly an LCM speed setup and is
   flagged as not a general baseline.
3. No official downscale factor or target resolution from the checkpoint's
   own author — the 8× divisor comes from the sibling LoRA's card and is
   offered as a starting point, not a documented recommendation.
4. The redistribution/commercial addendum was summarised from the model page,
   not verified against the full licence text.

That list is the best thing in the file. Each item names its source, says why
the nearest available number does not transfer, and is answerable by a matrix
— which is what a `summary.md` is supposed to hand to a `tips.md`.

## The three optional summaries — all three written

- **`perfectdeliberate_XL`** (27 lines): the PerfectDeliberate line began on
  SD 1.5; despite the "XL" name its current XL versions are built on the
  **Illustrious** family rather than plain SDXL 1.0. Could not confirm which
  version tag this file is, nor any sampler/CFG for the XL line.
- **`novaAnimeXL_ilV5b`** (31 lines): Nova Anime XL by *Crody*,
  DARE-merged from NoobAI / Illustrious / ChenkinNoob. **This project's
  `ilV5b` is an earlier revision than the documented latest**, and the
  settings found (Euler a, 20–30 steps, CFG 4–6) belong to that later
  version and are flagged as unverified for `ilV5b`.
- **`flux1-kontext-dev`** (32 lines): Black Forest Labs' 12B rectified-flow
  transformer for instruction-guided editing, non-commercial licence,
  guidance scale 2.5 in the example code. No recommended step count found,
  and no independent check of the fp8-scaled variant's quality/speed trade.

Each one flags a version mismatch between what is published and what this
server holds. That is worth more than a confident paragraph would have been.

## Publish-condition check (Developer, independent of the run's own)

`grep -rniE "agstudio|agpc|\.local|home\.arpa|/Users|localhost|<ipv4>|tailscale|7801|8188|autodev|gitea|README_PROJECT"` over all of `main/`
at `0ab93b3`: **no hits.** No hostname, IP, port, path, internal repository
name, and no dangling pointer.

The pointer matters: task1's `INDEX.md` told the reader to see
`README_PROJECT.md` for the `gentest-<subject>/` convention. That file is
workspace-local and outside `main/`, so a stranger reading a published copy
could not follow it — the same class of defect the pattern doc forbids for
`test.md`. Caught on review, fixed in task2's rewrite, and worth noting that
the run introduced it *by following* `README_PROJECT.md`'s own wording.

## Frictions

1. **The planner deleted the one fact the task could not derive.** The
   mission text gave the backend as a literal IPv4 with a note about the
   AAAA stall. The `task1.md` the superdirector wrote says only *"use the
   backend's IP address directly — the `.local` hostname stalls on an
   unanswered AAAA lookup for several seconds"* — **the address itself is
   gone**, and the warning that survived is about a name the task was never
   given either. The supercoder then spent roughly twenty tool calls
   hunting: `swarmui.local` (does not resolve), `find / -iname "*swarmui*"`,
   grepping the whole autolab checkout, `/etc/hosts`, `ifconfig`, `arp`,
   `dig`, and finally `tailscale status`, probing three tailnet peers until
   one answered on `:7801`. It succeeded — and reported the recovery
   plainly — but that is most of task1's 38 turns and a good share of its
   $0.77. Generalised: **planning paraphrases the mission, and a paraphrase
   drops literals.** Anything a task cannot re-derive (an endpoint, a
   credential path, a revision) must survive planning verbatim. Step 3's
   workplan therefore says so explicitly, since a generation matrix without
   an endpoint is not merely slower, it is nothing.
2. **21 minutes of dead time on a gate that answered immediately.** Posting
   into `workrun-task2-m-1` while `M-2` was still open got the one-line
   reply *"Please complete previous work (M-2)"* — the sequential gate at
   `mission.py:444`, working exactly as designed: a task whose predecessor
   is not in a `completed` Plane state never launches an agent. The
   Developer's own watcher was waiting on a *file* appearing rather than on
   a *reply* arriving, so the refusal sat unread for 21 minutes. Two things
   follow. The gate is well-behaved — it names what blocks it, costs
   nothing, and is not silent. And **the requester's acceptance is a
   scheduling dependency, not a courtesy**: autolab's contract that a task
   is not closed until the requester agrees is what holds the next task
   shut. Posting acceptance into `workrun-task1-m-1` (message 3081) made the
   run mark `M-2` Done, resolve its topic and clear the gate in 42 s /
   $0.149. Watch the conversation, not the filesystem.
3. **The close-out re-serve is a real cost line.** Accepting a finished task
   is another paid run ($0.149 here, and it will be one per task). Worth
   knowing when sizing a mission: a two-task mission is five runs, not
   three.
4. No permission-classifier stops on any in-system run this step. One stop
   on the Omni Agent's own side is recorded in `report3.md`, where it
   belongs.

## Deus Ex Machina interventions

- None. Every byte in `main/` was written by an autolab run. The Developer
  posted the mission, approved the plan, started each task, accepted each
  task, and verified the publish conditions afterwards — all of which is the
  requester's own role in the contract, not work taken off an agent.
