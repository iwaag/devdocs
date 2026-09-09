# Front Desk p4 — close requests from any work view

agdevworld `agentroom` and frontend only (four commits, see the steps);
pj-agdev pointer moved. Steps 1–3 done; three live completions carried
out from three different views.

## The plan, answered

> Make p3's completion flow available for ordinary Front conversations,
> individual routine runs, and requests made directly to Autolab or Forge.
> Keep the existing Front Desk button and share discovery, preview,
> execution, and result presentation across the views.

The completion is one operation on one root, `(channel, topic)`:
`GET /complete/plan`, `POST /complete`, `GET /complete/history`. The Front
Desk's `finish ✔` is one caller of it, unchanged in place. The operation
room has **finish ✔ this run** on the selected run; the agent room has
**finish ✔** on every topic of an agent's or a board's popup, project
workplans included, with a **✔ resolved: shown** chip so a finished request
can be reached; the ops row popup has a FINISH THIS REQUEST section beside
the board's "confirmed", which still only hides a display row. Every screen
draws the same plan lines (`completionState.planLines`), so the Phaser panel
and the DOM overlay cannot disagree.

Live today: a routine run closed from the operation room, an Autolab
request from the agent room, an ordinary Front conversation from the agent
room — each previewed, applied, and read back in Zulip, the channel list and
Plane; each re-previewed as *0 to change*; the routine's standing request,
schedule and earlier runs untouched.

## What decides the boundary

A conversation's **kind** is its name (`classify`): desk, front, routine
run, workplan, assetplan are requests and may be roots; workrun and
assetrun are execution topics and answer with their parent; a standing
request and an introduction are never targets, because a ✔ there retires
something. Whose a reached topic is comes from its **root notes**: naming
this request, or something it owns, or what the root was opened for → owned;
naming anything else → *anchored to another request*, excluded with the
note's id. An execution topic is decided by its **structural** home only
(autolab's plan for a task, forge's plan for a run); other root notes in it
are Front's visits and are shown, not obeyed. A topic with no root note is
owned when something owned reached it — unless it is itself a request, which
no served note can claim. The walk reads a reached topic's home once, and
the whole `work-<label>` channel of every owned autolab mission.

## What the realm taught, in order

1. **One task topic, two root notes** — a task re-run for a later routine run
   carries autolab's note (the plan) and Front's (the later run). p3's rule
   would have excluded it from every root; the structural-home rule is for
   it (step 1).
2. **Front serves tasks, not plans** — the plan a run never served is found
   through the task's own note, by reading upward (step 1).
3. **A refusal is a plan, and the answer to a close is the plan as it now
   stands** — p3's retry could only ever be refused (step 2).
4. **A target the close made unreadable keeps its row** — the archived
   channel vanished from its own answer (step 3).
5. **An ordinary Front conversation lives in `#front`**, not in a channel of
   Front's name; the room files it by the roster prefix (step 3).
6. **A topic in an archived channel cannot be resolved** — HTTP 400 on the
   move; it is *kept*, and the request closes over it (step 3).

## Left standing

- No Forge request with an open run existed to close live; the fixtures
  cover it. No force completion, no stopping a running agent — every
  payload still says so.
- The relay's history of operations is memory and a kickstart empties it.
- The screenshots of step 2 and 3 were captured, not looked at, in this
  session; every claim rests on DOM text or a realm read.
- Two August `front-*` conversations reach task topics in archived channels
  and would close with those *kept*; whether to close them is the human's.

## Steps

- [step 1](report1.md) — generalizing the completion target
- [step 2](report2.md) — one completion flow in the existing views
- [step 3](report3.md) — integrate, verify live, deploy
