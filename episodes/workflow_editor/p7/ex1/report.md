# Workflow editor p7/ex1 — report

## What was asked

The braindump: whether an activity is its own node or part of another is a matter of the designer's preference. `improve-game/run-004` showed that an agent cannot know the person's preference: research was requested, but no study node appeared and the research left no traceable evidence. The person should design the nodes they expect once, and agents should be told so (Tool Giving, not forced), together with how to add such a node.

## What exists now

- **User nodes** (agdev dashboard → User nodes; `wfe nodes`; `/api/nodes`). The person's catalog is `<area>/node-catalog.json`. Each template has an id, a revision, a name, an existing type (`study`/`do`/`talk`), a description (for the agent performing it), an expectation (for whoever designs the workflow) and an optional appearance. Writes are validated and atomic, refused when stale, and ids are never reused. An unreadable catalog is reported and never replaced. The p7 presets moved into the catalog at its one-time initialization. *Preliminary research* is offered as an example, never added by itself.
- **Templates in workflow editing.** *Add node* offers the person's templates beside the basic types. A template becomes an ordinary node with copied content, appearance and provenance (`template: {id, revision, expectation}`), which the inspector and the run view show. Provenance is outside the digests and the compiled graph. Catalog edits never change workflows or runs. `wfe nodes instantiate` and the HTTP route give agents the same operation.
- **Runs get the catalog as it was at creation**: a hashed snapshot, an explicit empty one, or a visible "unavailable". Every attempt's brief states the meaning and lists the expectations, and `attempt.begin` records the snapshot. `wfe run nodes`, `wfe run show` and the run view's *Node catalog* show what a run was given. Runs from before ex1 say "none supplied".
- **Adjustments from the original request.** The source is `request`, bound to the input file and the hash recorded at creation, and every existing fence still applies.
- **The common guide** (agautolab) states the catalog, the request source, and that requested research leaves traceable evidence in the reports. Both new paragraphs reference `improve-game/run-004`.

## Steps

| Step | Report | Commits |
| --- | --- | --- |
| 1 Contracts | [report1.md](report1.md) | agdevworld `54ac34b` |
| 2 Catalog and editor | [report2.md](report2.md) | `fbd8cac` |
| 3 Templates in editing | [report3.md](report3.md) | `e514bf6` |
| 4 Delivery to run agents | [report4.md](report4.md) | `2285682` |
| 5 Request-sourced adjustments | [report5.md](report5.md) | `6058c0f` |
| 6 Verification and release | [report6.md](report6.md) | release `6058c0f`; agautolab `f1c466d`; pj-agdev `0047974` |
| 7 Agent trial | [report7.md](report7.md) | isolated; records in ignored storage |

## Verification

- Editor tests: **211/211** (20 new tests, 2 updated).
- autolab executor tests: **31/31**.
- Browser checks: `nodes` 20/20 (new), `appearance`, `review`, `runs`, `exec`, `conversation`, and `docsteps`, which passes again.
- Live after the release: all 8 runs usable, the catalog created through agdevworld's route, and the User nodes page and run-004's "none supplied" checked in a browser.
- Trial: with run-004's request, the agent added the research node from the request, recorded 7 sources and its choice before implementing, and linked them from the implementation report. With existing research work it added nothing. For a light reference it integrated the research and said why.

## Limitations and open points

- The person's catalog holds only the seeded presets. Their own templates, including a research template in their words, are still to be written.
- A new live RTS run with the person's own request and answers has not happened. Step 7 is isolated evidence with one stand-in answer, labelled as such.
- In trial C the integrated "light reference" cites no source (report7).
- In the browser, a template node is added unconnected. Placement (`--after`/`--before`) is available through the CLI and API.
- Rollback to `cde57cd` loses the readability of runs that recorded a request-sourced adjustment, and needs the guide reverted too (report6).

## Departures from the plan

- The presets left the palette entirely and live only in the catalog (the plan asked to move them; nothing built-in remains).
- `template.expectation` is copied into the node's provenance, so a node can still be explained after its template changes or is deleted.
- The common guide reached the live executor only together with the release, not when step 4 was written, because wfexec reads it at every claim.
- The trial's research template is the offered example, since the person has not designed one.

## Deus ex machina

- did the review-stage cleanup of an isolated trial run (stopping its presentation by cancellation) for agent autolab (wfrun) — handoff candidate.
