# Step 3 — the `study-industry` routine guide

Date: 2026-09-15 JST (timestamps UTC). Plan: [plan.md](plan.md) section 3.
Previous: [report2.md](report2.md).

## The channel

`#routine-study-industry` — stream **164**, created from the Developer
credential in the existing `routine` channel folder (id **13**, looked up
by name rather than assumed), with the same description shape the other
routine channels carry and the same three subscribers: Developer (8),
Front (15), Opsroom Observer (22). The last is the observation path — the
credential the agentroom relay's mirror reads with — so the board sees the
channel without any further step.

## Front's listener

A fresh subscription does not reach an already-registered event queue, so
the listener was restarted. Checked first: the routine board showed
`open_runs: 0` for every routine, Front's log had been silent since
2026-09-14 05:59Z, and the listener process had no harness grandchild.
`launchctl kickstart -k` at **10:11:31Z**; the restart came up clean and
re-served nothing:

```
10:11:31Z front zulip listener starting (mirror in …; all topics in 'front-agstudio1', prefixes ('front-', 'routinerun-') elsewhere, …)
10:11:31Z listening as user_id=15
10:11:31Z registered event queue 1a94b896-…
10:11:33Z recovery (startup): 0 conversation(s) queued from the index, 0 pending
```

Read back from the Front bot's own subscription list afterwards:
`pj-studyindustry` and `routine-study-industry` are both there, beside the
six earlier routine channels. Subscription is the routing decision; this
is what makes a `routinerun-` topic in the new channel servable.

## The guide

One full post in `guide` — message **7015**, by the Developer, 7 797 bytes
sent, 7 774 characters stored and read back with no truncation marker.
The full text is kept in the ignored `.local/` beside this report.

Its substantive half is the plan's intent, as two block-quoted paragraphs:
the investigation goal (leading companies and market positions, emerging
companies attracting attention and investment, where participants invest,
other useful information; read existing methods and reports first; methods
in `methods/`, findings in `reports/`; scope, sources, methods, tools and
presentation are the researcher's; retain or improve methods on the
evidence) and the one added capability — local statistical or database
tools or services may be used, existing or started in Docker, after
notifying the requester, with cagent asked about existing services first,
and with reusable explanations and scripts kept in `methods/` sufficient to
rebuild the analysis without the local database.

The guide says in its own words that the second paragraph is an option and
that a run which judges no service useful, and says why, has answered it.

Around that, the operational context and nothing else:

- the project and channel, with `README_PROJECT.md` and `main/README.md`
  outranking the guide on layout;
- how the industry is chosen when the request names none (continue the
  newest report's industry, and say so);
- read before choosing, and say what was found and reused;
- a bounded scope with geography, segments, time coverage and cutoff
  stated, gaps explicit;
- a report outranks a plan for one;
- the human-dependency rule carried from `study-uspolitics` v2, plus
  "never conceal a material loss of coverage";
- **service coordination** — notify the requester (Front relays to the
  developer conversation) with purpose and resource; ask cagent, found from
  its introduction, which instances exist and whether research use is
  appropriate, and report the answer to Front; prefer a separate study
  database or schema on a suitable existing service; record any new
  service's owner, purpose, location and retention in ignored notes; retire
  study-owned temporaries, never shared data;
- honesty about data, made specific to this subject: market, denominator,
  period, geography and source for any share; estimates labelled;
  announced versus completed deals, funding versus acquisition value versus
  operating spend, attention versus traction; projections as inferences
  with assumptions and indicators;
- checks leave a dated record in the knowledge tree, location the
  researcher's (study-realworld finding 1);
- long jobs wake nobody: leave identity, outputs and next action in the
  conversation and arrange a callback through an agent on the board that
  waits (no agent named — the board says which);
- `main/` publish-ready, `.local/` for downloads, databases, caches, logs;
- commit and push `main`, never touch `publish/`;
- one task with parallel subagents;
- the approve-and-act closing line, and what the report names (now
  including the scope and the optional-tool decision).

**Not in the guide:** the plan's review questions for the trial (they are
the observer's), any source list, calculation sequence, report template,
database requirement, bot name or endpoint. The guide names no agent for
either the service inquiry or the waiting; both are read off the board at
run time.

## Discovery

The relay's `/routines` — the same read the operation dashboard and
routines view make — lists the new routine beside the others:

| routine | channel | guide message | runs |
|---|---|---|---|
| ghtrends | routine-ghtrends | 5498 | 1 |
| papers | routine-papers | 5499 | 1 |
| publish | routine-publish | 6254 | 4 |
| study-realworld | routine-study-realworld | 5497 | 1 |
| study-uspolitics | routine-study-uspolitics | 6393 | 3 |
| **study-industry** | **routine-study-industry** | **7015** | **0** |

(`anchorcheck` is listed as retired.) `retired: false`, `open_runs: 0`,
one guide post by the Developer. The mirror's health was `live` throughout,
and the channel reached the board without a relay restart — it is public
and the mirror's queue already carries every public channel.

## Step 3 conclusions

1. The routine exists as a channel in the `routine` folder with the
   required participants and is discoverable through the ordinary board.
2. Its guide is one full post, message 7015, carrying the plan's intent
   and the single added capability as an option, plus operational context
   only.
3. Front's listener has the subscription, restarted with nothing in flight
   and re-serving nothing.
4. No run has been requested and no paid run was bought in this step.
