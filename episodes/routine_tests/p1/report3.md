# Step 3 — what the run did, verified against the artifacts

All three stages ran and produced verified work, from one request, in order,
with **no manual stage-start nudge**. Every stage was also damaged on the way
by the same defect, and five manual interventions were needed to keep the
record intact. Both halves of that are the result.

Elapsed: **12:07:03Z → 12:37:09Z, 30 minutes 6 seconds.**

## Timeline

| Time (UTC) | Event |
|---|---|
| 12:07:03 | Request (6090) |
| 12:07:39 | Proposal (6092) — scope narrowed to two projects |
| 12:08:06 | Accepted (6093) |
| 12:08:29 | Stage A run opened (6096); started 12:08:32 |
| 12:09:18 | autolab plans `studyarxiv` |
| 12:09:46 | **Premature finish #1**; report delivered (6104); **continuation fires** |
| 12:10:08 | autolab plans `studyrealworld` |
| 12:10:56 | Callback forks **twin A**; Front reports status (6111) |
| 12:12:54 | `work-m6113 › workrun-task1-m6113` starts |
| ~12:14 | *Intervention 1* — twin A's origin anchor restored (6131) |
| 12:17:47 | autolab reports (6137): `main` `1563bba`, `publish` `8901b34` |
| 12:18:28 | Stage A ends `achieved: true` (6142); **continuation fires** |
| 12:19:02 | Stage B run opened (6149); started 12:19:06 |
| 12:19:56 | autolab plans `papers` |
| 12:20:02 | **Premature finish #2** (6155); **continuation fires** |
| ~12:20:2x | *Intervention 2* — stage B anchor pre-placed (6170) |
| 12:22:39 | Callback serves B through the twin |
| 12:23:52 | `work-m6162 › workrun-task1-m6162` starts |
| 12:25:42 | *Intervention 3* — looping finished workplan topic closed |
| 12:26:43 | Paper commit `8829a97` |
| 12:28:08 | Stage B ends `achieved: true` (6195); **continuation fires** |
| 12:29:02 | Stage C run opened; started 12:29:07 |
| 12:30:15 | autolab plans `workplan-publish-2609.10712` |
| 12:30:24 | **Premature finish #3** (6208); **continuation fires** |
| ~12:30:3x | *Intervention 4* — stage C anchor pre-placed (6213) |
| 12:31:42 | Callback serves C through the twin |
| 12:32:56 | `work-m6215 › workrun-task1-m6215` starts |
| 12:33:09 | **Premature finish #4** (6227); **continuation fires** |
| ~12:33:3x | *Intervention 5* — stage C anchor re-placed (6233) |
| ~12:33:44 | `main` `24cc19a`, `publish` `cac3527` |
| 12:35:38 | `front` role posts a finish block **into the run topic** (6239) — not parsed |
| 12:35:56 | `start_opened_runs` starts the bare topic as a run |
| 12:37:09 | Stage C ends `achieved: true`, delivered, resolved |

## Coverage

Measured against the step 1 baseline, not against Front's claim.

| Project | Candidates | Delegated? | Outcome |
|---|---|---|---|
| `studyarxiv` | 16 papers | yes | Already current; `diff` empty; no commit — a correct no-op |
| `studyrealworld` | 25 files / 6 sources | yes | `main` `1563bba` pushed; `publish` `8901b34`, not pushed |
| `ghtrends` | 8 of 10 summaries + index | **no** | **Still unpublished** |
| `mediagen` | whole tree (`publish/` has no commit at all) | **no** | **Still unpublished** |

**The user's request was not fully satisfied.** "Across all study projects"
covered half the projects. This is not a gate rejection and not a blocked
item — the two projects were never looked at, because the guide's table said
there were only two and that outranked both the rule above it and the request.

`studyarxiv` reporting nothing to do is the correct empty-backlog no-op the
plan anticipates, and it was proceeded past without an empty publication
commit, as required.

## Order

Message ids and timestamps show strict ordering, and this survived every
premature finish:

- Stage B opened at 12:19:02, after stage A's `achieved: true` report landed
  at 12:18:28. B's opening post names A as done.
- Stage C opened at 12:29:02, after B's `achieved: true` at 12:28:08, and
  after B's source commit `8829a97` existed (12:26:43).
- After each premature finish Front was asked again and explicitly held the
  line — "I'll report again once autolab's plan lands and stage 2 progresses,
  **then** move to stage 3" — rather than treating an ended run as a finished
  stage. It never jumped ahead.
- Front performed the task-start action autolab's introduction requires, in
  each `workrun-` topic, and accepted each result before moving on.

## The one new investigation

Compared against the index snapshot taken before stage B.

- Index went **16 → 17 rows**; exactly one paper added.
- `2609.10712`, *An Open Recipe for IMO Gold: Training Nemotron for Olympiad
  Mathematics* (NVIDIA). Submitted **2026-09-09**, three days before the run —
  well inside the guide's 30-day window.
- Public signal named with evidence: Hugging Face Daily Papers, 2026-09-11
  snapshot, ~20 upvotes.
- **The paper was read.** The summary carries paper-internal specifics that
  exist nowhere else — 414,890 filtered SFT examples, 512 GB200 GPUs, an
  asynchronous RL run over 9,597 selected problems.
- Verdict `runnable: unclear`, with a stated reason (open code/data/weights,
  but 550B parameters at cluster scale with no documented single-machine
  path). Under the guide only `runnable: yes` earns a manual, so **no manual
  was due**, and autolab separately confirmed all eight prior `yes` rows
  already had manuals — no backlog was skipped.
- One commit, `8829a97`, carrying `papers/2609.10712/summary.md` and the
  `INDEX.md` row together, pushed to origin.

## Publication correctness

Checked directly against the trees, not from the reports.

`studyrealworld` — I re-ran the comparison myself: `diff -r main/sources
publish/sources` and `diff -r main/reports publish/reports -x README.md` both
came back empty. 25 files published. Six files were fixed in `main/` **first**
(two missing citation-metadata fields, two instruction-phrased Question
sections reworded into questions, two instances of internal "pull" wording) —
check-2 and check-4 failures, fixed rather than deleted, which is the guide's
"a check never deletes knowledge" rule honoured. autolab reported the six
subagents ran genuinely concurrently, with out-of-order completions and
overlapping 49–104s durations.

`studyarxiv` — every published file byte-identical to `main`, `README.md` and
`LICENSE` excepted; 17 published papers and 17 matching README table rows.
Stage C's one fix was a check-2 provenance line rewritten in `main/` to the
project's standard *"First posted …; current version v1"* form, committed as
`24cc19a` and pushed, then copied unchanged.

**Repository outcome.** `main` pushed in both projects and level with origin.
Both `publish/` clones committed locally and **left unpushed**, each exactly 1
commit ahead of its origin — `8901b34` for study-realworld, `cac3527` for
study-arxiv. Both were even with origin before the run, so no already-ahead or
behind clone had to be reported, and nothing was reset, merged or pushed. Front
told the developer about both pending manual pushes, unprompted, in its final
wrap-up.

## Continuation and closure — what the step 1 repair bought

The handoff fired **six times** and worked every time, on successful and
unsuccessful completions alike. Each firing is a matched pair in the log:

```
delivered the run's report to front/front-routines-20260912T1212Z
continuing front/front-routines-20260912T1212Z: a run reported there and nothing has served it
```

All three runs ended exactly once, each report reached the right parent, and
the final wrap-up (message 6240) reached the original request naming all three
stages and both unpushed commits. Completing a child left the parent available
for the next stage, and the routine guides and unrelated runs were untouched.

**No stage was started by a nudge from me.** Stage B and stage C were each
opened by Front, out of the conversation, after the previous stage's report
landed — which is precisely what could not have happened before step 1's fix.

## Defects found

### 1. Coverage narrowed by a stale guide table

Described above. The guide's own opening rule and the developer's words both
say "any study project with a `publish/` folder"; the table's "Today there are
two" won. Owner: the `publish` guide (message 5496).

### 2. A run ends itself while its work is in flight — 4 times in 3 runs

Every run's **first** serving wrote an `ag-routinerun` block seconds after
opening a delegation, and stage C did it twice. Each block was
self-contradicting on its face:

- *"…so the run continues."* / *"Not applicable yet — the run has not ended."*
- *"…the run is not done until autolab reports."*
- *"Not final — stage 3 still in progress."*

The block is what ends a run, so the listener did exactly what it says:
delivered the report and resolved the topic. The `routine_run` guide already
forbids this — *"Do not write the block while a delegation is still in flight"*
— so **guidance alone is demonstrably insufficient**: four instances, one
trial. The role appears to read `achieved: false` as "I have not finished yet"
rather than "this run is ending without its goal", and the guide's own word
"finish" is overloaded between *finish the serving* and *end the run*.

### 3. A callback into a finished run forks a twin and loses its origin

The downstream damage of #2, and a genuine code defect. `handle_mention`
resolves home from the root note, which records the run's **bare** name;
`serve_topic` then reads that bare name, finds nothing because resolving
renamed the topic to `✔ …`, and posts — creating a second topic beside the
resolved one. The twin does not carry the `[selfnote][rootchat]` origin note,
so `origin_of` returns `None` and the run's eventual report would be recorded
inline and **never delivered to the requester**, stalling the composite request
with the developer told nothing.

It is self-healing in one respect: resolving the twin merges it back into the
`✔` topic. But that only happens if the run finishes again.

### 4. A finished run keeps a delegate talking

Stage A's twin kept the `pj-studyarxiv › workplan-publish-20260912`
conversation alive after the work was done: three further rounds in which
autolab asked what "stage 2" was and Front answered, each a paid run on both
sides, over work already complete. Closed by hand at 12:25:42 (intervention 3).

### 5. Continuation recovery does not run on queue re-registration

Front's event queue expired at 12:30:57 and re-registered with a full sweep.
The full sweep does not ask for pending continuations — only process startup
does (`recover_runs`). A handoff owed across a queue expiry would therefore
wait for a process restart. Not hit in this trial, found by reading the log
beside the code.

## Interventions — this attempt was assisted

Five, all state repairs or housekeeping; **none started a stage**.

1. ~12:14 — restored twin A's origin anchor (6131).
2. ~12:20 — pre-placed stage B's anchor (6170).
3. 12:25:42 — resolved the looping finished workplan topic.
4. ~12:30 — pre-placed stage C's anchor (6213).
5. ~12:33 — re-placed stage C's anchor (6233).

1, 2, 4 and 5 are the same repair: writing `[selfnote][rootchat]` as Front into
a run topic's bare name so defect 3 could not strand the report. Front has no
way to do this itself — it is told never to post into a run it opened, and
`agentchat` writes a root note only when posting into *another* agent's topic —
so only an outside actor could undo that damage.

Intervention 4/5 had a side effect worth recording: the pre-placed anchor made
the bare topic look like a run the conversation had opened, and at 12:35:56
`start_opened_runs` started it as one. Combined with Front's `front` role
posting a finish block directly into the run topic at 12:35:38 — itself a
guide violation, and unparsed because `split_finish` only reads a run
serving's own output — this produced one extra round. It converged on its own
at 12:37:09.

**Deus Ex Machina note:** *did the run-topic origin-anchor repair for agent
Front — handoff candidate.*

Because of these, **the first attempt is assisted, not autonomous**. What is
proven autonomous is narrower and still substantial: the composite request
stayed alive across three child completions and started stages B and C by
itself, six times over, with no nudge.
