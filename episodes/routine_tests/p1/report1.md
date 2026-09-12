# Step 1 — baseline and the one prerequisite that was missing

Read-only baseline, then one repair the inspection demonstrated. Everything
below is evidence gathered before the trial; nothing was asked of an agent.

## Service and placement state

`nctl status` and `nctl drift` both answer clean: **43 targets converged, 0
error and 0 warning diffs**, the 15 `info` rows being liveness and realization
summaries. Six agents are declared; five report `liveness=polling`
(autolab on agstudio, cagent, agforge, agfront, arxivsage) and one —
autolab's second placement, the VM — reports `liveness=stale`. That placement
owns no Zulip channel and answers nothing, so it is not on this trial's path.
The Celery worker is up with no pending jobs; every submodule is clean.

Front's listener is deployed from its own checkout and was idle at the
baseline: `full sweep: 0 awaiting, 0 mentioning`.

## The board

Five project channels (`pj-ghtrends`, `pj-mediagen`, `pj-papers`,
`pj-studyarxiv`, `pj-studyrealworld`), one open work channel (`work-m6013`,
its single `workrun-` topic resolved). No routine run is in flight: both
`routine-publish` and `routine-papers` hold nothing but `guide`, and the two
runs that exist elsewhere (`routine-ghtrends`, `routine-study-realworld`) are
resolved. Every `workplan-` topic is resolved except
`pj-studyrealworld › workplan-create`, which is old and whose last speaker is
the agent, so nothing sweeps it.

One pre-existing stray twin: `#front` holds both `✔ front-routine-mediagen`
and an unresolved `front-routine-mediagen` containing only Front's own ack and
"There is nothing in this topic to answer yet." Front is its last speaker, so
it is inert. Left as found — it belongs to an earlier episode.

## The project inventory, and where the guide's table is stale

The `publish` guide (message **5496**) opens with the general rule — *"Serve
any study project that has a `publish/` folder"* — and then carries a table
that says *"Today there are two, and unless the request names one, cover
both"*, naming only `studyarxiv` and `studyrealworld`.

Read from the project records and each `README_PROJECT.md`, the workspace
holds **four** study projects with a `publish/` folder, not two:

| project | pattern | `publish/` | backlog at baseline |
|---|---|---|---|
| `ghtrends` | study | yes | **8 of 10** repo summaries unpublished, plus the index |
| `mediagen` | study | yes | **everything** — the `publish/` clone has no commit at all |
| `studyarxiv` | study | yes | none — `publish/` matches `main/` |
| `studyrealworld` | study | yes | **3 of 6** sources and about ten report directories unpublished |

`papers` also declares the study pattern, but it is an **alias, not a fifth
project**: its `main/` and `studyarxiv`'s `main/` are clones of the same
repository, `papers` is the older of the two, and it has no `publish/` folder.
Deduplicated away, as the plan requires. `refactorp1`/`p2`/`p3` and
`runsmoke1` are not study projects.

`studyarxiv` being current was checked file by file: every published file is
byte-identical to its `main/` counterpart except `README.md`, which is the
guide's one documented exception, and the only `main/` files absent from
`publish/` are `.gitignore`, `papers/SELECTION-2026-08-28.md` (an internal
selection note) and `papers/INDEX.md` — whose content the published
`README.md` carries as the reader-facing paper table. So `studyarxiv` is an
empty-backlog no-op, not an omission.

Every working tree in all four projects is clean; no pre-existing local edit
had to be preserved.

**The stale table is deliberately left alone.** It is a selection question,
and the plan is explicit that a possible selection mistake is something to
observe, not to pre-empt. Whether the system reads "all study projects" as
the guide's opening rule (four projects) or as its table (two) is exactly what
this trial is for. The numbers above are the yardstick the result is measured
against in step 3.

## The prerequisite that inspection did demonstrate

The concern named in the plan is real, and it is not a matter of judgement.

`finish_run()` in `agfront/src/agfront/zulip_listener.py` delivers a finished
run's report into the requesting conversation with `send_to_channel` on
**Front's own credential**. That leaves Front as the last speaker there. Both
paths that could serve a conversation again refuse exactly that case:
`sweep_topics` skips a topic when `last_real_sender(history) in (None,
self_id)`, and the event path in `sweep_serve` re-reads the topic and applies
the same test before dispatching, so the delivery's own message event does not
rescue it either.

Nothing else provides the continuation. `start_opened_runs()` starts runs that
have been *opened* and never served — at the moment a report lands, the next
stage's run does not exist yet, so it has nothing to find.
`recover_unstarted_runs()` is the same question at startup. `sweep_rootchats`
looks for topics Front anchored on somebody else's behalf, and the requester's
conversation is deliberately never anchored — `finish_run` posts directly
rather than through `agentchat` precisely so no root note points from the
requester at the run. The existing test
`test_finishing_delivers_the_report_to_the_origin_and_resolves_the_run` pins
the delivery and asserts nothing happens afterwards.

So: a request for **one** routine completes. A request that asks for anything
**after** the routine stops at the first report, with nobody left to notice —
which is the whole of stage A → B in this trial.

## The repair

One generic completion handoff, committed as `d596f7f` in `agfront`.

The delivery is now followed by a second post into the requester's
conversation: `[selfnote][delivered] <run channel>/<run topic>`. A selfnote is
never counted as somebody speaking, so **who the sweeps serve is unchanged** —
no new topic becomes eligible for anybody. What the note buys is that "a
report landed here and nothing has served this conversation since" becomes a
question the chat itself can answer (`awaiting_continuation`), and
`continue_deliveries` asks it after every run serving and once at startup.

Three properties were deliberate:

- **It is generic.** The handoff says only that somebody should look. What to
  do next — open the next run, tell the developer the request is finished,
  stop because the report says the work failed — is decided by Front in the
  conversation, out of the request and the report that are both readable
  there. No stage of this episode, and no notion of a composite request, is
  in the listener.
- **It is asked on both routes.** A run almost always ends on a *callback*
  serving, not on its first, so `handle_mention` asks too. Asking only from
  `handle_topic` would have fired for the rare run and not the normal one.
- **The note is written after the report.** A crash between the two leaves the
  requester with the report and no handoff, never a handoff and no report.

Bounds, each with a test: an ordinary one-routine request does not loop
(Front's reply is speech and speech spends the note, so a second pass finds
nothing); a serving **ack** does not spend it, so a crash between a serving's
ack and its reply leaves the handoff owed and startup recovery finds it; a
resolved requester is not reopened; two reports into one conversation serve it
once; a run opened by hand writes no note; and the chain of stages inside one
serving is bounded by `CONTINUATION_DEPTH`.

Startup recovery gained the matching half: `recover_runs` already started runs
Front opened before it went down, and now also serves requesters a run
reported into while nobody was listening. Both are the same gap — Front is the
last speaker, so no sweep can reach either.

### Checks

`agfront`'s whole suite: **119 passed**, up from 109, with ten new tests in
`tests/test_routine_run.py`. The new coverage was confirmed to bite: with the
continuation short-circuited, five of the new tests fail and the rest of the
suite still passes, so they are testing the handoff and not the fixture.

### Deployment

Front's listener runs from this checkout under its own launchd job, so the
commit is the deployment. Its in-flight work was checked first — `0 awaiting,
0 mentioning` — and it was restarted afterwards, coming back clean at
12:05:28Z on the commit. Its contract did not change (same entrance, same
prefixes, same published options), so its introduction was **not** re-posted.
Startup recovery found nothing to continue, which is correct: the note did not
exist until this commit.

## What was deliberately not done

- The `publish` guide's two-project table was left stale, for the reason
  above.
- The role guides were left unchanged. The `routine_run` guide forbids a run
  opening another run, which is what the plan prefers — sequential siblings
  coordinated from the original conversation — and the `front` guide already
  covers opening a run and being told when one ends, and says nothing that
  prevents opening the next one. Neither demonstrably blocks a composite
  request, so neither was touched.
- No autolab or shared `pyagag` change: the defect was located in Front's
  listener and repaired there.
