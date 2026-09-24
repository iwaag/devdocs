# p4 request — the respawn biome view

Drafted in step 1 (2026-09-24) by the Omni Agent for the Developer to paste into `#front`
(a new `front-protoprey-p4-…` topic, or the Project Room's request door). The Omni Agent does not
post it. Everything below the line is the request; edit freely — it is the Developer's once posted.

---

I published **`protoprey-refs@a7b4763`**. Its `todo.md` asks for the next piece of ProtoPrey: a
scene, shown after a predation event ends, where the player chooses the biome to respawn in — the
Earth seen from space, rotating. Please take it to autolab as one mission in `#pj-protoprey`, base
`autodev/protoprey` `main` at `6330a3a`, and record the adopted reference in
`direction/REFERENCES.md`.

**References at `protoprey-refs@a7b4763`** (read them with `agrefs`):

- `scenes/modes/biome_view/images/bg.png` — **the image to use**: 1344×768 Flux2 still (its PNG
  `prompt` chunk holds the generation prompt). Earth lower-left, sun upper-right, starfield.
- `scenes/modes/biome_view/images/concept.png` — a concept mock ("Choose Your Spawn Biome",
  sixteen biome bubbles). **Reference only**, as `human_advice.md` says about concept images: take
  the mood and the idea of choosing from space, not the layout or the sixteen biomes.
- `todo.md`, `human_advice.md`.

**What to build**

1. **The scene.** In menu F, the biome choice becomes this scene: the Earth from space, rotating,
   with the biomes to choose from. It must appear when a predation event ends (the respawn —
   that is what it is for). Using it also for the first choice when F starts and after a location
   revisit is your call; say what you chose. Menu E keeps its own biome choice unchanged.
2. **Rotation.** How the Earth turns is yours to choose (bg.png has the Earth baked in; a rotating
   part of the still, a slow pan, a shader — anything that reads as a turning planet). Say what
   you did and why.
3. **Biomes.** The choices are the biomes the game has (`BIOMES`: meadow and forest playable,
   wetland listed but not playable). Do not invent the other thirteen from the mock.
4. **State.** The discovered predators and locations survive the respawn exactly as they do now,
   and reset on a new game (entering F from the menu).
5. **Import.** `bg.png` byte-for-byte into the game repository with its identity
   `protoprey-refs@a7b4763:scenes/modes/biome_view/images/bg.png` beside it. The window is
   1280×720; frame the still sensibly.
6. **Tests.** Headless: the scene appears after predation, picking meadow and forest leads to
   exploring them, wetland cannot be picked, discovery state survives the respawn, the existing
   suites still pass. README/VERIFY as usual; commit and push `main` and `direction/`.

Breaking changes inside F are fine; no backward compatibility is needed.
