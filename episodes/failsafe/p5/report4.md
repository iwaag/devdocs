# failsafe p5 — step 4: routine continuation and a resumable close-out

Date: 2026-09-27 (UTC). Deployed 12:48Z, with a fix at 12:50Z (below).

## What changed

**`agrun`** (agfront) replaces `agrunfinish`. It is granted to `front`,
`desk` and — new — `routine_run`, and its script is in Front's venv:

| command | what it does |
|---|---|
| `agrun status [<channel>/<topic>]` | the runs a conversation opened and where each one's end stands: open, or ended / report delivered / delivered note / resolved. The same text is written into every desk and front serving as `tools/runs.md` |
| `agrun continue <run> --because <post> [--note …]` | a visible line in the run plus `[selfnote][start] #<post> for <Front>`. That is the listener's own durable start mechanism, the one autolab starts tasks with: the note is enqueued and served, and a start nobody answered survives a restart. It is refused for an ended run, and writes nothing while a continuation is already owed |
| `agrun adopt <work topic> --run <run>` | moves Front's own root note in work opened beside the run (`[selfnote][rootchat-moved] <run> #<run anchor>`), so the work's answers serve the run and every reader's tree puts it under the run. Only work of the same request: the run must have been opened from the conversation the work answers to. Nothing is inferred from names |
| `agrun finish <run> --achieved/--not-achieved …` | the end record (unless one exists), then the close-out below |

**The close-out is one function over the records**
(`agfront.routine.close_out`). Starting from the run's end record, it
delivers the report into the origin if no report naming the run is there,
writes the `[delivered]` note if missing, and resolves the run if not ✔.
It is used in three places:

- after every run serving (`close_out_run`, before `continue_deliveries`);
- by `agrun finish` (the external finish);
- at startup, for ends of the last 24 h (`close_out_pending`).

**The order is now: end record first.** The listener used to deliver the
report during the run's serving and post the end record afterwards as the
reply. A crash in between left a delivered report with no end, and the
restart re-ran the run. Now the serving's reply is the end record, and
the delivery follows from it. An end record no longer implies a delivery;
a missing delivery is completed; nothing is written twice.

**Resuming a self-owned run.** Observer's `unheld` next action now says,
when the stalled conversation's owner is also its asker (Front's run),
that its own post serves nothing and names `agrun continue`
(`agag.trace`, pyagag `c668784`). Before, it said "a post there starts a
new serving", which is exactly what #13715/#13790 did, to no effect.
Recovery still counts only a new serving that did work (unchanged
failsafe rule), so a self-post is never reported as resumption.

**Guides**. front/desk:

- `tools/runs.md` and `agrun status`;
- continue when an answer the run needs lands here or Observer says the
  run stopped;
- end it from here when its work is complete by record;
- "opening the run is the whole of your work"; work opened here anyway is
  `agrun adopt`ed.

`routine_run`: adopt work that was opened from the requesting
conversation. Whether the goal was achieved stays Front's judgment: the
tools only make each of these acts effective and repeatable.

## Interruption results

`agfront/tests/test_agrun.py`, a fake realm, interrupting `agrun finish`
at each write:

| interrupted at | first attempt left | retry | result |
|---|---|---|---|
| the report's send | end record | report + note + ✔ | one report, one note, one end record, ✔ |
| the delivered note | end record + report | note + ✔ | the same |
| the resolve | end record + report + note | ✔ | the same |
| (listener path) restart after the end record | end record | startup recovery delivers | complete; a second recovery finds nothing |

Also:

- a finish repeated after completion writes nothing;
- `continue` writes one start and a repeat nothing, and refuses an ended
  run;
- `adopt` moves once and refuses another request's work;
- the self-finish path writes the record as its reply, and the close-out
  runs after it (`test_routine_run.py`).

agfront 193, pyagag 1042, agautolab 331, agobserver 175, relay 363.

## A duplicate delivery at deployment (developer-caused)

The first startup with the new recovery (12:48:46Z) delivered the report
of B growbox's run (`routinerun-20260927T103533Z`) a second time into
`front-desk-20260927-pp1b-growbox` (#13868 plus a delivered note #13869).
That run had delivered its report *before* its end record (#13829, then
#13831), the pre-p5 order, and the first version of the check looked for
a report only after the end record. The delivered note then bought one
Front desk serving (#13871, "Nothing new here since my last report …
reported three times over"): one paid run and one duplicate delivery,
both caused by the developer's change. Fixed at 12:50Z (agfront): a
report or delivered note naming the run counts wherever it is in the
origin, since a run topic's name is unique. A regression test covers it.
A read-only check of the last three days' runs found every close-out
complete, and the restarted listener found nothing to do.

## Not done here

- The concurrent trials in step 6 are where continuation and the desk-side
  finish happen live.
- The 24 h horizon means an end recorded earlier than that and never
  closed out is left alone. That is deliberate, because older reports used
  other wording; it is a limitation.

## Commits

pyagag `c668784` (unheld wording); agfront: `agrun`, close-out, guides,
grants, the recognition fix; pins in agautolab, agobserver and the relay;
pj-agdev pointers.
