# Robust workflow p2 — phase report

Plan: [plan.md](plan.md). Steps: [1](report1.md) gaps reproduced · [2](report2.md) identity by id ·
[3](report3.md) retention and positive recovery · [4](report4.md) monitor health watched from
outside · [5](report5.md) the final revision under combined failures · [6](report6.md) rollout.
Executed 2026-09-24 (JST) by the Omni Agent; models and harnesses of every in-system role left as
they were (Front, autolab, forge on Sonnet 5 through claude_code; Observer on the host's
`qwen3.8:27b-mxfp8`).

## What changed

| Area | Before | Now | Where |
|---|---|---|---|
| Identity | root notes, served marks, the trace's homes, Observer's requests and incidents joined on topic **names**; a callback serving carried the id of the post that named it elsewhere as its home anchor (23 of 64 anchored notes) | a record means the conversation its **post** is in now (`agag.identity`); served marks match by the post they name; a root note by its `#anchor`, never a conversation that began after it, and through `[replaces]` to a retired predecessor; callback servings anchored in home; autolab task and forge run notes carry the plan's id; a request is its origin's first post and an incident is (request, stalled conversation's anchor) with the kind an attribute | pyagag `agag.identity`, `listen`, `trace`, `topics`, `zulip`; agautolab, agforge, agfront; agobserver `monitor` |
| Retention | the 12 h window was both discovery and memory; a ✔ or renamed origin ended tracking | a persisted index keeps every request with open work or an open incident, whatever its age or name; a ✔ request with open work is reported (`origin_closed`) | agobserver |
| Recovery evidence | *rescued* = the candidate no longer produced; an answer "taken up" when the requester spoke at home since | *rescued* only when a fresh look reads the stalled conversation in the state its kind waited for; *cancelled* on a recorded decision; unreadable is said and reported, never recovered; a stale mirror asks nobody; an answer is taken up only by a **served mark** — and an `undelivered` ask carries `[selfnote][owed]`, which the requester's listener turns into that mark after the serving that read it replies | agobserver; pyagag `trace`, `listen`, `selfnote` |
| Callback delivery | a mention was judged by reading its topic by name, twice, across a ✔ that could move it in between; the trace ignored the parent hop through which a task autolab started itself reports to Front | the mention route judges its trigger post by id first; the trace follows the parent's requester one hop | pyagag `listen`, `trace` |
| Monitor health | the monitor's own liveness was nobody's: a thread in Observer's process, judgments (29–128 s) inside its loop, nctl reading a status file in the wrong place (`unobserved`) | a health record at every look and judgment; judgments on their own worker; the always-on relay evaluates the record (`stopped` / `stalled` / `judgment_stalled` / `unable_to_observe` / `degraded` / `missing` vs `ok` / `idle` / `disabled`), shows it on the ops board and `/healthz`, and DMs the owners; nctl reads Observer `polling` | agobserver; agdevworld `agentroom.watchdog`, ops board; launchd |
| Operator tools | — | fault hooks `monitor-stop`, `triage-stall`, `mirror-stale`; `python -m agobserver.withdraw` for an incident opened in error | agobserver |

## Versions

pyagag `deec513` in all seven consumers; agobserver `0bf46af` (code `dc21380`); agfront `243c9dd`,
agautolab `a507617`, agforge `62d7409`, agdevworld `e6643b4`, archsage `645969a`, cagent
(`pj-clusterintent`) `c27f04a`. All running on it, verified from the installed venvs and the process
start times (report5, report6).

## Evidence

**Fixtures.** Ten reproductions written in step 1 as strict-xfail tests (R1–R9 in the monitor, L1 in
the listener) — every one now passes, and each mark was removed by the change that fixed it — plus
the step 2–5 fixtures: pyagag 757, agobserver 108, agautolab 258, agforge 265, agfront 182,
relay 330, archsage 28, cagent 202 passing.

**Live, final revision** (attempt 3; report5):

| Scenario | Required | Result |
|---|---|---|
| Candidate unreadable or changing failure kind | no false recovery; the blockage explicit | a ✔'d task judged a deliberate close became `undelivered` on the same incident and allowance, asked, rescued on its receipt. Unreadable: fixture only |
| Origin and child rename/resolve; old name reused | tracking and delivery stay with the anchors; no duplicate incident or work | origin renamed (A3), task renamed mid-run (C3), task and origin ✔'d (C3): every report, request and callback went where the anchors were; the reused name got its own answer; one incident per stalled conversation |
| Inactivity past the window, then restart | unfinished work tracked, resumed once | fixture (time-compressed); live, the index survived three restarts with nothing dropped and nothing re-asked |
| Answer during another serving / unrelated home reply | only processed input marked handled | answers read by a serving that the owed note named were marked by that serving; the mention route did not serve them again (attempt 3); R8 fixture |
| Monitor thread stops, process alive | independent reporting | relay DM **5 min 10 s** after the stop, naming the process as alive |
| Triage stalls / mirror stale | others covered, degradation explicit | held judgment DM'd at **5 min 5 s** with looks continuing; stale DM'd at 1 min 34 s (injected), nobody asked |
| Human wait, cancellation, completion | no unnecessary recovery; distinct evidence | human waits up to 22 min on Front's question and every acceptance wait: no request; cancellation by fixture; completions ended in *done* |

**Measured.** Undelivered answer → request **6 min 40 s / 6 min 51 s**; request → receipt 15 s and
34 s after Front's listener returned; ✔ origin → report **7 min 43 s**; dead worker → `silent`
**45 min 17 s**, two requests, report at 67 min, one serving on restart. Oldest unchecked tracked
request at each look: under a second (every tracked request is looked at every cycle; a look over
15–18 requests takes 0.2–0.3 s off the mirror). Judgments 52–62 s once not held.

**Outcomes on the final revision:** 2 rescued (both on a receipt), 1 dismissed (a human ✔ on a task
waiting for acceptance, judged a human wait), 2 reported (a ✔ origin — then *moving again* after
the human's un-✔; a dead worker — *moving again* after the operator ended the fault). **0 false
rescues, 0 duplicate work, 0 lost tracked requests, 0 Omni Agent rescue.**

**Cost.** 120 runs over the three attempts and the deployment, $10.22 (final attempt: 51 runs,
$4.58). The monitor reads nothing from Zulip per look; it posts for incidents and requests only;
one `subscriptions` read per 10 min. The watchdog reads files; 9 DMs in the whole step.

## Prevented, recovered, cancelled, stopped

**Prevented** (cannot happen as it did):
- a renamed callback topic served again after a restart (L1); a served answer reading undelivered
  after a rename (R9); a reused name inheriting another request's threads, root notes, marks or
  incidents (R6, `remotes_for_home`);
- a stall closed as rescued because it became unreadable (R1), changed kind (R2), was renamed (R7) or
  its origin was renamed or reused (R5, R6); an unrelated reply consuming an answer (R8);
- an incident abandoned because its request aged out (R3) or its origin was ✔'d (R4);
- a callback lost to a ✔ landing between two reads (attempt 1, B);
- a dismissed judgment hiding a mechanical stall on the same work (attempt 2);
- a judgment stalling the look at every other request (step 4);
- a stopped or stuck monitor going unnoticed (step 4, live three times).

**Recovered in-system** (it still happens; it is found, asked about and verified): an answer
never served because the requester's listener was down (A3, B3; attempt 2 twice); a lost
callback (attempt 1 B — recovered, but reported for want of a receipt, which is what `[owed]`
fixed); a missing start (recovered by Front itself before Observer's grace).

**Cancelled / decided:** none live; a recorded `cancelled` closes an incident as a decision
(fixture). Nine p1-era incidents and their nine second episodes were **withdrawn** by the operator —
opened by the monitor on records the new receipt rule could not read (report5).

**Stopped (reported):** a ✔ origin with open work (C3); a dead worker (E3); one unnecessary report in
attempt 1 (a recovery with no receipt, before `[owed]`).

## Remaining limitations

- **Missions never close.** Nothing records a mission `done` after the requester accepts its last
  task, so every mission's workplan stays `awaiting_requester` and its request stays tracked for
  ever — about 20 ms per look each and no candidates, but the index only grows (15 after this
  step).
- **Discovery is `#front` only.** Routine runs, argues and Project Room posts are not looked at.
- **The listener's startup recovery skips ✔ topics**, so a callback its listener skipped *before*
  this phase's fixes, or that is lost some other way in a ✔ topic, comes back only through
  Observer's ask and its owed receipt.
- **An answer read without a receipt is served once more.** An owner-route serving that reads a new
  answer through its threads — not named by an owed note — leaves the mention owed, and the next
  serving says "nothing new" (A2, A3; one short run each).
- **Judged kinds rest on a 27B local model**: a human ✔ on a waiting task was *legit* here and a
  *stall* in p1. Consistency across similar cases is not measured.
- **Staleness was injected**, not produced by a real expired event queue; the injected flag has no
  `stale_since`, so its detection time is not the real one.
- **nctl still sees only listener liveness**; the monitor's health lives on the relay's board and in
  its DMs. The relay's DM to the owners extends its bot's posting contract (until now
  `#ops-testbed` only).
- Remaining name joins (report2): `Conversation` equality, autolab's worklog and forge's record
  writing through `live_topic_name` (an open twin wins), the listener's queue keyed by name,
  `listen.owed` preferring an open twin, Front's routine discovery.
- Front addresses Observer (`@**agobserver-agstudio1**`) when it answers Observer's asks — noise,
  no cost (Observer's mention route answers argues only).

## The next expansion

What recurred in this phase was not identity any more — the trials found no name join once the
paths were on anchors — but **receipts**: who has taken what up, recorded where a program can read
it. The lost callback, the unverifiable recovery, the double serving and the nine false incidents
were all a receipt missing or imprecise. The largest receipt still missing is the **mission's own
close**: it is why every request stays tracked, why "done" is never reached for a mission, and why
Front's final acceptance still costs autolab a run that only says "noted" (p1). Mission-close
simplification therefore now has priority over wider discovery; wider discovery (routines and
argues first) would inherit the same unbounded retention without it.

## Omni Agent work for in-system agents

- did the requester's part in every trial (requests, acceptances, reading the working tree, looking
  at icons) for Front — the Developer's role; not a handoff candidate beyond the Project Room.
- did operator fault injection and listener stops/starts — an operator's trial work; not a handoff
  candidate.
- did withdraw eighteen incidents that the monitor opened on pre-p2 records for agent Observer —
  handoff candidate: the monitor could withdraw an incident itself when a later revision's rule reads
  the same records as fine, instead of an operator doing it by hand.
- did ask nctl for one observation refresh of this host so the status-file fix could be read, for
  the cluster observation path — not a handoff candidate (the six-hourly job does the same).
- did fix nine defects found by the trials (report5) — an Omni Agent's job in a development
  episode; the incident topics that surfaced them stay in Observer's channel.
