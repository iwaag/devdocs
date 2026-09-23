# Task B — one new forest location (forge, then autolab; via Front)

Prepared in step 1 (2026-09-23) by the Omni Agent. Two posts: B1 asks Front to have forge
produce the package; B2, after forge delivers, asks Front to have autolab integrate it.
The bracketed parts are filled from step 2's delivery (the handoff format) and from
step 4's capability check (the ComfyUI route) before posting.

## B1 — to Front, for forge (`#front › front-protoprey-p3-forest-location-<stamp>`)

[relay by the Omni Agent — the Developer's request, carried as they made it; the creative choice of the place and the production method are forge's]

Please ask forge (`#agforge-agstudio1`, an `assetplan-protoprey-forest-location` topic) for **one new forest location package** for ProtoPrey, and carry the text below to it as it stands.

**References:** `protoprey-refs@6e017fb` (full `6e017fb4cf6bf83b915e6ca6f7ccfcbc7ba8d820`), through `agrefs`. Read the actual images and prose — `agrefs show` for the texts, the harness's image reader over `agrefs path` for the images, and the PNG `prompt` chunk (ComfyUI metadata) for how the Developer made them:

- `biomes/forest/locations/flower_pond/images/view.png` + `texts/discovery.md` + `texts/revisit.md`
- `biomes/forest/locations/mashroom_waterfall/image/view.png` + `texts/discovery.md` + `texts/revisit.md`
- `human_advice.md` — aesthetics ("虚構とリアリズムのバランス"), the image-generation advice (Flux2 dev for scenery through ComfyUI; the embedded parameters), the story-writing rules (second person, bodily sensation, awe before nature)
- `tools/image_flux2_text_to_image.json` — the Developer's ComfyUI workflow, the template they used

What the two forest examples establish, as read from the originals: a **micro point of view** — camera at ground level, the player miniature (flowers, mushrooms and pebbles tower; a bee is torso-sized); a 1344×768 Flux2 dev render (`flux2_dev_fp8mixed`, `mistral_3_small_flux2_bf16`, Turbo LoRA, 8 steps, guidance 4, euler), with prompts like "micro pov, very low angle, bottom view, waterfall on left, forest, rocks, dirt, very tall wild flowers on right, moss, a few tall mashrooms seen from below, trees, depth of field, tall plants"; discovery prose of five paragraphs that walks the reader into the place through the body (sound before sight, mist on skin, a droplet bursting against the legs) and ends on an open question; revisit prose of two paragraphs that conveys familiarity — the same place recognised, feet placed with knowledge — without retelling the discovery.

**The content example and the delivery shape** are the implemented format in `autodev/protoprey` (`docs/ADDING_A_LOCATION.md` at `528605c` or later; the four human locations under `data/locations/<id>/` and `assets/locations/<id>/` are the worked examples). Deliver exactly one folder `<id>/` holding:

```
<id>/view.png        1344x768 PNG, ground-level (micro) point of view like the two forest examples
<id>/discovery.txt   English, about five paragraphs (~3,000 characters), one paragraph per line
<id>/revisit.txt     English, about two paragraphs, one paragraph per line
<id>/NOTES.md        display name, biome `forest`, provenance per file, generation settings and attempts
```

`<id>` is `snake_case`, new (not `flower_pond` or `mashroom_waterfall`). `NOTES.md` states the reference revision (`protoprey-refs@6e017fb`) and which example files were read, the generation settings (model, prompt, seed, steps, guidance, size) of the kept image, how many attempts were made and why the kept one was chosen, elapsed time, and notable departures from the references. Provenance for the image and texts is "newly produced by forge" plus the request id.

**Production route.** Your SwarmUI image tool cannot load Flux2 (checked: every backend fails to load `flux2_dev_fp8mixed`). The Developer's route is available as a localised capability you can find with `agforge knowledge list`: `localize/flux2_scenery/` (state `verified`, 2026-09-23). Read its README first (`agforge knowledge show localize/flux2_scenery/README.md`). It runs the Developer's own graph through ComfyUI, synchronously, about 40 s per 1344×768 render on an empty queue:

```
uv run --with requests python "$(agforge knowledge path localize/flux2_scenery/generate.py)" \
  --prompt "<your prompt>" --out <dir> [--seed N] [--width 1344 --height 768] [--steps 8] [--guidance 4]
```

It prints one JSON line with the output path and the settings used; the PNG keeps the ComfyUI `prompt` metadata like the references. The Developer's guidance is the starting point for scenery: Flux2 dev, the `tools/image_flux2_text_to_image.json` graph, 1344×768. Image-to-image is optional. Choose the place yourself — a forest location that does not duplicate the pond or the waterfall — and inspect your own output before delivering: viewpoint at ground level, the miniature scale legible, atmosphere consistent between image and text, no text, UI or watermark in the image, no people.

Record attempts and elapsed time in the plan and the delivery. If a part of the route is not available to you, say which part, and what you would need, rather than substituting a different route silently.

## B2 — to Front, for autolab (same `#front` conversation, after forge's result)

[relay by the Omni Agent]

forge delivered the forest location package: `[S3KEY] <key>` (download `<url>`, request `a<id>`, run topic `assetrun-…`). Please open `workplan-add-<location id>` in `#pj-protoprey` for autolab: fetch the package (ask forge's resign endpoint for a fresh URL if the link has expired), store the actual files in `autodev/protoprey` in the established location format, add the registration entry, keep forge's provenance note with the import (source `assetrun-…`, request `a<id>`, reference `protoprey-refs@6e017fb`), run the location tests and the wolf regression, and report: commit, whether any engine change or asset repair was needed (this is what the trial measures), and how to play the new location alongside the human's examples.
