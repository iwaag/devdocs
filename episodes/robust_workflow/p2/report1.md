# Step 1 report — the gaps, reproduced, and what counts as evidence

Plan: [plan.md](plan.md) step 1. 2026-09-24 (JST), by the Omni Agent. Nothing deployed in this
step; the reproductions are fixtures committed beside the code they exercise.

## Where things stand

| Check | Result |
|---|---|
| Listeners | all six on pyagag `87ac87e` (p1 step 5's last fix), running since p1's rollout; the monitor's newest log line is an hourly re-judgment at 20:02Z. It logs nothing on an idle tick |
| `nctl status` | Nautobot, worker, dumps healthy; node observations 5.5 h old (the refresh job runs every 6 h) |
| `nctl drift` liveness | Front, forge, autolab (this host), archsage, cagent `polling`; autolab on the VM `stale`; **Observer `unobserved`, reason `no_status_file`** |
| p1 incidents on record | 7: 2 rescued, 1 reported, 1 rescued (`resolved_live`, un-✔), 3 dismissed; 2 of the dismissed ones are re-judged every hour on the local model because the p3 twin they concern stays ✔-while-answered |
| Local judgments | 7 `triage` runs: 29, 33, 33, 37, 43, 54 and **128 s** — each inside the monitor's loop, which waits for it |
| Trace cost off the mirror | all 58 `#front › front-…` conversations traced in 0.94 s, no Zulip call |

**Why Observer is `unobserved`.** nodeutils reads `<workspace>/.local/agag-status.json`. Observer's
declared workspace is the superproject `pj-agdev` (it is a plain directory there, not a
submodule), but its listener writes `agobserver/.local/agag-status.json`. The file is fresh — it
just is not where the collector looks. Even found, it would say only that the listener's intake
polled: it is written by `agag.listen`'s intake loop, which knows nothing about the monitor
thread. A dead monitor and a live one write the same file.

**Why "rediscover everything" is not a retention strategy.** Of those 58 conversations, 46 have
at least one unfinished node and the trace produces 74 candidates today: 70 are `resolved_live`
on conversations whose origin is itself ✔ and older than 12 h (history closed without a finished
`[state]`), 2 `unacknowledged` in ✔ origins, 2 `resolved_live` in open ones. Retention has to
start from work that was actually being tracked, not from a rescan of the realm.

## Reproductions

Each is a fixture that states the required outcome and is `xfail(strict=True)` today, so the fix
has to remove the mark: `agobserver/tests/test_p2_reproductions.py` (R1–R9, the monitor over a
fake realm and a real mirror, built on p1's `stalled_realm`) and
`pyagag/tests/test_p2_reproductions.py` (L1, the listener). All ten fail for the stated reason
(checked with the mark removed). Suites otherwise green: agobserver 82 passed, pyagag 741 passed.

| # | Scenario | What happens now | Path |
|---|---|---|---|
| R1 | the stalled task becomes unreadable (the mirror returns nothing for it) | trace reads it as `not_started` with no identity, the `unstarted` candidate vanishes, incident **rescued** | `MirrorReader.topic_history` → `trace.read` → `classify([])`; `Monitor.tick` closes on absence |
| R2 | the failure changes kind: somebody posts into the task, autolab never acks | `unstarted` **rescued**, a new `unacknowledged` incident opened with a fresh allowance of two requests | `Candidate.key` includes the kind; rescue = key not seen |
| R3 | 13 h pass with the incident open | the origin leaves the 12 h window; the incident is never looked at again, neither rescued nor reported | `Monitor.origins` window; rescue loop only for looked-at origins |
| R4 | the origin is ✔ while task 2 is still unstarted | `origins()` skips ✔ conversations; incident stays `recovering` for ever; nobody is told. (The handler's own "origin is ✔ → report" branch is unreachable.) | `origins(include_resolved=False)` |
| R5 | the origin is renamed and the task then starts | rescue never recorded: the record's origin key is the old name | record `origin.key = "front/<name>"` |
| R6 | the origin is renamed and a new request takes the old name | the new conversation is looked at as the incident's origin: incident **rescued**; and task 1's answer is read against the *new* conversation's served marks, opening a false `undelivered` incident | origin key by name; trace reads the requester's home by name |
| R7 | the stalled task itself is renamed | old incident **rescued**, a second incident opened, fresh allowance | `Candidate.key` = kind + channel/bare name + evidence id (`(0,)` for `unstarted`) |
| R8 | an answer names Front; Front then speaks at home about something else | `_taken_up` accepts the later speech; `undelivered` **rescued** although nothing ever read the answer | `trace._taken_up` |
| R9 | an answer served (mark written), then its topic renamed | the mark names the old topic name; the answer reads `awaiting_delivery` again: false `undelivered` | `trace._served_marks` compares names |
| L1 | a callback served and marked, then its topic renamed, listener restarted | the finished exchange is **served a second time** | `Listener.served_marks` keyed `(channel, bare name)`; `recover()` |

All ten are code-review hypotheses made concrete; none was observed in production during p1,
which did not rename origins or tasks mid-incident and never ran a request longer than 12 h.

### Code-review findings checked against the live realm

| Finding | Status |
|---|---|
| A run served by a callback gets `AGENTCHAT_HOME_ANCHOR` = the **remote** mention's id (`agent.py` sets it from the journal's `trigger_id`, which on the mention route is the remote post), so its root notes name home with a foreign anchor | **observed**: 23 of the realm's 64 anchored root notes carry an anchor that lives in another conversation (all in another channel) |
| The same `trigger_id` is the reply's destination anchor (`topics.py`, `replies_here`); `locate` accepts an anchor found in the same channel as the home, so a callback from a topic in home's own channel would be answered in the caller's topic | **hypothesis**: 0 such anchors on record; nothing in this realm delegates within one channel today |
| autolab's `handle_mention` serves the root note's home by name without `locate` or a ✔ check and writes `note_served` for the newest message, not the trigger | code fact; not reproduced |
| `resume_prepared` redelivers to the reply name frozen at prepare time | code fact; the explicit_reply fixtures cover a rename before prepare, not between prepare and redelivery |
| `listen.owed` prefers the open twin, so a mention in `✔ x` beside an open `x` is judged against the twin | code fact; twins are what p1's `unresolve` folds, rare now |

## Identity today

**Where identity is already an anchor**: `trace`'s root lookup; `zulip.locate` / `conversation_of`
/ `inherited_rootchat`; the owner route's reply destination; Front's `handle_mention` (locates
home by the root note's anchor); autolab's mission/task notes (`mission_tasks`); forge's
`request_of_run` / `origin_of`; Observer's watches and destinations (p1 ex1).

**Where a name still joins** (the ones that matter for this phase):

| Area | Join | Consequence |
|---|---|---|
| `Conversation` | `==` / hash on `(channel, topic)`; the anchor is ignored | every dict or set of conversations is a name join, anchor or not |
| served marks | written `[served] <remote name> <trigger id>`; read by `(channel, bare name)` in three readers that disagree (`trace` and `Listener` strip ✔, `zulip.served_marks` does not); four writers, two of which mark the newest message instead of the trigger | L1, R9; continuation shows a finished thread as "answered, not dealt with" after a rename |
| trace | children found by the root note's *name* (`_anchored_children` ignores `#anchor`); the requester's home read by name | R6; after a retire the replacement shows the retired work's tasks |
| Observer monitor | origin by live name, incident by `Candidate.key` (kind + name), rescue = key absent from an origin looked at by name | R2–R7 |
| root notes → home | five implementations (`rootchat_notes`, Front's routine `MirrorReader.rootchats` — a copy —, `trace._anchored_children`, `routine.origin_of` via `own_rootchat` which ignores `[rootchat-moved]`, `rootchat_home`) | a reused `front-…` name inherits the old request's threads, continuation and routine reports |
| anchor → conversation | four implementations; autolab's and forge's reduce the anchor's exact `✔` location to the bare name and then `live_topic_name` it | a write can land in an open twin although the anchor had found the real conversation |
| "follow ✔ by name" | a dozen places that disagree about twins (prefer open, prefer bare-if-non-empty, merge both) | readers see different conversations under one name |

## What counts as evidence

The monitor has to tell five outcomes apart. The rule for all of them: **absence is not
evidence**. A candidate that is no longer produced tells nothing until a fresh, complete look
says *why*.

| Outcome | Evidence required | Not evidence |
|---|---|---|
| **Recovered** | a fresh look (mirror `live`, every conversation on the incident's path read with complete coverage, located by anchor) shows the transition the incident was waiting for: `unstarted` → a post in the task acknowledged by its owner, or the task terminal; `unacknowledged` → the owner's ack or answer after the evidence post; `undelivered` → the requester's served mark covering the answer (matched by id, not name); `failed` → a newer post the owner acknowledged, or a terminal decision; `resolved_live` → un-✔ with the work live, or a terminal state recorded; `silent` → the owner's answer or progress after the last sign of work | the candidate vanishing; the requester speaking at home; a rename; the origin leaving discovery |
| **Cancelled / decided** | an explicit terminal record: the owner's `[state] cancelled/replaced/retired`, the requester's `accepted`/`done`, or a human's decision in the origin | a ✔ on the origin or the task by itself (a ✔ renames; it stops nothing — p1) |
| **Observation failure** | the path cannot be read (anchor not held, coverage incomplete, channel unknown) or the mirror is stale | — it is reported as *unobservable since …*, never as recovered or as not started |
| **A different blockage** | the same stalled work (same anchor) now fails a different check | — it continues the same incident, with the same allowance of requests |
| **Escalated** | requests spent, nobody to ask (origin ✔, the owner is the one not answering), or unobservable past a bound | — reported once to the realm's owners, then no more requests |

## Acceptance criteria for steps 2–4

Step 2 (identity):
- An incident is keyed by the stalled work's **anchor** (its identity note or first message) and
  the request's **origin anchor**; kind is an attribute that can change. A rename of either, a ✔,
  a restart or a lost store does not open a second incident or reset the request allowance (R7).
- The origin is located from its anchor on every look; its old name reused by another request is a
  different request (R5, R6).
- Served marks match the remote by the id they carry, not the name (L1, R9); one reader, one rule.
- A callback serving's anchor is a message in **home** (no foreign `AGENTCHAT_HOME_ANCHOR`, no
  foreign destination anchor); the trace follows root notes by anchor where one exists.
- Existing behavior kept: p1's automatic task progression, delivery recovery, and every existing
  fixture.

Step 3 (retention and positive recovery):
- R1–R4 and R8 pass: rescue only on the evidence above; unreadable is `unobservable`, not
  recovered; a different blockage continues the incident; the origin leaving the window or being
  ✔ does not end tracking; later speech at home does not consume an answer.
- A small persisted index of tracked requests survives restart and is rebuilt from the incident
  notes and the index itself; the realm's historical debris (the 74 candidates above) is not
  adopted wholesale.
- Request allowance and history survive restart; an unobservable or exhausted incident is
  reported once and not re-asked.

Step 4 (monitor health):
- The monitor writes a health record every cycle (cycle start/end, duration, source freshness,
  requests tracked/unchecked, oldest unchecked age, latest failure); a judgment cannot hold the
  other requests hostage (bounded or moved off the look loop).
- A process independent of the monitor evaluates the record and shows stopped / stalled /
  unable-to-observe distinctly from idle; nctl stops reporting Observer as `unobserved` for the
  wrong reason.
