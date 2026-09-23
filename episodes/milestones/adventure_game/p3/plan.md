# Adventure game p3 — Produce locations from human references

Source: [braindump.md](braindump.md). Previous evidence: [p2 report](../p2/report.md).

## Goal and working approach

First implement discoverable, revisitable locations using the human's examples. Then run a separate production task: forge creates one new forest location and autolab integrates it. Demonstrate reference-led creation and delivery, and measure the effort required to repeat it.

- This is a private experimental environment and a breaking phase. Backward compatibility is unnecessary; implementers may replace code, interfaces and disposable state. Choose implementation details and tools freely. Add infrastructure or constraints only where the trial shows a concrete need.
- Use the ordinary in-system workflow for production. Omni Agent work should concentrate on observation and missing capabilities; record any work performed for an agent as a handoff candidate.
- Human references remain the human's originals. Put derivatives in production workspaces and record the adopted source revision. Human creative acceptance is recorded in their words, separately from technical verification.
- Keep secrets and machine-specific details in ignored local files under existing policy. Routine implementation choices do not need repeated approval.

## step1 — Confirm the inputs and prepare the two tasks

- Read the prerequisite documents and p2 findings. Check service state and observation timestamps through Nautobot or `pj-clusterintent/nctl` before relying on local services.
- Inspect the current production checkout and published references through `agrefs`. Select one published revision for this trial and carry its identity and relevant paths into both tasks.
- Prepare an autolab task for the location feature, followed by a distinct forge-to-autolab task for one new forest location. Define the small asset handoff together with the implementation; no general content pipeline is required.

Done: both tasks have concrete inputs, expected outputs and reference identities.

Hints: at planning time, published `protoprey-refs@6e017fb` contains `flower_pond` and `mashroom_waterfall` in forest, plus `little_burrow` and `mound` in meadow. Each has a view image and `texts/discovery.md` / `texts/revisit.md`. Read `human_advice.md` and `todo.md`. The waterfall uses `image/` rather than `images/`; map such differences during import instead of editing the originals. p2 proved reference retrieval and adoption, but not forge's reference-led production on a real request.

## step2 — Implement locations using the human examples

- Have autolab implement exploration, first discovery, a discovered-location list and revisits. Use the four supplied locations, enable meadow exploration, and keep forest's wolf interaction working.
- Make locations data-driven: stable ID, display name, biome, image, discovery text and revisit text are sufficient initially. Choose the file format and layout. A further location should require only content and registration data, without new gameplay branches.
- Keep discovered locations across biome visits and respawns; reset them for a new game. A disk save system is outside this trial. Choose a simple discovery policy that makes all supplied locations reachable and easy to verify.
- Reuse the existing image/text presentation where helpful. Ensure long discovery prose is readable and images display appropriately. Leave map navigation for later; a list or buttons can use the same location IDs and discovery state.
- Import adopted assets into the game repository, and record references and any adaptations in the project's ordinary documentation.

Done: the human examples are playable through first discovery and revisit, with a concise example of how to add another location.

Hints: `scripts/forest_flow.gd` currently hardcodes the playable forest and wolf sequence. `scripts/event_player.gd` already loads image/text steps but uses predator-specific titles. `scripts/main.gd` has a shared presentation and a wolf-specific discovery indicator. Refactor or replace these as useful; preserving the old demo architecture is unnecessary.

## step3 — Verify the feature before creating new content

- Check first discovery, one list entry per location, revisit text, biome association, return to exploration, persistence across respawn and reset on a new game. Exercise the existing wolf discovery and predation loop as well.
- Use focused state/data tests and a real game pass. Verify missing asset references, long text and image framing; avoid tests tied only to internal implementation details.
- Deliver a playable revision with the hand-authored examples for human assessment. Record functional findings separately from creative feedback, and address issues that would obstruct the new-content trial.

Done: the location mechanism and handoff format are demonstrated, providing a stable basis for the separate content task.

## step4 — Have forge create one new forest location

- Give forge the pinned references, the implemented content example and the delivery shape: one view image, English discovery and revisit texts, location identity and a short provenance note. Let forge choose the new place and production method.
- Have forge inspect the actual images and prose, including embedded generation metadata where useful. Aim for the references' low viewpoint, miniature bodily scale, atmosphere and consistency between image and text. Revisit prose should convey familiarity rather than repeat the discovery.
- Use the human's Flux2 guidance as the starting point for scenery. Inspect `tools/image_flux2_text_to_image.json` and the PNG metadata: the inspected forest examples contain ComfyUI prompt/workflow data, Flux2 dev settings and 1344 × 768 dimensions.
- Check whether forge's available tools can execute the chosen route. Its existing image CLI uses SwarmUI, and p2's `--init-image` support does not establish a ComfyUI image route. If a capability is missing, provide the smallest useful tool or adapter and usage guidance, then let forge perform the production. Image-to-image is optional.
- Have forge inspect and refine its output, then deliver the complete package with reference revision, generation settings and notable departures. Record attempts, elapsed time and cost where available.

Done: forge has delivered a new location package through its ordinary work record, ready for autolab to consume. This is an agent delivery, not a substitute creation by the Omni Agent.

## step5 — Have autolab integrate and demonstrate the new location

- Pass forge's delivered package to autolab through the ordinary workflow. Store the actual files in the game repository so the result survives expiry of delivery links.
- Add the location to forest using the established data format. Record any engine changes or asset repairs still needed; these reveal whether the first implementation supports repeated additions.
- Play discovery and revisit of the new location alongside the human examples, and run the relevant regression checks. Present the playable revision and comparison to the human for creative evaluation.
- Address feedback through the responsible agents. Record human edits, Omni Agent intervention and additional generation attempts so the production effort stays visible.

Done: forge's location is playable and its technical result and human evaluation are recorded. Pending human evaluation remains explicitly pending.

## step6 — Report what the trial proves

- Write `report.md` with reference and production revisions, supporting work conversations, verification results and the human's assessment.
- Report production time, generation attempts, agent cost where available, human corrections and Omni Agent interventions. State whether adding the location required gameplay code changes.
- Distinguish a successful one-location production cycle from proven production at scale. Identify the observed bottleneck and the smallest useful next experiment or improvement.
- Update reusable guidance only where findings justify it. Commit and push implementation, documentation and necessary submodule references under the existing repository workflow.

Done: the reference → forge → autolab → playable location path is evidenced, and its quality and repeatability limits are clear.
