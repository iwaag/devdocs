# improve_wfrun p2 step 4: the workflow-definition handout

Date: 2026-11-02. Separate from the storage work: WFE code in its own commit, agdevworld `fb47811`; `docs/integration.md`.

## Who did what

| Fact | Who |
| --- | --- |
| Found `partial-animation` invalid | Omni Agent, in improve_wfrun p1 (handout) |
| Requested the repair and the first run | Omni Agent as the check actor `improve-wfrun-p2-check`, under this episode's plan |
| Implemented the repair | gimmickimg's `freework/run-019` (agent `autolab`) |
| Ran the workflow | gimmickimg's `partial-animation/run-001` (agent `autolab`) |
| Verified the definition and the run | Omni Agent, from the integrated definition, `wfe delegate targets`, the run's records and the shots; findings below are its own |
| Confirmed the run's result (q1) | `improve-wfrun-p2-check`. **Not the developer**: this checks that the workflow runs. Nobody has judged how the leaf looks as art direction |

## The correction

`freework/run-019`, published revision `d4d8ea9` → main `fcd191e`. The diff under `devdocs/workflows/` is one line in
`partial-animation.yaml`: node `review`, `type: review` → `type: talk`, keeping `confirms: result`, description and edges.
`wfe delegate targets gimmickimg` now says `partial-animation ready, valid: yes, requests: yes, channel: provisioned`.

**The repair run was not clean.** The first two attempts made and committed the edit, then exited without recording a node, so the
run was held as interrupted. `agops` recovery resumed it twice, and the third attempt recorded the nodes afterwards
(`run.record-done fix-definition`). Its `talk`, `present` and `review` nodes say nothing was needed. What I can say about the cause:

- **A WFE defect, fixed here**: `wfe validate` inside a run's copy read the registered workspace, so both agents could not validate the very fix they
  were asked to validate, and they spent their attempts on that. `wfe validate` now reads the copy (test in `runrecord.test.ts`).
- **Not diagnosed**: an agent ending with no node recorded. It recurred on `wfrun-trial stamp/run-005`, an agent-style `--request`
  of one word. It did not recur on `partial-animation/run-001`, also a `--request`, so the request kind alone does not explain it. Recovery handles it; the cause is open.

## The first run

`partial-animation/run-001`, definition and code from the published revision `fcd191e`, on `studio`; integrated by fast-forward to main `af2fa0f`.
It ran as authored: talk, analyze, edge-refine, clean-plate, build-layer, verify, present, review, cleanup, final report. The freework scene of `run-018` is not
counted, and the scene was not rebuilt: the request was one leaf.

Output: `scenes/leaf2_tip/recipe.json` and `experiments/partial-animation/leaf2_run/`, on main.

What I checked myself rather than read from its report:

- Recomputed from its shots: 12475 pixels change by more than 2 levels between phases, bounding box x 1006–1200, y 285–430. These equal its numbers.
- Pixels outside that box changing **at all** (threshold 0): 154. Its "0 changed" is at its threshold outside its dilated mask. Not a contradiction, but
  not literally nothing moving; these are at most 2 levels.
- Mean dimming ratio of the whole picture 0.546 at 0.55, consistent with its 0.548.
- Opened the presentation (HTTP 200 while it served), viewed `phases.png`: the leaf moves, a clean plate lies beneath, and it looks plausible at that size. After the run,
  the page no longer answers, as its cleanup says.

Limits it states: a slightly soft cut-out, Forward+ not tested, and no earlier revision to compare. Not checked by me: the animation in motion, and any embedding in another project.

## WFE side

- **At integration** (`server/definitionCheck.ts`): workflow files the integration added or changed are validated at the revision that reached the default branch and against
  published peers, not the editing workspace. `run.integrate` records `definitions` as `valid`, `invalid` with each error, or `incomplete`. The timeline and a requester's assessment say it.
- **Before that**, the end-of-work advice names a changed workflow that does not validate in the copy, while the agent can still fix it.
- **No gate**: the integration still happens. `incomplete` (peers or revision unreadable, no published source) is stated as no verdict.

Tests: `publishedDelegation.test.ts` (the `confirms-type` case against a stale workspace, a valid one, unavailable peers), `integration.test.ts` (the three statements; only changed files), `runrecord.test.ts`.
Full suite in release builds: 694/694 at `fb47811`.

The live `partial-animation` correction predates this release, so no live integration has yet recorded an `invalid` result; only fixtures show that path. Later live
integrations that changed no workflow correctly record nothing.
