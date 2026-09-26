# sage p2 — step 5 report: deploy, three entry cases, evidence

## Deployment and service state

- **nctl, before the live cases.** `nctl drift` showed 45 converged and
  `snake-web: service_missing`. After `nctl reconcile agstudio
  --refresh-observation --max-rounds 1 --yes` and `nctl agents observe` it
  showed converged=46, with every listener `polling` (agautolab1 VM `stale`,
  as usual).
  - Note: with `--yes` that refresh also **re-applied `snake-web`**
    (static_web_app), which restarted that service. This is a local
    scratch service and it is now converged, but it was an actuation, not
    only an observation.
  - No desired-state change was needed. No new logical sage needs an
    account or a service.
- **Versions.** pyagag `a73c216` runs in agfront (`7bf5956`), agautolab
  (`15853c2`) and archsage (`b57f71a` … `a623066`, plus the intro fix).
  Other consumers keep their older pins, because nothing they use changed.
  Listeners were restarted with nothing in flight.
- **Tests:**
  - pyagag: 919 passed, plus the new project and routine tests.
  - agautolab: 304.
  - archsage: 37.
  - agfront: 180, including the new nested-journal test.

  Together they cover partial setup and retry, pattern initialization,
  callbacks across a restart, source replacement, guide read-back, and a
  registered routine starting nothing.

## Case A: a new study with an initial research run (`aisvgs`)

The entrance was `#front › front-desk-20260926-sagep2-aisvgs`, from #11522.
The Omni Agent relayed the Developer's #11484 and the seed
`general@b3a9870ac84bdfabee70225c2cf4d15e64edf075:study_aisvgs/`.

| what | evidence |
|---|---|
| Front routes to archsage | #11525 in `#archsage-agstudio1 › study-aisvgs` (Front's root note → desk) |
| research plan | `pj-aisvgs › researchplan-aisvgs` #11531 (seven strands; the seed is to be verified, not copied) |
| setup request with `ag-setup` | #11533; archsage's progress reply #11536 **named nobody** |
| workspace | autolab #11535: `study layout established: main/ = …/autodev/aisvgs.git at 81f5276b58bc`; `README_PROJECT.md` + `main/` only |
| callback → setup complete | archsage #11550: channel, plan, repository and revision, routine, sage; "set up and not researched" |
| routine | `#routine-study-aisvgs` (stream 222), guide v1 #11543 (6,562 chars, read back intact) |
| sage | `sage:aisvgs` (source `main`), tree `81f5276b58bc`, "no findings yet"; store commit `43064ca` |
| research run | Front opened `routinerun-20260926-2100` (#11559) itself; mission m11579 (`workplan-aisvgs-round1` #11568, plan #11580) |
| research result | autolab #11682/#11684: strands 1–4, `reports/INDEX.md` with 4 rows, `main` `13e0d6e2d08a` (fast-forward; commits `5c8ddc0`, `13e0d6e`) |
| acceptance | the Developer's words via the Omni Agent, #11699 → Front ran `agentchat accept 11579 --evidence 11699`: `[selfnote][acceptance] #11699`, `[state] done` (#11704–#11705) |
| knowledge refreshed | Front asked archsage (#11688); archsage #11694: tree `81f5276b58bc → 13e0d6e2d08a`, 6 knowledge files, queue empty |
| sage answers from the findings | `sage:aisvgs` #11709 (question #11700): the open-weight text-to-SVG models, their licences and hardware, citing `reports/strand3-tools.md` and `reports/INDEX.md` at `13e0d6e2d08a`; it separates measured figures from the study's estimates |

The Gitea head `13e0d6e2d08a695aea79a293008c5d74e48c0704` equals the
sage's tree. The workspace holds `README_PROJECT.md`, `main/` and
autolab's local task record `devlog/`, which is a plain folder. There is
no `aisvgs-devlog` or `-direction` repository.

## Case B: an existing study missing its connections (`worldtrend`)

The entrance was `#front › front-desk-20260926-sagep2-worldtrend` (#11538),
asking for setup only.

- **Failure.** Front's first request (#11542) began with "sage:worldtrend
  has no study attached…". In archsage's own channel, a leading
  `sage:<name>` addresses that sage, so `sage:worldtrend` answered (#11556).
  It said, correctly, that it could not do setup.
- **Recovered without help.** Front re-sent the request in
  `study-worldtrend-setup` (#11575) and said "this post is addressed to
  you, the council". It told the requester it had misrouted (#11576).
- **Result.** archsage #11605:
  - It reused `#pj-worldtrend` (`ready`, plan #7233, repository
    `50e841c913a4`, no findings).
  - It ran `sage attach worldtrend --project worldtrend`: empty →
    `50e841c913a4`, store `05b68a8`.
  - It created `#routine-study-worldtrend` with guide #11604 (5,386 chars,
    intact). The sage's queued round-1 rules went into the guide as a
    checklist, and the note stays queued.
  - Front relayed it to the requester (#11608).
- **Fix (evidence-driven, one sentence).** archsage's introduction now says
  that a request *about* a sage does not begin with its selector.

## Case C: an existing sage missing its study (`growbox`)

The entrance was `#front › front-desk-20260926-sagep2-growbox` (#11539),
asking for setup only.

- Front → archsage `study-growbox` (#11549).
- archsage wrote plan #11593, with four strands and the sage's queued note
  as the first questions, and posted setup #11595.
- autolab was busy with case A's task, so the setup waited about 20
  minutes. Observer opened `incident-unacknowledged-11594`, asked twice
  (#11624, #11638) and recorded "Rescued" (#11651) when autolab took it up.
- autolab #11653: `main` `dd59ce11eed9`.
- archsage #11659 reported:
  - routine `#routine-study-growbox`, guide #11596 (6,247 chars, intact);
  - `sage:growbox` attached, `empty → dd59ce11eed9`, no findings;
  - the queued note kept;
  - "no research done and none scheduled".
- Front relayed it (#11662).

## Board and discovery

- `/routines` lists `study-aisvgs` ("AI-made SVG images"),
  `study-growbox` ("Unattended desktop grow box") and `study-worldtrend`.
- `agroutine show`: the two setup-only routines have **0 runs**, and
  aisvgs has the one run Front opened. Members, folder and description are
  correct on all three.
- `agentchat intro archsage-agstudio1` lists `sage:aisvgs` with its study
  and revision, and `worldtrend`/`growbox` with their studies. Every
  definition change re-posted the introduction.

## Failures found and fixed during the step

- **A receipt written into the wrong conversation** (agfront `7bf5956`).
  The callback serving that read archsage's #11550 also started the run it
  had opened, in the same pass and sharing the entry's journal. The
  receipt for #11550 was therefore written into `routinerun-20260926-2100`
  (log: `marked archsage-agstudio1/study-aisvgs served up to 11550 in
  routine-study-aisvgs/routinerun-20260926-2100`), not the desk.
  - Observer correctly saw an unreceipted answer. It asked Front twice
    (Front answered both, #11618 and #11641) and then **DM-escalated to the
    Developer** (#11666). That escalation was a false alarm caused by this
    bug.
  - The incident could not be withdrawn afterwards ("already reported").
  - Fix: a run started from inside a serving, and a requester served after
    a run's report, each get a journal of their own. The pre-existing
    defect predates this phase: any callback that opens a routine run had
    it.
- **Leading selector misroute** (case B): fixed in the introduction, as
  above.

## Intervention and remaining gaps (handoff candidates)

- **did X for Front/autolab: nudged a stalled research task** (#11669).
  autolab's task run ended at 12:20 with "strand 2 is still running"
  (#11648). A background subagent had not delivered, and a run cannot
  outlive itself. The trace showed a deadlock: the task was `awaiting
  requester` because the post named Front, while Front's run recorded it
  as progress and waited on autolab. One post at the desk (the Developer's
  ordinary entrance) made Front ask autolab to continue (#11672). autolab
  then wrote strand 2 in-session and finished. Handoff candidate: the
  supercoder should never end a serving while a subagent it launched is
  unfinished, or a "still running" progress post needs a waiter.
- **The routine run was left open.** Front's nudge was sent from the desk
  conversation, so its root note re-anchored the task topic to the desk.
  The report, acceptance and refresh all went through the desk, and
  `routinerun-20260926-2100` never received its report. Front correctly
  refuses to post into a run from outside it (#11707), so the run stays
  open. Handoff candidate: a delegation made on behalf of a run should be
  made from the run. Otherwise the requester conversation needs a way to
  hand a finished result back to a run it owns.
- **The acceptance needed a person's post.** Front's first attempt used
  the original request as evidence and was refused, as designed, because
  it is older than the mission. The Developer-side acceptance #11699 is the
  intended path, not a workaround.
- **Observer asks buy Front runs.** The Observer's four posts in the two
  desks each bought a Front serving. In case C, Front's final relay named
  the Observer (the last speaker) instead of the requester.
- **archsage's mention is doubled** (listener plus model) in #11550, #11694
  and others, despite the guide. This is cosmetic.
- **`agentchat trace` shows the Front-only run topic as `NOT_STARTED`**
  although Front served it three times. The trace misreads a run whose
  posts are all Front's.

## Cost of the step's runs (since 11:35Z, including the step 2 probes)

| role | runs | cost |
|---|---|---|
| archsage (frontier) | 10 | $6.47 |
| sage | 2 | $0.18 |
| Front desk / present / routine_run | 15 / 12 / 3 | $1.47 / $0.58 / $0.32 |
| autolab superdirector / supercoder | 5 / 2 | $0.51 / $8.96 |
| **total** | | **$18.50** |
