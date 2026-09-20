# study_import p1 — step 1: current state and the test conditions

Date: 2026-09-20. Everything below was read or probed on this host; nothing
was changed yet.

## Services, as nctl and the endpoints say

`nctl status` is green: Nautobot 3.1.3, one worker, eight hosts dumped
within the last 3 h except the retired fixture. Every agent listener is up
under launchd (agfront, agautolab, agforge, agobserver, archsage, cagent,
comfy-notifier, agentroom, agforge request service). agforge's listener
saw a Zulip 502 at 06:00 today and re-registered its queue by itself.

The generation backends forge is configured for both answer:

| backend | state | what it offers |
|---|---|---|
| SwarmUI 0.9.7.4 on the GPU node | answers (302 on `/`) | `agforge image generate`'s only path; default model `perfectdeliberate_XL` |
| ComfyUI 0.35.0 on the same node, Quadro RTX 8000 48 GB, ~15 GB free at probe time | 200, 1744 node classes | direct graphs, the path mediagen's p6 chose over SwarmUI |

Checkpoints on the card that matter here: `perfectdeliberate_XL`,
`pixelArtDiffusionXL_spriteShaper`, `novaAnimeXL_ilV5b`, plus
`flux1-kontext-dev` (bf16 and fp8) as UNETs and the `pixel-art-xl-v1.1`
LoRA. ComfyUI also has `RemoveBackground` / `LoadBackgroundRemovalModel`
and the Bria/Recraft cloud nodes; whether the local remove-background model
is actually installed is unverified and is a step 3 question.

## Knowledge: where it is and what revision

- mediagen `main/` is `autodev/mediagen` on the local Gitea, checked out in
  autolab's project workspace, HEAD **`c415b0c`**, clean, equal to origin.
  `publish/` is `7220044` (first gate-reviewed publication of all seven
  subjects). Its `subjects/INDEX.md` lists four checkpoints, three workflow
  families and one music subject. **No subject covers icons, UI/HUD art,
  templates, vector drawing or background removal.** The nearest material:
  `pixelArtDiffusionXL/tips.md` (subject dominates every axis; CFG effects
  are subject-conditioned; sentence prompts hurt most subjects),
  `perfectdeliberate_XL/summary.md` (no tips; recommended 1024² or
  1024×1536), `flux1-kontext-dev/summary.md` (editing model, guidance 2.5,
  no tips, non-commercial licence), `videoFrameExtraction/tips.md` (ComfyUI
  still is six nodes on `POST /prompt` → `/history` → `/view`; byte-identical
  at a fixed seed; SwarmUI and ComfyUI stills differ pixel-wise at the same
  settings).
- The gentest repositories hold reusable code: `gentest-pixelArtDiffusionXL/runner.py`
  (SwarmUI matrix runner + 64 px post-process + contact sheet) and
  `gentest-videoLoopPipeline/pipeline.py` (ComfyUI still graph, `POST /free`).
  Both carry host values as defaults, so they are localised code, not `main/`.
- The advance_mediagen_study episode (`devdocs/episodes/advance_mediagen_study/p1..p6`)
  is the Omni-side record of those studies.

**What forge can reach today: none of it.** A generator run gets `tools/`
(copies of `agent/toolsets/toolset-*.md` chosen by the front), `agents.md`,
`required_items.md`/`plan.md`/`chatlog.md`, and nothing else. The image
toolset names `agforge image generate`, `curl`, Pillow, `sips`, `file`,
`jq`. No path to mediagen, no index, no way to learn that `flux1-kontext`
or the pixel-art LoRA exist. The harness layer (`agag.harness`, `add_dirs`)
can already pass `--add-dir` to claude_code / agy / codex, but forge does not
use it. The `speech` toolset is an empty body: listed, not executable, the
exact distinction step 4 must make visible.

## forge's tools, grants and delivery path (as of `944b75b`)

- Roles: `front` (sonnet, read/write + `agforge`/`agentchat`), `generator`
  (sonnet; `agforge`, `uv`, `python3`, `curl`, `sips`, `magick`, `ffmpeg`,
  `jq`, file verbs, Read/Write/Edit/Glob/Grep/WebFetch). `magick` and
  `ffmpeg` are granted but **not installed** on this Mac; `rsvg-convert`,
  `inkscape`, `cairosvg`, `numpy`, `rembg` are absent too. What exists:
  Pillow 12.3 in forge's venv, `sips`, `node` v26, `jq`, `curl`.
- Flow: `assetplan-<stem>` → front writes `required_items.md` +
  `toolsets.csv` → generator writes `plan.md` (posted, `[selfnote][doc]`,
  `[selfnote][tools]`) → forge opens `assetrun-<stem>-a<id>` → a post there
  runs the generator with `plan.md`, `tools/`, `chatlog.md`; `result/` is
  zipped, uploaded to MinIO (`agforge` bucket), a presigned URL plus
  `[S3KEY]` is delivered to the plan topic. `transform.py` resizes,
  converts and re-uploads one file. Image generation is synchronous via
  SwarmUI; ComfyUI is used only by video/music submit + `comfy fetch`.
- The generator's cwd is `.local/agentws/r<run id>/generator/`; the last
  runs on disk are r5820…r6036, each with `plan.md`, `tools/`, `result/`.

## The first request, made concrete

Taken from `test_theme.md` with these assumptions filled in (recorded here
so nobody waits on them; step 5 changes any of them by asking):

| item | assumption |
|---|---|
| set | five fruits chosen for hue separation: apple (red), banana (yellow), grape (purple), orange (orange), pear (green) |
| master | 512×512 PNG, RGBA, one file per fruit `icon_<fruit>.png` |
| display sizes | 128, 64 and 32 px; the 32 px check is the identity test |
| tile | rounded square, corner radius 22 % of the side, one hue per icon, vertical linear gradient light-top → darker-bottom of the same hue, ~10 % margin inside the tile edge, fruit fills ~65 % of the tile |
| outside the tile | fully transparent (alpha 0 at the corners) |
| delivery | the five PNGs + one preview sheet (`preview.png`: all five at 512 and rows at 64/32) in `result/`, through the existing zip → MinIO → presigned URL path |
| states | static only; hover/pressed/disabled not required in p1 |

Second and third requests for step 5: replace one fruit (e.g. pear →
strawberry) keeping the other four byte-identical; change the tile hue or
output size for all five from the same editable sources.

## Evaluation conditions

1. **Identity at 32 px**: each of the five is nameable from the 32 px row
   alone, judged by eye on the preview sheet, not on the 512 master.
2. **Template consistency**: radius, gradient stops, margin and fruit
   bounding box measured from the pixels are equal across the five
   (a Pillow check, not an impression).
3. **Transparency**: alpha 0 outside the rounded tile; `file` reports RGBA.
4. **Style unity**: same rendering method and line weight across the five.
5. **Cost**: wall time per icon and per set, and how many manual touch-ups
   the set needed, recorded per method.
6. **Rework**: the one-fruit replacement leaves the other four unchanged
   (hash-equal), and the hue/size change is one parameter, not five edits.

## Candidate methods to try in step 3 (order, not a fixed route)

1. **Template + fruit, composed** (first candidate): tile drawn in Pillow
   from parameters; fruit either drawn as vector-ish shapes in Pillow, or
   generated on a flat background by SDXL (ComfyUI, `perfectdeliberate_XL`
   or `novaAnimeXL`) and keyed/removed (ComfyUI `RemoveBackground` if its
   model is present, else colour keying in Pillow), then composed.
2. **Whole icon generated**: one prompt per fruit describing the tile; the
   comparison case for consistency and transparency.
3. **Whole icon as SVG**: Pillow cannot rasterise SVG and no rasteriser is
   installed; `node` is present, so a small JS/canvas rasteriser or a
   Python-only path would be needed. Kept as a comparison case.
4. **Kontext edit**: generate one icon and ask `flux1-kontext` to swap the
   fruit — the "replace one" case done by model rather than by template.

Commercial alternatives (Recraft/Bria nodes are present in ComfyUI as
cloud nodes, plus icon generators such as Recraft or OpenAI image edits)
are proposals only; nothing here is contracted.

## Who does what from step 2

autolab experiments and implements in the mediagen workspace; cagent is
asked for anything the environment lacks (a background-removal model, a
Python package in a venv, a rasteriser); forge is the user who must find and
run the result. The Omni Agent writes these reports and marks every place it
stands in for an agent as a hand-off candidate.
