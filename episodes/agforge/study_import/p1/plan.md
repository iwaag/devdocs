# Study import p1 — knowledge to usable forge workflows

## Goal

Put mediagen's general knowledge into local practice and turn it into tools, workflows and guides that forge can reuse through ordinary requests. Start with the five GUI/HUD icons in `test_theme.md`, and demonstrate replacing part of the set after delivery.

## Implementation stance

- This is an experimental environment undergoing breaking changes. Backward compatibility and migration shims are unnecessary; remove obsolete implementation and guides.
- The implementer chooses the structure, technologies and experiment order. The steps below describe outcomes and hints, not a fixed execution route. Add agents, study types or general infrastructure when needed.
- forge discovers knowledge and chooses methods. Do not restrict it to documents selected externally; reduce the effort of browsing indexes, searching and reading instead.
- Keep local facts and credentials in ignored `.local/` files under the existing policy. Commercial services may be proposed as alternatives; distinguish proposals from new contracts or spending that has not been authorised. Do not add approval stages for ordinary experiments or rebuilding.

## step1 — Inspect the current state and define the trial

- Re-read the prerequisite documents and relevant implementation. Inspect mediagen's published knowledge, existing generation experiments, and forge's tools, grants and delivery route. Use nctl/Nautobot to understand current services, following `pj-clusterintent/nctl/README.md`.
- Record knowledge sources, revisions and what forge can currently access. Reuse existing experiments and tools.
- Specify icon dimensions, intended display size, transparency outside the background tile and delivery format. Make and record reasonable assumptions for unspecified details rather than waiting for unnecessary clarification.
- Focus p1 on static icons. Add hover/pressed/disabled button states if useful, but do not make them mandatory.

Done: the initial request, evaluation criteria, available knowledge and tools are clear.

## step2 — Establish the localisation work and its knowledge storage

- Start with autolab's existing project mechanism. Describe the purpose, layout and operation of localisation without publishing in `README_PROJECT.md`. The existing study contract keeps `main/` publish-ready; omitting publication alone does not make it a place for local facts.
- Arrange references to general knowledge, reusable code/workflows/guides, environment configuration and experimental evidence. Version reusable procedures and keep host-specific values separate.
- autolab owns experiments and implementation; cagent owns environment discovery and necessary environment changes; forge performs acceptance as the consumer. Coordinate through existing conversations and workplan/workrun routes.
- Retain limitations, failed methods, relaxed requirements, alternatives including commercial services, and their tradeoffs alongside successful procedures. Return generalisable findings to mediagen.

Done: the next worker can find the knowledge, execution methods, constraints and experimental results.

## step3 — Discover an icon production method

- Try promising methods from image generation, vector drawing, image processing and template composition. Exhaustively testing every method is unnecessary.
- The first candidate is a shared template for the rounded background, gradient and margins, combined with drawn or generated fruit. Consider generating the whole icon or drawing it entirely in SVG as alternatives. A method that uses no generative model is acceptable.
- Check whether all five fruits are recognisable, whether style/margins/scale agree, and whether the icons read at their intended display size. Measure execution time and manual correction effort; do not judge only large previews.
- Make promising procedures repeatable. Turn recurring mechanical steps into tools, and document parameters, editable sources, dependencies, usage and known limitations.
- If the requirements prove difficult, avoid endless attempts: explain the difficulty and propose changes that could help. Do not count changed requirements as fulfilment of the original request.

Done: a candidate procedure supports producing five icons and replacing one, or there is an evidence-based alternative proposal. A proposal alone does not satisfy the final delivery demonstration.

## step4 — Implement forge's knowledge access and execution handover

- Give forge access to mediagen's knowledge and the localisation outputs. Provide a lightweight index of uses, constraints, verification status and references to details. Make unverified or unimplemented entries recognisable as such.
- Let forge search and read during both planning and execution. It must remain able to explore other knowledge or methods after its initial toolset selection. Verify access under the actual role grants and PATH.
- Pass explicit references to project requirements. Make general knowledge, local knowledge, project requirements and individual assetplan/assetrun instructions distinguishable.
- Record the revisions of adopted references, the workflow and execution conditions. If sources change after planning, keep the actual execution traceable. There is no need to pin access to sources that have not been adopted.
- For asynchronous work, retain the submitted job's conditions and collection destination. Reuse the existing waiting, notification and resumption mechanism.

Implementation hints:

- `pj-agdev/agforge/src/agforge/toolsets.py`: currently lists, resolves and copies toolsets.
- `assetplan_topic.py` / `assetrun_topic.py`: planning-time selection and execution workspace construction. Currently record toolset names and place their files at execution time; preserve `plan.md` and `tools/` while waiting.
- `record.py` / `anchor.py`: request/run records whose authority is the conversation. Consider extending these before adding another ledger.
- `agent/guides/` / `agent/toolsets/` / `agents.toml` / `role_run.py`: decision guidance, usage information, role grants and execution environment.
- The image toolset already explains image generation and Pillow. `transform.py` supports resizing, format conversion and re-uploading. Start with existing tools where useful.
- `pj-agdev/agautolab/agent/project_pattern.md`: existing study/gentest contracts. Some toolsets, such as speech, have empty bodies; distinguish being listed from being usable.

Done: forge can find references, choose a method and carry the necessary information into ordinary planning and execution.

## step5 — Demonstrate the ordinary request route

- Run the `test_theme.md` request through ordinary `assetplan` → `assetrun`, delivering five individual files and a set preview. Check specifications, appearance and downloadability.
- Follow with requests to replace one fruit and change the background colour or output size. Check consistency of unchanged parts and reuse of editable sources.
- Repeat through a new request/run. Inspect discovery and method-selection records for dependence on a lucky first success or manual help from the Omni Agent.
- Add or update relevant automated tests, focusing on knowledge access, requirement/revision handover and delivery. If the chosen route is asynchronous, also verify resumption and duplicate notifications.
- Check responses to difficult requests: explain unmet requirements and propose requirement changes or other methods. Fix implementation failures and retry where feasible.

Done: ordinary requests support delivery, revision and reuse in a separate run. Document any work performed for an agent by the Omni Agent as a handoff candidate.

## step6 — Operational handover and report

- Write short update and re-verification instructions. It is sufficient to re-check affected capabilities when a model, workflow, dependency or environment changes, or a run fails. p1 does not require automated monitoring infrastructure.
- Document routes for environment problems to cagent, implementation problems to autolab and gaps in general knowledge to mediagen.
- Apply necessary deployment/service changes and update introductions and developer documentation. Commit and push changed repositories.
- In `report.md`, briefly record the adopted method, reference revisions, demonstrated requests and results, limitations and remaining work. Refer to `.local/` for environment-specific evidence.

Done: one capability has demonstrated knowledge import → environment adaptation → discovery and use by forge → delivery → revision and reuse, with a process that can be applied to the next capability.
