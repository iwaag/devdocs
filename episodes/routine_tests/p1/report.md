# routine_tests p1 — one composite request, three routines, five defects

## The request and how it was read

The braindump asked for a harder routine instruction, to find where the system
breaks. One request went in at Front's ordinary entrance
(`#front` › `front-routines-20260912T1212Z`, message **6090**,
2026-09-12T12:07:03Z):

> Across all study projects, publish everything that has not yet been
> published. After all of that is finished, investigate one arXiv trend item,
> and publish that result too.

Nothing about stages, counts, projects or routine names was given. Front
proposed three stages, was accepted at 12:08:06Z, and ran them to completion
by **12:37:09Z — 30 minutes 6 seconds**.

**"Publish" means the existing routine meaning and nothing more**: pass the
publication gate and commit `publish/` **locally**. The guide reserves the push
for the developer, and both `publish/` commits produced here are sitting
unpushed, one commit ahead of their origins. Remote publication is still
awaiting the developer:

- `studyrealworld` — `8901b34`
- `studyarxiv` — `cac3527`

## What was completed

| Stage | Work | Source commit | Publication commit |
|---|---|---|---|
| A | `studyarxiv`: already current, 16 papers, no commit — a correct no-op | — | — |
| A | `studyrealworld`: 25 files, 6 sources, 6 gate fixes | `1563bba` (pushed) | `8901b34` (local only) |
| B | One new paper `2609.10712` | `8829a97` (pushed) | — |
| C | That paper into `studyarxiv`'s `publish/` | `24cc19a` (pushed) | `cac3527` (local only) |

Verified against the trees rather than the reports: `diff -r` empty for both
`studyrealworld` trees; all 17 `studyarxiv` published papers byte-identical to
`main` with `README.md` and `LICENSE` excepted, and 17 matching README rows.

**The one investigation** is `2609.10712`, *An Open Recipe for IMO Gold:
Training Nemotron for Olympiad Mathematics* (NVIDIA) — submitted 2026-09-09,
three days before the run; flagged by Hugging Face Daily Papers on 2026-09-11
at ~20 upvotes; not previously indexed; index 16 → 17 rows; verdict
`runnable: unclear`, so no manual was due and the eight prior `runnable: yes`
rows were confirmed already to have manuals. The summary carries
paper-internal detail (414,890 SFT examples, 512 GB200 GPUs, 9,597 RL
problems) that only reading it supplies.

**Backends, from the run records:** every run on both agents was `claude_code`
on `anthropic/claude-sonnet-5`. Front spent 22 runs / **$5.4178**; autolab 9
runs (6 `superdirector`, 3 `supercoder`) / **$3.9846**. The records carry no
`duration_seconds`, so no per-run timing is claimed; the timeline in
[report3.md](report3.md) comes from message timestamps and listener log lines.

## What was not completed

**Two of the four study projects were never looked at.** `ghtrends` (8 of 10
summaries plus its index) and `mediagen` (whose `publish/` clone has no commit
at all) are still unpublished. This is not a gate rejection and not a blocked
item — they were never delegated, because the publish guide's table said
*"Today there are two"* and that outranked both the guide's own opening rule
("serve any study project that has a `publish/` folder") and the developer's
"all study projects".

So **the user's request was half-satisfied by the trial**, and the cause is a
stale line in a shared contract rather than any agent's reasoning about the
work it was given. The follow-up run ([report4.md](report4.md)) closed that
gap: `ghtrends` and `mediagen` are now published too.

## Prerequisite fixed before the trial

One, and it was demonstrated by inspection rather than suspected
([report1.md](report1.md)). A run's report is delivered into the requesting
conversation using Front's own credential, so that conversation is left with
Front as its last speaker — and `sweep_topics`, the event path's re-check, and
`sweep_rootchats` all skip exactly that. A one-routine request ended there
happily; a request that asked for anything *after* the routine had nobody left
to notice, and its next stage would never start.

Fixed in `agfront` `d596f7f`: the delivery is followed by
`[selfnote][delivered] <run>` in the requester's conversation, and
`continue_deliveries` asks the chat "has a report landed here that nothing has
served?" after every run serving, on both routes, and at startup. The handoff
is generic — it says only that somebody should look; what to do next is
decided by Front in the conversation. Ten regression tests, verified to fail
without it.

**It worked, six times, on its first live outing** — on successful and
unsuccessful completions alike. Stages B and C were each opened by Front
itself after the previous report landed. That is the one thing this episode
set out to prove and it held.

## Defects the trial found

1. **Coverage narrowed by a stale guide table** — described above. Owner: the
   `publish` guide.
2. **A run ends itself while its work is in flight — four times in three
   runs.** Every run's first serving wrote an `ag-routinerun` block seconds
   after opening a delegation, each contradicting itself: *"so the run
   continues"*, *"Not applicable yet — the run has not ended"*, *"Not final"*.
   The block is what ends a run. The guide already forbade this in one line,
   so one line is demonstrably not enough.
3. **A callback into a finished run forks a twin.** Ending a run resolves its
   topic and resolving *renames* it, so the callback route read the bare name
   the root note recorded, found nothing, and posted — a second topic beside
   the real one, carrying no origin note, whose report could never reach the
   requester. Three times.
4. **A finished run keeps a delegate talking** — stage A's twin sustained
   three further paid rounds with autolab over completed work.
5. **Continuation recovery ran only at process start**, so a queue expiry —
   which happened mid-trial at 12:30:57 — would have left an owed handoff
   waiting for a restart.

## Fixes made after the trial

- `pyagag` `a7bc4a5` — `on_sweep`, so a listener can recover after every full
  sweep and not only at process start (defect 5).
- `agfront` `acea1fc` — a resolved home is never served, so no twin is forked;
  a late answer is instead named to the run's origin with a delivered note, so
  the conversation that can decide about it is brought back; and the
  `routine_run` guide now separates "a serving ending" from "the run ending",
  states that the block is irreversible, and gives the test to apply before
  writing one (defects 2, 3, 4). Five new regression tests, verified to bite.
- `agfront` `21b1f7c` — pyagag pin.
- `routine-publish` › `guide` **v2**, message **6254** — the project table is
  no longer the definition: discover the set from the workspace and each
  `README_PROJECT.md`, deduplicate aliases, cover every project found and
  report a candidate count for each; all four projects listed; a `publish/`
  clone with no commit is a first publication, not a fault (defect 1).

Defect 2 was answered with guidance rather than a code guard, deliberately.
The evidence said the wording was wrong, not that the block needed a lock, and
the policy here is to change guidance on evidence and re-measure. **The
follow-up run measured it: 17 entries, exactly one finish block, at the end.**
Four self-contradicting blocks in three runs became none in one. No guard is
warranted on this evidence.

Front was restarted onto the fixed code at 12:44:33Z, idle and with no
in-flight work. Its contract did not change, so its introduction was not
re-posted.

## Interventions — this attempt was assisted

Five, all state repairs or housekeeping, **none of them a stage start**: four
writings of a run topic's `[selfnote][rootchat]` origin anchor (messages 6131,
6170, 6213, 6233) to stop defect 3 stranding a report, and one resolve of the
looping finished workplan topic at 12:25:42.

Front cannot make the anchor repair itself — it is told never to post into a
run it opened, and `agentchat` writes a root note only into *another* agent's
topic — so only an outside actor could undo that damage.

**Deus Ex Machina note:** *did the run-topic origin-anchor repair for agent
Front — handoff candidate.*

One of those repairs had a side effect worth recording: the pre-placed anchor
made a bare topic look like a run the conversation had opened, and
`start_opened_runs` started it as one, which combined with Front's `front`
role posting a finish block directly into a run topic to produce one extra
round at 12:35–12:37. It converged on its own.

## Conclusions, kept apart

**Work completed.** Two of four study projects published to the local
`publish/` boundary, one new arXiv investigation written and pushed, and that
result published — all from one request, in order, with artifacts verified
independently of the agents' claims.

**Autonomous coordination proven.** A composite request now stays alive across
child completions: the handoff fired six times and Front opened stages B and C
by itself, choosing the next stage from the conversation rather than from
anything encoded in the listener. Ordering held under four premature finishes
— Front never mistook an ended run for a finished stage.

**Proven on the follow-up** ([report4.md](report4.md)). One request, four
projects, **no interventions at all**: coverage now discovered from the
workspace (five entities found where the table named two, `papers`
deduplicated, an archived empty project surfaced and accounted for), and the
premature finish gone. `ghtrends` and `mediagen` are published and verified.

**Not proven, and still open.**
- **The trial itself was assisted** — five interventions — and that does not
  change retrospectively. What the follow-up proves is that the *repaired*
  system runs this shape of request unassisted; it does not re-prove the
  three-stage composite chain, which has only ever run assisted.
- `relay_late_answer` (defect 3's fix) was **not exercised in the realm**: no
  run ended early, so no callback reached a resolved run. It is covered by
  tests only.
- The post-sweep recovery hook (defect 5) has not yet had an owed handoff to
  recover.
- **Four `publish/` commits await the developer's manual push**: `8901b34`
  (study-realworld), `cac3527` (study-arxiv), `44ab05a` (ghtrends), `7220044`
  (mediagen). The last two sit on origins that hold no branch at all, so their
  clones read as *gone* rather than *ahead*.
- Two `workplan-` topics from the trial are left unresolved. Front noticed and
  reported them rather than tidying on its own, which is its contract.
