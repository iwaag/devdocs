# scheduled_routine p9 — Plan

Braindump: `braindump.md`. Create a new **`mediagen`** study project: media
generation (ComfyUI / SwarmUI, as agforge uses them) where models and
workflows are tried under varied prompts and settings and the results are
evaluated. Feeding the findings back into agforge or a new agent is
**out of scope** for this phase.

The braindump's shape: reuse the `studyarxiv` framework — `summary.md`
knowledge files built from checkpoint/workflow basics and public best
practices, plus a dedicated **generation-test repository** in the same mould
as `localtest-<id>/`, expanded into the project workspace, where generation
tests are run freely. Two decisions added to the braindump:

- **First subject: `pixelArtDiffusionXL_spriteShaper`** (pixel art, handy
  for development assets). Sweep prompt wording and style to find the
  conditions under which images usable *as assets for a stated
  requirement* come out easily.
- **No level scale.** The arXiv local-test L1–L4 axis does not fit media
  generation and is **not imported**. Its replacement is a **`tips.md`**
  per subject — a free-form, append-only collection of generation tips the
  tests discover.

Experimental, non-public environment; destructive phase, no backward
compatibility. Only the **MUST NOT** lines are prohibitions — everything
else is advice the implementer may override with reason.

## Background the implementer should know

### The study framework you are reusing

- `agautolab/agent/project_pattern.md` defines the `study` pattern
  (`main/` = publish-ready knowledge, `publish/` = reviewed copy, never
  pushed by the agent) and the repository-backed local test
  (`localtest-<id>/`, `autolab project init-localtest <id>`, states in
  `localtest.yaml`, raw log in `report.md`, distilled `test.md` in
  `main/`). The level part of that contract stays with papers; here the
  distilled output is `tips.md`. The pattern doc says a pattern is a starting
  layout, not a constraint: extra folders via `init-repo` are fine and go
  into `README_PROJECT.md`.
- Project bootstrap is a **Developer request in Zulip**, not a script
  (p5 `report1.md`): create `#pj-mediagen` in the project channel folder,
  drop the 80-byte marker `README_PROJECT.md` into
  `agautolab/.local/projects/mediagen/`, post a one-paragraph request in
  `#pj-mediagen › workplan-create`. The listener ensures the Plane project
  and the superdirector creates `main/`/`publish/` on Gitea with the
  standard names (`autodev/mediagen`, `autodev/mediagen-publish`). ~20 s,
  ~$0.12 last time. Clones come back on an unborn branch — the first commit
  needs `git push -u origin main`.
- Missions after that are `workplan-…` topics in `#pj-mediagen`, posted by
  the Developer (p7/p8 precedent). The schedule/Front leg (`#front ›
  routine-<name>` standing text + `rtschedule` one-shot event through the
  production dispatcher, `pj-agdev/devenv/routine/dispatch.py`) exists and
  works — use it only if a step gains something from it.
- `init-localtest` is paper-id-shaped (`localtest-<id>/`,
  `autodev/<project>-localtest-<id>`, `paper_id:` in the yaml). It is
  still the cheapest way to get a stateful, repository-backed test folder;
  using it with a subject id such as `sdxl-perfectdeliberate` works
  mechanically. If the `paper_id`/`localtest` naming grates, `autolab
  project init-repo gentest-<subject>` plus a hand-written `gentest.yaml`
  is the alternative; either is fine, just record the choice in
  `README_PROJECT.md`. Renaming the CLI for media is not this phase's job.

### The live generation stack (checked 2026-08-30)

- **SwarmUI** `http://agpc.local:7801` (0.9.7.4) on the `agpc` GPU node:
  one **Quadro RTX 8000, 48 GB VRAM**, 128 GB RAM. It is agforge's
  production backend — every `assetrun` shares this GPU. Keep tests
  image-first and sized; a 14B video run holds the card for minutes.
- **ComfyUI**: the standalone `:8188` that `AGFORGE_COMFYUI_URL` points at
  answered nothing today (connection refused). SwarmUI's embedded ComfyUI
  0.33.1 is reachable at `http://agpc.local:7801/ComfyBackendDirect/…`
  (`/system_stats`, `/prompt`, `/history`, `/object_info` all work there).
  Use that, or start the standalone one on agpc
  (`~/StabilityMatrix/Packages/ComfyUI`, deployment profile `comfyui` in
  pj-clusterintent, observe-only) — your call; note which in the report.
  `agpc.local` lookups can stall ~5 s on AAAA; use `192.168.0.110` or
  force IPv4 if it bites.
- **Models on agpc** (SwarmUI `ListModels`, `subtype: Stable-Diffusion`,
  `depth: 3`; the API needs a `session_id` from `POST /API/GetNewSession`):
  - SDXL/Illustrious/Pony family: `perfectdeliberate_XL` (agforge's
    default, photoreal), `novaAnimeXL_ilV5b`, `noobaiXLNAIXL_vPred10Version`
    (v-pred — needs its own sampler settings, a nice test subject),
    `pixelArtDiffusionXL_spriteShaper`, `oneObsession_v16/19/22`,
    `prefectPonyXL_v6`, `symPonyWorld_v20`, `autismmixSDXL`, and several
    adult-oriented checkpoints/LoRAs.
  - Flux: `flux1-kontext-dev`, `flux1-dev-kontext_fp8_scaled` (editing);
    `omnigen2_fp16`.
  - Video: Wan 2.2 t2v/i2v 14B fp8 + `lightx2v_4steps` LoRAs (4-step
    distill — big speed lever), LTX-2 19B dev/distilled (+ LTX VAEs and
    LoRAs), MiniMax H3 (agforge's video workflow).
  - Audio: `ace_step_1.5_turbo_aio` (agforge's music workflow).
  - LoRAs: `pixel-art-xl-v1.1`, the Wan/LTX speed LoRAs, and many
    character LoRAs. ControlNet: none.
  The library contains adult content. `main/` and `publish/` are written
  publish-ready, so pick general checkpoints for the study and leave the
  rest uncatalogued unless there is a reason.
- **Quirks agforge already paid for** (`agforge/.local/devenv.md`):
  `GenerateText2Image` rejects a request without `model`; other params
  fall back to server defaults; the image ref in the response is a
  relative `View/local/raw/...` path, plain GET; output is JPEG 512² under
  current server defaults, so pass width/height explicitly.
- **Reusable code, not to be re-invented**: `agforge image generate`
  (SwarmUI text2image → MinIO presigned URL; flags `--model --width
  --height --steps --cfgscale --seed`), `agforge video generate`,
  `src/agforge/comfy_video.py` (submit an API-format workflow by
  `class_type`, poll `/history`, fetch outputs — copy the pattern),
  `service/transform.py` (resize/convert/re-upload), API-format workflows
  in `agforge/.local/resources/comfywf/{video,music}/`. The gentest repo
  may vendor a small client or shell out to `agforge`; the CLI is on every
  agforge role's PATH but not on autolab's — the workspace can call
  `uv run --project <agforge path> agforge …` or talk HTTP directly.
- **Where images go**: generated files are large and often not
  publishable. Keep them in the gentest repo's ignored `.local/` and/or the
  `agforge` MinIO bucket (presigned URLs expire in 60 min; keep the
  `[S3KEY]` and re-sign via `POST /api/resign` on `:8092`). A handful of
  small contact sheets committed to the gentest repo is fine; `main/`
  should carry text plus at most a few small representative images.
- **Evaluation**: judge against the stated asset requirement, not a
  generic score. autolab's Claude runs can read images, so the running
  agent can do the judging itself; a local vision model on agpc via
  Ollama is the cheaper alternative; a Zulip post with the contact sheet
  for a human verdict is also legitimate for a first pass. Seed-fixed
  A/B grids varying one axis at a time answer most "which wording works"
  questions; record per-image wall time from SwarmUI.
- **`tips.md` is the deliverable**, not a grade. Shape suggestion: one
  bullet per tip, each with the evidence that produced it (matrix id or
  seed + settings, one line), dated, appended never rewritten — a later
  run can contradict an earlier tip by adding a newer one. A tip is
  "under these conditions this comes out"; failures ("negative prompt X
  did nothing") are tips too. Keep `summary.md` (what the model is,
  upstream recommendations) and `tips.md` (what *we* found) separate so
  the second never gets polluted with hearsay.
- **Pixel art specifics worth knowing before the matrix**: SpriteShaper
  is an SDXL pixel-art checkpoint; `pixel-art-xl-v1.1` LoRA is also on
  agpc and can be compared against it. Native SDXL output is not on a
  pixel grid — usable assets usually need a post step (downscale with
  nearest-neighbour to e.g. 64–128 px, optional palette quantisation,
  background removal for sprites); Pillow does all of it and the runner
  should carry it so "asset-usable" is judged after the post step, not on
  the raw 1024² image. Typical asset requirements to sweep against:
  single subject, centered, plain/flat background, side or 3/4 view,
  consistent outline, N-colour palette, transparent-able background.
  Sweep axes: prompt wording (tags vs sentences, "pixel art"/"sprite"
  /"16-bit" tokens, negative prompt), style words, CFG, steps, resolution
  (SDXL sizes; square vs 2:1 for tiles), with/without LoRA, seed fixed.
- p7/p8 learnings that apply: missions cannot quote their own cost —
  read `agautolab/.local/agent/<role>/run-NNNN.json`; `WORK_TIMEOUT_SECONDS
  = 1200` — a matrix of ~20 SDXL images fits, a video matrix does not, so
  split video work into its own task or leave it for later; if a workrun
  dies oddly, check the transcript for leaked-CLAUDE.md permission denials
  first; the `main` clone may need `git remote set-url` to
  `agstudio.local`.
- In-system edits to `main/`, `publish/` and the gentest repo go through
  autolab missions; harness-side edits (pattern doc, standing texts,
  marker file) are ordinary Developer work. Record Deus Ex Machina
  interventions with the one-line handoff note.

**MUST NOT**: push `publish/` (Developer pushes by hand); commit
credentials or private host facts (`agpc`, `agstudio`, IPs, ports, MinIO
keys) into `main/` or `publish/`; work around the permission classifier
(stop and report, `localrule.md`).

## Step 1 — the project and its contract

1. Extend `agautolab/agent/project_pattern.md` with a short "generation
   test" paragraph: that a study project may hold repository-backed
   generation tests (same folder/repo naming path as local tests), what
   the resumable yaml records (subject, backend, model/workflow, state),
   that the distilled result is `main/<subject>/tips.md` (append-only,
   evidence per tip, no level scale), and that `INDEX.md`'s per-subject
   column just says whether tips exist. A few sentences, not a
   framework. Commit and push agautolab
   (`localrule.md`); no restart needed for a doc change.
2. Bootstrap `mediagen` as the Developer, p5-style: channel
   `#pj-mediagen`, marker file, then in `workplan-create` a request in the
   spirit of: *study pattern, standard repository names; `main/` holds
   summaries of generation models and ComfyUI/SwarmUI workflows and the
   results of generation tests; `publish/` holds reviewed copies only.*
3. Verify: Plane project exists, both Gitea repos exist, `README_PROJECT.md`
   updated by the run.

Report `report1.md`: pattern-doc diff, the request and reply, run record
numbers (cost/turns/time), anything the bootstrap did differently from
p5.

## Step 2 — knowledge: `summary.md` files and the index

One autolab mission in `#pj-mediagen › workplan-…`: build the first
knowledge layer in `main/`.

- Inventory what the live SwarmUI actually serves (ListModels) and write
  `main/INDEX.md` — one row per catalogued subject (checkpoint, LoRA
  set, or workflow family) with architecture, intended use, and a
  `tips` column (`no` / `yes`, or the date of the last tip). General-
  purpose subjects only; the agent decides the cut.
- Write `main/pixelArtDiffusionXL/summary.md` first and thoroughly
  (checkpoint origin and version, the `pixel-art-xl` LoRA as a sibling,
  recommended sampler/steps/CFG/resolution/prompt vocabulary from the
  model page and public pixel-art SDXL practice, cite URLs, the
  post-processing pixel-art pipelines people use, and the first tests
  worth running). Then, if budget allows, brief summaries for 2–3 more
  general subjects (e.g. `perfectdeliberate_XL`, `novaAnimeXL`,
  `flux1-kontext-dev`) so the INDEX is not a one-row table — optional. Web access is the mission's;
  paraphrase, no long quotations (publish conditions apply from day one).
- Commit and push `main`.

Report `report2.md`: mission text, INDEX as landed, one summary in full,
what the agent could not find out, cost numbers from the run JSON.

## Step 3 — the generation-test repository and one real matrix

One mission (two tasks if the agent prefers): create the test repository
for `pixelArtDiffusionXL` and run a bounded matrix aimed at one concrete
asset requirement — the Developer states it in the workplan (e.g. "a
64×64 side-view walking-animal sprite on a flat background for a 2D
game", or a 32×32 item icon set). Pick one; the point is that "usable"
is judged against a requirement, not on taste.

- Create it from the workspace (`init-localtest <subject>` or
  `init-repo gentest-<subject>`; record in `README_PROJECT.md`). Commit a
  small runner (shell or Python) that takes a matrix spec (prompts ×
  settings, fixed seed) and produces images, a `results.md`/`.csv` with
  per-image parameters and timings, and a contact sheet. `.local/` for
  raw outputs.
- Run a matrix that fits one task budget (~20–30 SDXL images): e.g.
  3 subjects for the requirement × {tag-style, sentence-style} prompts ×
  {2 CFG} at fixed steps and seed, then one follow-up axis chosen from
  the first results (LoRA on/off, negative prompt, resolution). Apply the
  post step, judge each cell against the requirement (usable / usable
  with edits / not), keep the seed.
- Distil into `main/pixelArtDiffusionXL/tips.md`: every tip the matrix
  supports, each with its evidence line; a "still open" list at the end.
  Set the INDEX column. Commit/push `main` and the test repo. Keep
  `main/` free of host facts and of links into the test repo; one or two
  small contact-sheet PNGs in `main/` are fine if they carry the point.

Report `report3.md`: repo layout, runner usage, the matrix and its
timings, the contact sheet (path or re-signed URL), `tips.md` as
landed, backend used (SwarmUI API vs ComfyBackendDirect vs standalone
ComfyUI), frictions (GPU contention with agforge, timeouts, model-load
time), cost numbers.

## Step 4 — phase report

`report.md`:

- Does the study framework fit media generation as the braindump hoped?
  What of `localtest` transferred as-is, what was renamed or bent, and
  whether `tips.md` (free-form, no scale) held up as the result format —
  did the tips stay evidence-backed, and would a second run know where
  to append.
- Did the matrix find conditions under which SpriteShaper yields
  asset-usable pixel art for the stated requirement? The one or two
  findings agforge's image toolset would care about (hand-off material
  for a later phase, not action).
- Costs/timings from the run JSONs; GPU time actually consumed.
- Recommend only: whether a `routine-mediagen` standing text (one subject
  summarised + one matrix per fire, p6/p8 shape) is worth adding, or the
  project stays manual-fire; whether video/audio subjects belong in the
  next phase; whether `publish/` should get its first review run.
- Deus Ex Machina interventions, each with its handoff line.

## Out of scope unless a live run forces it

Feeding results into agforge (new toolsets, model choice logic, new
agents); a recurring schedule; video/audio matrices beyond a smoke
generation; renaming `init-localtest`/`paper_id` for media; any level or scoring
scale for generation tests; a
`publish/` review run; fixing the standalone ComfyUI service on agpc
beyond starting it; model downloads to agpc; dispatcher/schema changes.
