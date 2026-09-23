# Step 1 report — inputs confirmed, two tasks prepared

Plan: [plan.md](plan.md) step 1. 2026-09-23 (13:00–13:30 UTC), by the Omni Agent. Nothing
was changed in code, configuration or the human's repository; this step reads and decides.
No agent run was bought.

## Environment as found

| Check | Result |
|---|---|
| `nctl status` (from `pj-clusterintent/nctl`) | Nautobot 3.1.3 reachable and authenticated; 1 celery worker, 0 pending jobs; all five submodules clean |
| Observation age | agstudio, agautolab1, aghub, agpc, agdnsmasq, agbach collected 4.1 h before this step; agfixture stale (known, 1344 h) |
| Listeners on this Mac (launchd) | agfront, agautolab, agforge, archsage, agobserver, cagent, comfy-notifier, agentroom, agforge request service all running |
| ComfyUI on the GPU node | 0.35.0 answering; queue empty; every node of the Developer's workflow present (`Flux2Scheduler`, `EmptyFlux2LatentImage`, `ComfySwitchNode`, `SamplerCustomAdvanced`, …); the exact model files the references were made with are installed: `flux2_dev_fp8mixed.safetensors`, `mistral_3_small_flux2_bf16.safetensors`, `Flux_2-Turbo-LoRA_comfyui.safetensors`, `full_encoder_small_decoder.safetensors`; `LoadImage`/`VAEEncode` exist, so image-to-image is possible |
| forge's image tools | `agforge image generate` is SwarmUI only (prompt, size, steps, cfg, seed, `--init-image`); `agforge comfy fetch` collects a finished ComfyUI job; `video submit` / `music submit` queue ComfyUI jobs through the notifier. **There is no ComfyUI image submission** — the step 4 gap the plan anticipated |
| Production checkout | `autodev/protoprey` `1f4c2f4` (fresh clone in the scratchpad); autolab's workspace `main` at the same commit, clean; `direction/` at `26690b4` with both p2 adoption entries committed (the lag noted in p2 report 5 is gone) |
| Godot | 4.7.2.stable on this Mac (`/opt/homebrew/bin/godot`) |

## The published reference, inspected

`agrefs revision protoprey-refs` → **`6e017fb`** ("new todo", 2026-09-23, full
`6e017fb4cf6bf83b915e6ca6f7ccfcbc7ba8d820`). This is the revision adopted for the whole trial.
`agrefs changes protoprey-refs@0cd3c0b..6e017fb`: 16 files added under `biomes/`, one under
`tools/`, `human_advice.md` and `todo.md` modified. Front's cache holds the snapshot; autolab's
and forge's caches still end at `0cd3c0b` and will fetch it themselves when their tasks name it
(as p2 showed they do).

What is there, read in full by the Omni Agent:

- **Four locations**, each a `view.png` (1344×768) and `texts/discovery.md` (five paragraphs,
  2,900–3,300 characters) plus `texts/revisit.md` (two paragraphs): forest `flower_pond`,
  forest `mashroom_waterfall`, meadow `little_burrow`, meadow `mound`. The waterfall keeps its
  image under `image/` (singular); the other three use `images/`. The tasks say to map that on
  import, not to correct the original.
- **Biome art**: `biomes/{forest,meadow}/images/view.png` (top-down 1344×768) and `icon.png`
  (512×512), unrequested by `todo.md`; offered to autolab as optional.
- **Embedded generation metadata**: every PNG carries ComfyUI `prompt` and `workflow` chunks.
  All eight were made with Flux2 dev (`flux2_dev_fp8mixed`), the Mistral-3-small text encoder,
  the Turbo LoRA (8 steps), guidance 4, euler, seeds recorded. The four location prompts share a
  vocabulary — "micro pov / micro view, very low angle, bottom view, … seen from below, depth of
  field" — that is the look the trial asks forge to match.
- **The images, viewed**: ground-level camera, the player miniature (a torso-sized bee on a leaf
  at the pond; mushroom caps as parasols at the waterfall; a caterpillar and a burrow mouth
  wider than a person at the burrow; a dirt rise as a hill with twig "branches" at the mound).
  Image and text agree in every case; each discovery text ends on an open question and each
  revisit text says the place is known now.
- **`todo.md`**: two biome templates; exploring discovers a location "the same as a predator";
  a discovered location can be revisited with changed flavour text; map later, list or buttons
  now. **`human_advice.md`**: a new "画像生成のコツ" section — Flux2 through ComfyUI for
  scenery (prompt-following, but needs detailed prompts; uncanny with creatures), OneObsession
  through SwarmUI for creatures; the workflow file `tools/image_flux2_text_to_image.json` is
  the ComfyUI template as is; "check first that the embedded parameters are readable".
- **`tools/image_flux2_text_to_image.json`**: the API-format graph (positive prompt node
  `98:6`, size in `98:47`/`98:48`, seed in `98:25`, Turbo switch `98:104`, steps 20/8). Its
  prompt is still the template's fashion-shoot text; the location PNGs' own `prompt` chunks are
  the real examples.

## The two tasks

Both are written out, ready to post, in this folder:

- **[task_locations.md](task_locations.md)** — Task A, for autolab through Front
  (`front-protoprey-p3-locations-20260923` → `#pj-protoprey › workplan-locations`). Inputs:
  `protoprey-refs@6e017fb` with every path, the `todo.md` text quoted and translated, the base
  commit `1f4c2f4`, the four ids as the Developer spelled them, the `image/`/`images/` mapping.
  Expected outputs: a data-driven location format; exploration → first discovery → list →
  revisit → back to exploration in F; the meadow enabled; the wolf loop intact; state carried
  across respawns and reset on a new game; long text readable and 1344×768 framed in 1280×720;
  the adopted files imported with their identity; **the handoff contract**
  (`docs/ADDING_A_LOCATION.md`); headless tests; `direction/REFERENCES.md` entry.
- **[task_forest_location.md](task_forest_location.md)** — Task B, in two posts. B1 asks Front
  to have forge produce one new forest location package: the same references, what the two
  forest examples establish (as read from the originals, with the actual prompts and settings),
  the delivery shape (one 1344×768 PNG, `discovery.md`, `revisit.md`, identity, provenance
  note with settings, attempts, elapsed time, departures), Flux2 as the starting point, the
  place and method forge's own. B2 asks Front to have autolab integrate the delivered package
  and to report whether any engine change or asset repair was needed. Two bracketed parts are
  filled later: the handoff contract from step 2, and the ComfyUI route from step 4's
  capability check.

Both tasks name the reference by identity only (`protoprey-refs@6e017fb:<path>`); no path on
this machine travels in them.

## Decisions taken in this step

- **One revision for the trial**: `6e017fb`. A later push by the Developer during the trial
  will be adopted only by an explicit new unit, as in p2.
- **Route**: both tasks go through Front as the Developer's entrance, relayed and marked as
  relays, so that Front → autolab and Front → forge → autolab are the paths measured. The Omni
  Agent posts, watches, verifies on a fresh clone, and answers routine questions as the
  requester's stand-in; anything it has to do for an agent is recorded as a handoff candidate.
- **The handoff format is autolab's to define** in Task A (item 7), because the integrator is
  the party that has to consume it; forge is then asked for exactly that shape. This keeps
  "define the small asset handoff together with the implementation" literal.
- **The ComfyUI image route is left to step 4**, where the plan puts the capability check. What
  is known now: the models and nodes are on the node, forge has a fetch and an async ticket
  pattern for video/music, and nothing submits an image graph. The smallest adapter is likely
  a `submit` that takes the Developer's workflow file with prompt, size and seed overrides,
  reusing the existing notifier callback and `comfy fetch`.

## Not done, on purpose

- No sync of `6e017fb` into autolab's or forge's cache by the Omni Agent: each agent fetches
  the revision its task names, which is part of what the trial observes.
- No deploy to agautolab1 and no change to any listener.
- The wolf-scene creative evaluation from p2 is still the Developer's; nothing here depends
  on it.

## Cost of the step

No agent run was bought. Omni Agent only (reads, one `agrefs sync` in agfront's cache, one
ComfyUI `object_info` read).
