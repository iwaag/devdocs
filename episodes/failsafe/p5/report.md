# failsafe p5 — report: wait correctly and finish delegated studies

## Outcome

| completion condition | result |
|---|---|
| the required-results table | ✓ all six rows (report6). The blocker-stops row is proven by accelerated tests, labelled as such; the rest live |
| both concurrent-study trials on the final implementation pass | ✓ **F** and **H**: two studies each, overlapping for their whole lifetimes. Each finished with its research accepted and integrated, the sage refreshed for that run, the run ended and the report delivered. No human input after the requests; Observer posted nothing. H ran on the exact final code |
| the repeated study returns to its own run | ✓ growbox and worldtrend each ran five times (C–H). Ten refreshes went to ten topics of their runs' own, each recorded `for=` that run and `includes=` its commit, and no answer reached an earlier run |
| one confirmed-stop check on operational timing | ✓ 129 s from the killed task serving to Observer's request to Front (goal 300 s). Front resumed 15 s later; completed |

Step reports: `report1.md` (the gaps reproduced), `report2.md` (queue
evidence), `report3.md` (acceptance), `report4.md` (continuation and
close-out), `report5.md` (refresh binding), `report6.md` (validation).

## Adopted acceptance semantics

- **The decision holder decides, whoever records it.** The holders are
  the mission's requester and everyone up its root notes to the request's
  origin. For a study run by Front, that is Front (entrusted by the
  routine's guide) and the person who asked.
- **A person can reserve it** (`agentchat reserve --evidence <their
  post>`); then only they hold it, and the mission waits for their words.
- **The evidence**:
  - must be a holder's post, never the mission's own agent's;
  - must come after the last result shown for review (`+shown=`), so the
    initial request accepts nothing;
  - if an agent's words, must be in the work's own conversations.
- **The record** is `[selfnote][acceptance] #<post> by <user> (<name>)
  after=#<shown>`. Front recording its own agreement and autolab
  recording the same post produce the same record; the second writes
  nothing.
- **Four records kept apart**: task acceptance (the review serving's close,
  bound to the checkpoint), mission acceptance, the routine run's end, and
  report delivery.

## Queue and recovery timing

- **One reading of a queued post** (`agag.waits`) for the panel and
  Observer, from autolab's own listener journal where available
  (`probe_queue`), else from the open servings of the traced requests, 30
  min at most. A post that names nobody belongs to the agent whose roster
  serves its topic.
- **Observer's actions by state**:

  | state | Observer |
  |---|---|
  | `behind` | defers, and checks again every look |
  | `blocked` | leaves it to the blocker's own request, for at most 10 min |
  | `unserved` (idle 90 s, passed over by 3 servings, or given up) | reports to the owners at once |
  | `unknown` | the plain rule |

  An incident on a wait later confirmed legitimate closes `excused`.
- **Live**: confirmed healthy queues of 344 s (H) and 495 s (D) were
  deferred with no request. A killed task reached Front in 129 s. Four
  false interventions surfaced during the trials, and each was fixed before
  the next trial (report6).

## Close-out interruption results

The run's end record comes first. The report, the `[delivered]` note and
the ✔ follow from it and are completed where missing: after a serving, by
`agrun finish`, and at startup. Interruptions at the report, the note and
the resolve all converged on one report, one note and one end record in
tests. Live, every run of C–H closed out once. The first startup
re-delivered one pre-p5 report, because the old order had the report
*before* the end; this was fixed within minutes (report4).

`agrun continue` gives Front a real serving of its own run through the
listener's start note (used live in D). `agrun adopt` puts work opened
beside a run under it. Observer's `unheld` for Front's own run names
`agrun continue`, not a post.

## Callback and revision evidence

- `agentchat send` refuses to post where answers would return to another
  request.
- The study guides (v3) ask for a refresh topic of the run's own, addressed
  to archsage, naming the commit.
- `archsage sage sync --require <commit>` records `for=` and
  `includes=`/`missing=`.
- The panel's `knowledge_refreshed` counts a refresh by that relation and
  revision, never by project and time. Overlapping and reordered refreshes
  are tested.

## Interventions and cost

- **Stand-in input**: only the requests, except trial G's planned reserved
  approval and one explanation Front asked for.
- **Developer repairs during the step**: listed in report6. Trial E growbox
  is an assisted run.
- **Duplicates**: one duplicate report delivery, caused by the developer at
  step 4's deployment. No duplicate execution or acceptance.
- **Cost**: $49.20 for the step-6 trials (C $8.95, D $8.43, E $9.83,
  F $8.38, G $4.77, H $8.84), plus one paid Front serving for the
  duplicate; steps 1–5 ran no model.

## Limitations

- Execution health is still autolab's alone. Front's and archsage's queues
  are read from conversations, and that excuse is bounded at 30 min.
- A live blocker that stops while a post queues behind it was not
  provoked; that path is test-only.
- The completion door records the pressing person's decision without the
  holder check (report3).
- Front still spends retries on `agentchat send` arguments (`--to`
  missing or guessed). The refusals say what is wrong, but it keeps
  happening.
- Front's run waits inside its serving while a task runs (D: 5½ min),
  which lengthens the queue for other requests' answers. Serial listeners
  are accepted in this phase.
- The close-out recovery horizon is 24 h.
- comfynotify keeps its old pin. agautolab1 (VM) was not redeployed.

## Left for the Developer

- The ✔ on the review topics (statuses recorded, report6).
- The older holds are unchanged and separate: o11711/m11741 (present and
  recover its existing results first) and o8512/m8519 (can be accepted
  after reviewing its shown task 2 result). Neither was accepted, resumed
  or cancelled by this phase.

## Where it lives

- pyagag:
  - `7725a52` (waits, queue probe, recheck);
  - `f62acec`, `1677a19` (acceptance);
  - `c668784` (unheld for self-owned runs);
  - `a44850a` (refresh relation, cross-request refusal);
  - `ab8c98a`, `1c7937d`, `e9e6229` (trial fixes);
  - `fb61ea3` (display string).
- agfront: `agrun`, the close-out, guides, grants. agautolab: close-out
  wording and introduction. agobserver: `review_queues`, `unserved`,
  `excused`, `QUEUED_KINDS`. relay: queued probes. archsage: `sagesync`
  relation and `--require`, and its guide. All pins are committed and
  pushed.
- Study routine guides v3 (#14400–#14402). README_DEV *Waiting and
  finishing studies*. Host notes in `pj-agdev/.local/devenv.md`.
