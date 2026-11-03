# improve_wfnode p1: node expectations, examples and list groups

Date: 2026-11-03. Plan: [plan.md](plan.md). Source: [braindump.md](braindump.md). Step reports: [1](report1.md), [2](report2.md), [3](report3.md), [4](report4.md), [5](report5.md).

## Result

**The completion criteria are met in the deployed system, with the limits below.** A person can attach expectations to a node as preferred or required, and author a variable list of example nodes with a named boundary. The list is a first-class group in stored definitions and run revisions. Real agents turned the same three examples into 0, 1 and 2 concrete tasks before delegating, and the run shows the group, its actual members, where they came from and which expectations were met.

The person has not reviewed it yet. All browser evidence is automated checks and my own reading of screenshots.

## What exists now

- **Groups**: `groups.<id>` with name, `kind: list`, `cardinality: variable`, description and ordered `members`, the sole authority for membership. Flat and non-overlapping; order creates no edges.
- **Examples**: `example: true` on any node. Not work: refused if started or cancelled, outside progress counts, and a run with one cannot read as completed. A run resolves it through an ordinary `wfe run adjust`, with `specializes` provenance on what replaces it and refusal when a dependency would be lost.
- **Expectations**: `expectations: [{key, strength, note}]`, with six documented keys, unknown keys kept and shown. `wfe run expectation` states Met, Unmet or Unverified with checked evidence, labeled as the agent's report. Preferred and not met reads Alternative. A required unmet stays visible on the finished run, and strength gates nothing.
- **Frozen with the run**: compiler v6 carries groups and each key's words.
- **Editor and run view**: boundary labelled `List · Variable`, group selection and movement, member editing, badges such as `Browser · Required`, expectations beside the result, source examples linked to their revision.
- **Catalog**: templates carry expectations and example status. The `present` template is now r3, with the browser assumption as an explicit preferred expectation.

Contracts: agdevworld `workflow_editor/docs/contract.md`, `runs.md`, `adjustments.md`, `catalog.md`.

## Revisions

| What | Revision |
| --- | --- |
| WFE, agdevworld (coordinator and gpupc) | `7fe52df`, previous `1053a73`; no store change |
| Executor, agautolab | `f067be0` (a reference note only), previous `1f20225` here and `96b5a32` on gpupc |
| pj-agdev pin | `263d2f4` |
| Catalog | `present` r3, generation 13 |
| Demonstration | `wfrun-trial`, workflow `scene-world`, runs 001, 002, 004, 005 |

## Checks

`npm run check` 768 of 768. All runs of all 22 workspaces replayed on the first new release before it was activated (665 usable, 0 refused). Four real-agent runs, one cancelled draft trial, and three browser checks (26, 15 and 7 checks, on an isolated service and on the deployed one). Details and the plan's nine verification items: [report5.md](report5.md).

The trials found three defects that the tests had not: agents did not state expectations, the command to state them could not work at all, and `wfe run show` crashed on the result. All three are fixed. Only the last run (run-005) shows the whole path working, so that path has **one** successful real-agent run behind it.

## Decisions I made where the plan left room

- Provenance is the node field `specializes: {examples, revision}`; no specialization helper was needed, and agents used `wfe run adjust` unaided.
- Losing a dependency through a removed example is a refusal, not a note.
- An added reminder, `expectation-unstated`, is justified by run-001 and run-002. It stops a completion once, and `--yes` passes.
- An example with an in-place resolution keeps its node id.

## Remaining limits

- A boundary is its members' bounding box, so a non-member placed between them appears inside it. Membership stays explicit in the data and panel, but the picture can mislead.
- `native`, `nfs` and `s3` express intent only, as the plan scoped: no process lifecycle or storage backend is added. Trials reported them honestly as Alternative.
- "Met" is the agent's statement. In run-005 it means an HTTP 200, not a look in a browser.
- `placeholder` (a catalog template by ponderer-agent) overlaps in purpose and was left alone: it is the project agent's own, and means "anything may replace this".
- Not exercised: `--artifact` evidence from a remote worker, a real agent reporting a required unmet, five items by a real agent, the catalog editor's new fields in a browser, phone width.
- Possibly broken, not changed: `wfe conversation workflow attach --commit`, by the same flag collision (read in code, not run).
