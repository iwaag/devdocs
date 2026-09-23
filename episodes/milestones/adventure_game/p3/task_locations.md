# Task A — locations from the human's examples (autolab, via Front)

Prepared in step 1 (2026-09-23) by the Omni Agent. Posted as a relay into `#front ›
front-protoprey-p3-locations-20260923`; Front opens the mission in `#pj-protoprey`.

---

[relay by the Omni Agent — the Developer published this themselves on 2026-09-23 and asked me to carry it; what is marked as the Developer's is theirs, routine implementation choices are autolab's]

The Developer published **`protoprey-refs@6e017fb`** ("new todo", full `6e017fb4cf6bf83b915e6ca6f7ccfcbc7ba8d820`). `agrefs changes protoprey-refs@0cd3c0b..6e017fb`: `biomes/` (new, 16 files), `tools/image_flux2_text_to_image.json` (new), `human_advice.md` and `todo.md` modified. `todo.md` now says (Japanese, translated):

> バイオームのテンプレートを二つ用意した。 — "I prepared two biome templates."
> バイオームの探索によりlocationを発見、この点ではpredatorと同じ。 — "Exploring a biome discovers a location; in this respect it is the same as a predator."
> 一度発見した後のlocationは再訪でき、フレバーテキストが変化する前提。 — "A location once discovered can be revisited, and its flavour text changes."
> locationの再訪は、後からマップシーンを用意して選択できるようにしようと思うが、今はとりあえずリストUIかボタンで選択という感じでいい。 — "Revisiting will later be chosen on a map scene; for now a list UI or buttons is fine."

Please open one mission for autolab in `#pj-protoprey`, `workplan-locations`, adopting **`protoprey-refs@6e017fb`** (record it in `direction/REFERENCES.md`; the earlier units keep `a3c8196` / `0cd3c0b`). Base: `autodev/protoprey` at `1f4c2f4`. Carry the text below to autolab as it stands — the paths and the numbered items are the request.

**References at `protoprey-refs@6e017fb`** (read the originals with `agrefs`; the four location images are Flux2 renders whose PNG `prompt` chunk holds the generation prompt):

- forest: `biomes/forest/locations/flower_pond/images/view.png`, `…/flower_pond/texts/discovery.md`, `…/flower_pond/texts/revisit.md`; `biomes/forest/locations/mashroom_waterfall/image/view.png`, `…/mashroom_waterfall/texts/discovery.md`, `…/mashroom_waterfall/texts/revisit.md` — note **`image/` (singular)** in the waterfall folder: map it on import, do not correct the original.
- meadow: `biomes/meadow/locations/little_burrow/{images/view.png,texts/discovery.md,texts/revisit.md}`, `biomes/meadow/locations/mound/{images/view.png,texts/discovery.md,texts/revisit.md}`.
- biome art, optional: `biomes/forest/images/{view.png,icon.png}`, `biomes/meadow/images/{view.png,icon.png}` (the views are top-down 1344×768, the icons 512×512).
- `human_advice.md` (aesthetics and story rules, unchanged in substance; its image-generation section is new), and `todo.md` quoted above.

All four `view.png` are 1344×768. Discovery texts are five paragraphs (about 3,000 characters); revisit texts are two.

**What to build**

1. **Locations, data-driven.** One location = stable id, display name, biome, view image, discovery text, revisit text. Choose the file format and layout (JSON in the manner of the wolf event files is fine); registration is data too. A further location must need only its content files and one registration entry — no new gameplay branch. Keep the ids `flower_pond`, `mashroom_waterfall`, `little_burrow`, `mound` as the Developer spelled them.
2. **Exploration.** In the F flow, exploring a biome discovers a location the first time (discovery text over its image) and adds it to a discovered-locations list; a discovered location can be revisited from a list or buttons (revisit text over the same image), then play returns to exploration. **Enable the meadow** for exploration (it has locations and no predator yet — say so in-game rather than disabling it). Keep the forest wolf discovery and the predation loop working alongside; the Developer says a location is discovered "the same as a predator".
3. **Discovery policy.** Simple and verifiable, so every supplied location is reachable in a short play — for example each explore step discovers the next undiscovered location of that biome, and the wolf stays on the forest's first explore as now. State the rule you choose. No randomness that can hide a location.
4. **State.** Discovered locations persist across biome visits and respawns (like the discovered predators) and reset on a new game (entering F from the menu). No disk save.
5. **Presentation.** Reuse the image+text step display. The discovery texts are long: make them readable (scrolling or paging) over the still, and frame a 1344×768 image sensibly in the 1280×720 window. Map navigation is for later; a list or buttons may use the same location ids and discovery state.
6. **Import.** Bring the adopted files into the game repository under `assets/` and `data/` — images byte-for-byte; texts may be converted from `.md` if your format needs it, and say so. Keep the identity `protoprey-refs@6e017fb:<path>` beside each import (ASSETS.md or the registration data).
7. **Handoff format for a new location** — the second deliverable. Write a short `docs/ADDING_A_LOCATION.md` (or a README section) that says exactly what a content producer delivers for one new location (folder and file names, image size and format, the two text files, id / display name / biome, a provenance note) and what the integrator does with it (where the files go, which registration entry to add). In a following mission forge will produce one new forest location in that shape and autolab will integrate it, so the format is the contract.
8. **Tests.** Headless assertions for: first discovery, one list entry per location, revisit text differs from discovery, biome association, return to exploration, persistence across respawn, reset on new game, missing-asset detection, and the existing wolf discovery + predation. README/VERIFY as usual. Commit and push `main` and `direction/`.

The old demo architecture (the hardcoded flow in `scripts/forest_flow.gd`, the predator-specific titles in `scripts/event_player.gd`, the wolf-specific label in `scripts/main.gd`) may be refactored or replaced; backward compatibility is not required. Menu entries A–E stay as they are.

Already decided by the Developer: list or buttons now, map later; English for every generated text. Unless a real ambiguity remains — bring it here before starting — plan and start in one pass, and bring the delivery here: commit, how to play it, the discovery rule, the handoff format, and which engine changes were needed.
