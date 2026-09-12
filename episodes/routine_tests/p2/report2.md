# routine_tests p2 step 2 — the `study-uspolitics` routine guide

Date: 2026-09-13 JST. Plan: [plan.md](plan.md) step 2.
Previous: [report1.md](report1.md).

## The channel

`#routine-study-uspolitics` — stream **154**, created in the existing
`routine` channel folder (id **13**, the one the other four routines are
filed in), with the same description shape they carry. Subscribed through
the ordinary provisioning route: Developer (8), Front (15), Opsroom
Observer (22) — the same three as `#routine-publish` and
`#routine-study-realworld`.

## Front's listener

The environment notes record that a fresh subscription does not reach an
already-registered event queue, so the listener has to be restarted or the
run sits unserved.

**In-flight work was checked first**, on the routine board rather than by
guessing: all four existing routines showed `open_runs: 0`, and Front's
listener log had been silent since 13:14:54Z — the last `publish` run's
delivery — for over two hours. Nothing was in flight, so the restart was
safe.

`launchctl kickstart -k` at **15:32:14Z**. The restart came up clean:

```
front zulip listener starting (pull sweep: all topics in 'front-agstudio1',
  prefixes ('front-', 'routinerun-') elsewhere, routes ['front-', 'routinerun-'] + DM thread)
listening as user_id=15
registered event queue fed3c09e-…
registered event queue 63f2c7ad-…
full sweep: 0 awaiting, 0 mentioning, 71 calls spent, 930 left in the window
```

`0 awaiting, 0 mentioning` — the restart re-served nothing, which is the
check that matters after p1's experience with recovery routes.

Front's subscriptions now include both of this trial's new channels,
`pj-studyuspolitics` and `routine-study-uspolitics`. Subscription is the
routing decision, so this is what makes a `routinerun-` topic here servable
at all.

## The guide

One full post in `guide` — message **6357**, by the Developer, 4213
characters as stored (the post was checked back from the server, because
this realm truncates over-long posts silently; nothing was lost).

Its substantive half is the braindump's own instruction, reproduced as the
plan specified, as a block quote: investigate publicly available
political-contribution information concerning members of the United States
Congress, use it as a starting point for analysing political developments,
reliable collection and reliable analysis as the minimum, everything beyond
that free — including improving the methods themselves — and accumulate the
knowledge in the project's `main` repository with methods, analysis scripts
and source-use notes in `methods/` and completed reports in `reports/`,
organized as the researcher chooses, building on what earlier runs left.

Around it, and nothing else, the operational context a run needs:

- which project and channel, and that `README_PROJECT.md` and
  `main/README.md` outrank the guide on layout;
- read `methods/` and `reports/` before choosing, and say what was found
  and what was done with it — with the explicit note that retaining a
  working method with evidence beats changing one so that something can be
  called an improvement;
- a proposed collector or setup alone is not the work; a narrow declared
  scope is fine;
- say where each figure came from and when it was retrieved; missing data
  stays missing; keep observation, interpretation and uncertainty apart;
- `main/` is publish-ready — no host facts, paths, ports, internal
  repository names, credentials, or bulk verbatim third-party text;
- commit and push `main`, do not touch `publish/` — publication is the
  separate routine's own mission;
- one task with parallel subagents inside it, because autolab's sweep is
  single-threaded;
- the ordinary approve/act closing line and what a run's report must name.

It also says, in its own words, that the guide deliberately chooses no data
provider, population, period, measure, report shape or research order, and
that a later version should only start choosing where a run has shown that
leaving the choice open went wrong.

**The executor's evaluation questions from plan step 4 were not pasted into
the guide.** They are the observer's, not the researcher's checklist.

## Discovery

Through the ordinary routine interface — the agentroom relay's `/routines`,
which is what the operation dashboard and routines view read:

| routine | channel | guide message | runs |
|---|---|---|---|
| ghtrends | routine-ghtrends | 5498 | 1 |
| papers | routine-papers | 5499 | 1 |
| publish | routine-publish | 6254 | 3 |
| study-realworld | routine-study-realworld | 5497 | 1 |
| **study-uspolitics** | **routine-study-uspolitics** | **6357** | **0** |

Health `live`. The routine appears with the right guide message and no runs
yet, which is exactly the state it should be in before step 3.

This also re-confirms the plan's note that the live `publish` guide is
message **6254** (v2).

## Step 2 conclusions

1. The routine exists as a channel in the `routine` folder with the required
   participants, and is discoverable through the ordinary interface.
2. Its guide is one full post, message 6357, carrying the braindump's
   instruction and only the operational context a run needs.
3. Front's listener has the new subscriptions, restarted with nothing in
   flight and re-serving nothing.
4. No run has been requested yet.
