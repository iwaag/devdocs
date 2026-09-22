# Adventure game p2 — Human-authored references guide production

Source: [braindump.md](braindump.md). Previous evidence: [p1 report](../p1/report.md).

## Goal and approach

Let the human edit a dedicated repository of stories, images, templates and runnable examples. Give Front, autolab, forge and archsage direct access to the same published revision, preserving its folder structure. Demonstrate that these references improve one scene of the existing game, then carry a human revision through another production pass.

- Keep the human's authoring workspace separate from agent workspaces. Use `main/` for production and `direction/` for interpretation, scope and decisions. Human-selected generated assets may also become references; distinguish authorship from adoption.
- Preserve the published originals and their identity. Agents may question them, propose alternatives and create derivatives in their own workspaces. The human decides creative acceptance and changes to the intended experience; silence is not approval. Continue independent work while judgement is pending.
- This is a private experimental environment and a breaking phase. Backward compatibility is unnecessary. Implementers may replace existing structures, interfaces and disposable state, and choose tools, storage details and work order. Avoid broad security hardening, blanket non-destructive rules, repeated approvals and a fixed workflow engine. Keep secrets and host-specific facts in ignored local files under the existing policy.
- Minimize human maintenance: start with ordinary files and a short README, not a mandatory specification format or per-file metadata. A few representative examples should guide many outputs.

## step1 — Inspect the existing paths and settle the reference contract

- Read the prerequisite documents, p1 reports and relevant code. Check current service state and observation timestamps through Nautobot or `pj-clusterintent/nctl` before relying on the local environment.
- Inspect project setup, forge knowledge access, archsage knowledge sync and the role tools that actually read files and images. Identify where production requests lose references between agents.
- Define the smallest shared reference identity: repository or stable source ID, immutable revision and relative path. Record which revision a production unit adopts; a running task keeps that revision until explicitly replanned.
- Choose where the human edits, how a revision is published and how each consumer obtains it. Prefer the existing Git hosting and per-machine copies; no shared absolute path is required. A small shared resolver is an option, not a prerequisite.

Done: the implementation points and the authoring → publication → consumption path are concrete enough to build.

Hints: `agautolab/agent/project_pattern.md` already allows extra folders and external repositories. ProtoPrey's `direction/` currently contains only `.gitignore`. Forge's `knowledge.py` reads configured local trees and records revisions, but does not fetch pinned snapshots; archsage's `sages.py` currently clones or fast-forwards a study tree.

## step2 — Provide a human authoring repository and revisioned delivery

- Create a dedicated repository and a human editing workspace, separate from autolab's generated project repositories. Document editing, publishing and retrieving a previous version in a short README. Use commit/push as the initial publication boundary unless a simpler equivalent emerges.
- Provide consumers with local copies of the selected revision, retaining relative paths and all referenced assets. Keep editing drafts out of published snapshots. Ensure image bytes remain available after temporary delivery links expire.
- Establish a practical ownership boundary: agents consume originals and write derivatives elsewhere. Use repository-scoped read access where practical; do not turn this into an isolation project. Same-user file permissions alone are not an ownership boundary.
- Record the source relationship in the project workspace. Keep environment-specific URLs, credentials and cache paths in the appropriate local configuration. Choose large-file handling only if the actual examples need it.

Done: an independently prepared consumer workspace can retrieve the exact published files and tree without access to the human's working folder.

Hint: p1 lost the human-approved meadow image because it was never imported and its download expired. Persist adopted files themselves, not only conversation links. A remote-capable design should be checked from another node when available; otherwise report the cross-machine check as pending.

## step3 — Carry references through planning, generation and research

- Make the source discoverable to Front, autolab, forge and archsage, with concise usage information and suitable tools. Reuse or extend existing readers and sync mechanisms where useful; keep creative references distinguishable from general research knowledge.
- Carry source identity, revision and relevant paths from the project request into workplans, delegated tasks, asset requests and results. Workers must be able to inspect originals rather than relying solely on another agent's summary.
- Support the actual media: reading text, browsing the tree, viewing images and, where the chosen generation method supports it, supplying reference images to that method. Treat executable examples as inputs to try in a working copy.
- Put interpretations and production decisions in `direction/`, derivatives and implementation in `main/`, and execution evidence in the ordinary work records. Record departures from the reference and unresolved conflicts clearly. Do not silently substitute a new reference or claim to have read unavailable material.
- Give agents discretion over methods and relevant files. Clarify human-specified intent when necessary without requiring approval for routine implementation choices.

Done: each participating role can retrieve and use a named reference revision, and a delegated result identifies the reference it used.

Hints: forge's `knowledge show` currently reports binary files without showing their content; returning a path alone does not prove visual access. Its existing knowledge precedence rules concern technical guidance and need deliberate integration with creative direction. Archsage's study tree should remain distinguishable from project-specific creative inputs.

## step4 — Produce one scene from human examples

- Ask the human to select one scene from the existing game and supply the smallest useful set of references: for example prose, a visual composition and optionally a playable sketch. Explain what each example should establish, such as tone, framing, pacing or interaction. Do not invent the human's examples or acceptance.
- Agree on the scene's intended experience and evaluation points. Have agents explain their interpretation and surface material ambiguities before expanding production.
- Run the ordinary Front/autolab/forge workflow, involving archsage where research is useful. Build one playable scene using the published references and record which elements were reused, transformed or newly created.
- Verify launch, assets, interaction and reference provenance. Present the playable result and relevant comparisons to the human, keeping technical verification separate from creative acceptance.

Done: the human can compare one playable scene with their originals and say where the interpretation succeeds or fails. Pending human evaluation remains incomplete.

## step5 — Carry a human revision through the next production pass

- Have the human edit or replace a reference and publish a new revision in response to the scene. Show the changed files and let agents identify affected plans, assets and implementation.
- Adopt that revision for the next production unit and revise the affected work. Preserve the identity of the earlier result; unrelated completed work need not be regenerated.
- Deliver the revised scene and obtain the human's assessment of whether it moved closer to the intended experience. If they choose to stop, record the demonstrated scope rather than inventing a completed iteration.

Done: a human edit has travelled through the same project's planning and production into a human-evaluated revision.

## step6 — Verify the mechanism and report the outcome

- Run focused checks for revision selection, tree and asset fidelity, independent consumer retrieval, and reference propagation across delegation. Verify actual media access and a revision change through the participating roles; a successful clone alone is insufficient.
- Report the human's evaluation, remaining gaps, human effort, agent cost where available, and any lost or misinterpreted references. Record Omni Agent work performed for in-system agents as handoff candidates.
- Update relevant guides and environment notes, and write `report.md` with source revisions, production versions and supporting conversations. Separate what the one-scene trial proves from untested large-scale or long-story production.
- Commit and push changed repositories and necessary submodule references. Add further infrastructure only where the trial supplies evidence that it helps.

Done: the reusable authoring and consumption path is documented, checks are recorded, and the creative outcome is stated in the human's terms.

## References

- `devpolicy/styles.md`, `devpolicy/terms.md`, `devdocs/README_DEV.md`, `localrule.md`.
- `pj-agdev/.local/devenv.md`, `pj-clusterintent/.local/localenv_memo.md`, `pj-clusterintent/nctl/README.md`.
- `pj-agdev/agautolab/agent/project_pattern.md`, `pj-agdev/agautolab/src/agautolab/project_init.py`.
- `pj-agdev/agforge/src/agforge/knowledge.py`, `archsage/src/archsage/sages.py`.
- `devdocs/episodes/agforge/study_import/p1/report.md`, `devdocs/episodes/milestones/adventure_game/p1/report4.md`.
