# failsafe p1 — step 3: the recovery loop through Front and autolab

## What Observer brings to Front

**Where.** A recovery request for `unheld`, `quiet` or `silent` goes to the
conversation **Front itself holds closest above the stalled work**
(`Monitor.asked_where`). For a routine that is the `routinerun-` topic, not
the Front Desk above it. Anything Front sends while serving a conversation
is anchored to that conversation. A resumption asked for from the Front Desk
would therefore bring the task's answer back to the Desk, and the run
waiting for it would never hear it. With no Front-held conversation between
the stalled work and the origin, the request goes to the origin as before.
The older kinds (`undelivered`, `unstarted`, …) keep the origin, since
their receipts are bound to it. The incident topic records where each
request went.

**What.** The request (`Monitor.request`) now carries:

- the stalled conversation and the request, as Zulip conversation links;
- what the records show;
- the **execution**: whether a serving is open or its last one ended, and
  who holds the work;
- **what is still owed**: a unit of work is unfinished until its owner's
  record ends it, and "a reply, a ✔ or this request ends nothing";
- expected next, responsible, the `agentchat trace` evidence;
- **what is not known**:
  - after an ended serving: whether anything it started still runs, so
    check before starting again and continue from what was left;
  - while a serving is open: whether it is alive, so ask its owner or wait,
    and never start a second run beside it.

It still names nobody. It is request *n* of 2.

**When a person has been asked.** `unheld` is not raised again once
somebody has put an explicit question to a person about the work since it
stopped (pyagag `9b79405`). The incident stays open without further
requests. If the person answers and nothing moves, the stall is due again
and the second request follows.

## What Front and autolab were given

Tool giving and evidence-driven guidance. No harness failure is scripted.

- **Front** (`desk`, `front` and `routine_run` guides): a section "When
  Observer says work has stopped". Check with `agentchat trace` and `read`,
  then choose one:
  - resume stopped work by posting into **that same conversation**: a new
    serving of the same work, with what the stopped one left (autolab's
    mission copy), asked to check what was left and continue rather than
    redo;
  - do not start a second run beside an open serving; ask its owner only
    when nothing has shown work for long;
  - ask the developer with a response request when it cannot go on or the
    choice is theirs.

  The routine guide's rule "never post a second start into a running topic"
  now names this exception. The guide also says what Observer counts as
  recovery (fresh work; not an ack or another promise) and that it reports
  to the owners after two requests.
- **autolab** (`workrun_supercoder` guide): "Work in flight when your reply
  ends":
  - m11741 as evidence: in this mode the reply ends the serving and what it
    had running. So wait for every subagent and background command before
    replying, and say work is still going only when something that will
    wake the task holds it (another agent, the notifier watch);
  - on resumption: start from what the chatlog says was in flight and what
    the copy holds, check nothing still runs, keep what is finished.
- autolab's resumption path needs no code: a post in a `workrun-` topic
  serves the task again in its mission copy, which keeps the stopped
  serving's uncommitted work. The autolab listener is serial, so a post
  made while a serving is still open waits for it and never runs beside
  it.

## Attempts, verification and intervals

All of this is persisted in the incident record and survives a restart
(step 2 tests):

| what | value | why |
|---|---|---|
| look interval | 120 s | unchanged; the mirror makes a look free |
| `unheld` grace | 300 s | a listener re-serves input that arrived mid-run within seconds; five minutes covers the rest |
| `quiet` | 1800 s of no movement below the origin | long enough for a normal serving plus a callback |
| `silent` | 2700 s | unchanged |
| requests | at most 2, 600 s apart, then a report to the realm's owners (the existing human-reporting path) | unchanged bound |
| legit postponement | 1 h, doubling on unmoving evidence to 4 h; reported at 6 h | a judgment postpones a review, never ends it |
| judgment deadline | 900 s, then `unclear` (two → reported) | the review never waits on a judge |
| recovery (`unheld`/`quiet`) | a serving after detection that showed work while open, or ended handing the move on | an ack or a second promise is not recovery |

After a rescue the request stays tracked. Only the obligation's record
releases it, and a new stall of the same work is a new episode of the same
incident.

## Deployment (16:03 UTC)

- `nctl status` ok before the restarts. The drift on agstudio was converged
  at step 2.
- pyagag `9b79405` is in every consumer that reads or writes post lines:
  - agfront `2583175`, agautolab `a971bdd`, agforge `eb7f20b`;
  - agdevworld relay `42fc532`, archsage `241dd02`, cagent (pj-clusterintent
    `c656da4`), agobserver (pj-agdev `671034a`).

  comfynotify keeps its pin: it parses no post lines. The consumers' tests
  compare reply words without the new mark through a small `endmark.plain`
  helper; the pyagag tests assert the mark itself.
- Tests at deployment:

  | suite | passed |
  |---|---|
  | pyagag | 934 |
  | agfront | 180 |
  | agautolab | 304 |
  | agforge | 265 |
  | archsage | 37 |
  | cagent | 204 |
  | relay | 353 |
  | agobserver | 130 |

- Restarted, each alone, with no serving in flight:
  - Front, autolab (listener and gateway), forge (listener and request
    service), archsage;
  - cagent (listener and API), the relay, Observer.

  Every listener came back and recovered its queue.
- autolab's startup recovery reported `work-m11741 › workrun-task1-m11741:
  nothing owed now; skipped`. The truncation change to `owed_start` did not
  re-serve the held live stall, because its serving journal records the
  delivery. m11741 stays held (`agobserver.hold`) and untouched.
