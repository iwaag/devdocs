# Step 3 report — a mission is accepted by one evidenced record

Plan: [plan.md](plan.md) step 3. 2026-09-24 (JST), by the Omni Agent.

## When is a mission accepted

The requester contract already had two review points, and they stay separate:

- **A task** is accepted in its own topic, by the requester saying it is done. That message is
  what makes the task's run write its report, close it (`completed`, ✔) and start the next one
  (p1). Accepting a task accepts that task only.
- **The mission** is accepted by the requester after its last task is closed. That decision is the
  requester's, and so is its record. The two may coincide in one message only when the words say
  so ("that completes this mission from my side", C3 #9804). Even then the record waits until the
  task's closing report has arrived: the operation refuses while a task is open.

## What changed

| Area | Before | Now | Where |
|---|---|---|---|
| The operation | No close on the normal path. `mission_done` (autolab's entrance, on request) and the relay's completion door wrote `done` with nothing saying whose decision it was | **`accept_mission`**, one operation. Each finished task gets `[state] accepted`, then `[selfnote][acceptance] #<evidence> by <user id> (<name>)`, then `[state] done`, then the mission's conversation is ✔. All writes are selfnotes, so nobody is served. It refuses, writing nothing, on a non-mission, a mission cancelled, replaced or retired, an unfinished task, tasks it cannot all see (serials not 1…N), missing evidence (unless a person records their own decision), and evidence written by the recorder or by the mission's owner, or older than the mission. It is idempotent: an acceptance already on record is reused, so an interrupted attempt is finished by the next with the *first* attempt's evidence, and a repeat after `done` writes nothing | pyagag `agag.acceptance` |
| The requester's tool | Relay the acceptance into the `workplan-` topic (a planning run that answered "noted" and named Front, which bought Front a run), or say nothing | **`agentchat accept <mission> --evidence <post>`**. A refusal leaves an `[opfail]` note at home like every other write command | pyagag `agag.chat` |
| autolab | The last close-out said "the mission waits for your acceptance in workplan-…" | The line says how to record it. A requester who says it *in the plan's conversation* (a person without the tool) is served by the planning run as before, which now writes `accept.flag` → the same operation, evidence = the post that serving answers, topic ✔ after the reply. `mission_done` writes through the same operation with `--evidence`, or the serving's `AGENTCHAT_HOME_ANCHOR` (the human's request to close out), and without either moves nothing | agautolab `zulip_listener`, `mission_done`, `workplan_superdirector` guide |
| The completion door | `accepted` per task, `done` on the mission | The same, with the shared acceptance note (the person pressing, evidence 0) written before `done`. Its preview already read `done` from anybody, so it and the normal path agree | agdevworld `agentroom.close` |
| The contract | "The mission waits for your acceptance" and nothing about how | autolab's introduction has **Closing the mission**: the decision and its record are the requester's, the command, why the acceptance is not posted in the `workplan-` topic, and when a task acceptance also closes the mission. Re-posted in `#agents` (#9983. #9982 is an identical duplicate: the command printed nothing and was run twice) | agautolab `params/intro.md` |

Nothing obsolete had to be deleted from Front's guides: they never described relaying a mission
acceptance. That relay was Front following autolab's close-out line, which now says the opposite.
The only guide change is the superdirector's one line about `accept.flag`.

## Evidence

**Fixtures.** pyagag `tests/test_acceptance.py` (11):
- one record with whose words it rests on, the trace reads the mission `done`, and a repeat writes
  nothing;
- an attempt interrupted before `done` is finished by the next, with the first evidence and no
  second note;
- the door's case (a person, no post);
- seven refusals that write nothing;
- the CLI's success and refusal lines.

agautolab (262): `accept.flag` goes through the operation with the serving's requester post as
evidence, and says so when refused; `mission_done` writes the shared record and moves nothing
without evidence; the test realm now carries the root note `anchor_run_topic` writes. agentroom
(331): the door's writes include the note. agfront 182 and pyagag 768 pass on the new pin.

**Live, on the realm.** One acceptance first (`m9738` on #9804), traced. Then the other
seventeen p1–p3 missions whose acceptance is on record, each on the post where the requester's
side said so:

| Mission | Evidence |
|---|---|
| m7275, m7314 (mediagen) | the Developer: "Mission done from my side" (#7309, #7396) |
| m8084, m8243 (protoprey p2) | the Developer's words relayed (#8229 "The MVP passes", #8275 "Pass") |
| m8298, m8476, m8627, m8671 (protoprey p3) | the stand-in: "Please record the acceptance of mission m…" (#8458, #8501, #8657, #8698) |
| m8741, m8823, m8972, m9017 (robust p1) | "That completes the mission/request" (#8801, #8903, #8994, #9085) |
| m9238, m9395, m9523, m9570, m9738, m9828 (robust p2) | "That completes this mission/request from my side" (#9326, #9500, #9545, #9674, #9804, #9868) |

The writes served nobody: no serving appeared in autolab's or Front's log, because selfnotes and a
✔ are not speech. m8084's cancelled task 1 was left alone.

**Kept visibly pending, no evidence:** m7601 (the research round: #7712 is a decision about
direction, not an acceptance of the round), m7732 (the first build: no acceptance found), m9349 and
m9697 (trials B and B3: only the icon was accepted).

**Observer released on evidence.** On its next look (02:43:55Z) the tracked index went from
**15 to 5**, exactly the requests with something genuinely unfinished:

| Still tracked | Why |
|---|---|
| o8286 | p3's `✔ workplan-locations-2`, a planning twin closed by hand with no state recorded |
| o8512 | `workrun-task2-m8519`, a p3 task still awaiting its requester |
| o8721 | `workplan-setup-robustp1`, the p1 setup conversation (no mission) |
| o9227 | trial B: m9349 unaccepted, and the answer #9368 without a receipt |
| o9562 | trial B3: m9697 unaccepted |

**The door agrees.** `/complete/plan` for C3 reads `work:m9738 done — already done; closing it
again changes nothing`.

**Deployed:** pyagag `86d1d14` in agautolab, agfront (the `agentchat` Front's runs use) and the
relay. autolab's listener and the relay were restarted on it. autolab's startup recovery queued 0.

## Omni Agent work for in-system agents

- did record eighteen mission acceptances for agent Front, with Front's credential, each on the
  requester-side post that gave it. The missions predate the operation, and Front would have
  relayed them into the plans. Not a handoff candidate: a one-time backfill. New missions are
  recorded by Front itself (step 5).
