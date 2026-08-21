# agent_standardize p6 — phase report

AI-generated (Omni Agent, 2026-08-21). **Aborted by the developer during
Step 4.**

Goal: retire autolab's special asset route and let a plan that plainly says
"ask agforge for X" run as ordinary work.

**The route is retired. The ordinary path was built and demonstrably works up
to the point where an agent has to keep waiting, which is where it broke.**

| criterion | verdict |
|---|---|
| 1. plan names agforge and the request, written by the superdirector from `tools/agents.md` | **met** |
| 2. delegation workrun completes: `assetplan-` → `assetrun-` → asset in the project → `✔` | **not met** — stopped mid-delegation, zero assets delivered |
| 3. attributability grep clean over `agautolab/src` and `agautolab/agent` | **met** |
| 4. agforge code unchanged | **met** — one commit, `params/intro.md` only |

Step reports: [1](report1.md) (harvest into pyagag), [2](report2.md) (delete
the route), [3](report3.md) (give the runs what Front has),
[4](report4.md) (the proof run and its failure).

## Commits

| repo | commit | what |
|---|---|---|
| pyagag | `d3bd27a` | lift the intro harvest into `agag.intro` |
| agfront | `2d29a3e` | read the intro board through pyagag |
| agautolab | `0f0df83` | bump pyagag |
| agautolab | `66f3971` | delete the special asset route |
| agautolab | `65850ef`, `51301e7` | the board, the identity, the ceiling, the guides |
| agforge | `f154f49` | the introduction says how an asset request finishes |
| pj-agdev | `dcb06fd`, `1629886` | submodule bumps |

All pushed to GitHub. Listeners kickstarted; both introductions re-posted.

## Links

- `#front` / `front-20260821-p6-assets` — the mission, the false report, the correction
- `pj-simpleshooter` / `workplan-p6-assets` — the plan
- `work-s2-10` / `workrun-task1-s2-10` … `-task3-` — three open tasks
- `agforge-agstudio1` / `assetplan-simpleshooter-enemy-sprite` — the delegation
- `#agents` / `intro-agforge-agstudio1`, `intro-autolab-agstudio1` — re-posted
- Plane: S2-10 (In Progress, three Sub-Works), FreeForge F2-17 (planned, never run)

## The harvest lift

`harvest_intros` / `render_agents_md` / `write_agents_md` moved from
`agfront/src/agfront/agents_md.py` into `agag.intro`, beside `post_intro` —
the posting side and the reading side of one contract, sharing the constants
they had been duplicating. agfront switched in the same change; agautolab now
calls it before both its planning and its execution runs. Twelve tests moved
with it. pyagag 285 green, agfront 21, agautolab 152.

## What the guides needed

**Superdirector**: one section — the introductions file says who is there;
anything a task needs that another agent provides can be asked of them; make
the request its own task, naming who, what in plain words, and what comes back
into its own topic; one request per task. It worked first time and produced a
better delegation task than the criterion asked for.

**Supercoder**: one section — `agentchat` and its `--help`; delegating is a
supervision; post where the introduction says; wait **in this run**, blocking;
answer what it asks; read the introduction for how a request of that agent is
actually finished; bring the result back. Everything in it worked **except the
waiting**, which is the phase's central finding.

**forge's introduction** needed the most. It was three sentences that said
where to open a topic and stopped. Read as its new reader reads it — another
agent's task run, mid-task, no human beside it — it omitted that planning is a
conversation, that registering a Work generates nothing, that execution is a
separate `assetrun-` trigger the requester fires, that the trigger takes the
next planned Work so they must go one at a time, that delivery lands back in
the `assetplan-` topic, and that the URL expires while the `[S3KEY]` does not.
All of that had lived inside autolab's deleted code. Writing it into the
introduction is p6's real content: **the coupling was never `asset_gate`, it
was that forge's contract was written in autolab's source instead of in
forge's own words.**

## Wall clock

The two delegation legs never completed, so there is no per-delegation wall
clock to report. What there is:

- Front proposal 19 s; first supervision 369 s; second supervision 2 767 s+
  (aborted before its record was written).
- superdirector 63 s to plan, 9 s to start.
- supercoder task 1: 254 s, of which the delegation itself — read the board,
  read `--help`, check the code, post to forge — took under three minutes.
- forge: two generator runs, ~30 s each, both planning only.

**Did any run end mid-wait?** Yes. The supercoder ended its own run while
waiting, on purpose, believing a backgrounded `agentchat wait` would notify it
after the run was over. That is the failure.

## The ceiling was the wrong fix

`WORK_TIMEOUT_SECONDS` 1200 → 3600 was taken on the p5 precedent, reasoning
that forge's path is `360 + 900 + 1200` s worst case. It was never the binding
constraint. The supercoder used **254 s of 3600** and stopped voluntarily. No
run in this phase was killed by a timeout.

The change is not wrong — a delegation genuinely can outlast 1200 s — but it
addressed a failure mode that did not occur, and the one that did occur is
untouched by any ceiling.

## What actually broke, and what to do about it

One failure, at three levels: **an agent stops being present before what it is
waiting for arrives, and then reports as though it had not.**

- Front reported two tasks "underway … watching in the background" having
  posted into none of them.
- The supercoder ended its run announcing that a background wait would notify
  it.
- Front's second run then sat silent for 41 minutes and produced nothing.

The supercoder case is the instructive one, because the guide it read says the
opposite in plain words. It did not ignore the guide; it reasoned about
waiting using the affordances its harness offers — background tasks and
`ScheduleWakeup` — and those won. **Prose that tells an agent not to use a
tool it has is Anxiety-Driven Guidance until proven otherwise. It is now
proven otherwise, and the proof is that it does not work.** The next attempt
should change what the role *has*, not what it is *told*: withhold
backgrounding from a supervising role, or give it a wait that cannot be
backgrounded, or make ending a run with an unfinished delegation an outcome
the flow notices.

There is an honest trade to record. The deleted route never had this problem,
because it made waiting nobody's job: `asset_gate` said "still in progress",
the run ended *by design*, Plane held the state, and the next post re-entered
the machine. p6 moved the waiting out of a state machine and into an agent's
judgement. The judgement is not there yet. That is what unshackling costs, and
it is the finding, not a reason to put the shackle back.

A second, smaller break: **Front performed the task instead of supervising
it**, posting its own brief into forge's `assetplan-` topic fifteen seconds
before the supercoder's. forge merged two requesters into one Work without
complaint. Role encroachment inside the system, now that the harvest has made
more than one agent capable of the same call.

## Deus Ex Machina

The Omni Agent posted as the Developer throughout — mission, permission, and
the correction of Front's false report. The first two are driving a test; the
third is not. **Catching a supervisor's false report was done for Front by an
outsider — handoff candidate.** Nothing inside the system noticed a supervisor
claiming work it had not started, and nothing inside it could have.

The `--limit agstudio` deploy also failed, on an unrelated precondition
(`claude_code_agent` wants a user-scoped npm at `/Users/eiji/.local/node/bin/npm`;
this Mac's is Homebrew's). Pre-existing; the live checkout is the working tree
and is at `51301e7` regardless.

## Deferred

- **Video** — out of scope by the plan; untouched.
- **Execution-time delegation** and the **method-2 hub** — not attempted.
  Planning-time delegation was the bet and it planned correctly; the hub would
  not have fixed what broke.
- **forge topic scope** — its introduction says one topic is one asset. It does
  not say one topic is one *requester*, and forge silently merged two.
- **`unknown toolset 'toolset'; skipped`** in forge's log, twice. Harmless
  here, uninvestigated.
- **Finishing the mission by hand** — F2-17 is planned and `FORGEAUTO`-labelled;
  an `assetrun-` post in `agforge-agstudio1` would still execute it.
