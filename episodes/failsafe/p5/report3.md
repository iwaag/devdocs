# failsafe p5 — step 3: acceptance independent of the recording route

Date: 2026-09-27 (UTC). Deployed 12:36Z (autolab, gateway, Front, Observer
and relay restarted with nothing in flight; autolab's introduction
re-posted, #13867).

## Adopted semantics

A mission's acceptance is **the decision of whoever holds it**. Anyone may
record it; the recorder changes nothing (`agag.acceptance`, pyagag
`f62acec`, `1677a19`).

- **Holders**: the mission's requester — the one whose root note opened
  the `workplan-` conversation, else its first speaker — and everybody
  that requester was asking *for*, followed up its root notes to the
  request's origin. For a study run through Front, that is Front (the
  runner, entrusted by the routine's guide) and the person who made the
  Front Desk request. The existing mission relations are enough: nothing
  new is written for the ordinary case.
- **Reserved approval**: `agentchat reserve --evidence <their post>` writes
  `[selfnote][approval] reserved <user> (<name>) #<post>` in the
  conversation where the person said it. The note is checked to be the
  person's own post in that same conversation. That person is then the
  **only** holder for every mission opened from there or from the runs
  opened from it. Front still agrees to tasks, which is the task's own
  close, but the mission waits for the person's own words.
- **Evidence** must be:
  - a holder's post;
  - never the mission's own agent (a worker's completion is not its
    requester's acceptance);
  - later than the mission;
  - later than the **last result shown for review**, i.e. the newest
    `+shown=` bound by a task's close-out (p4). An early agreement or the
    initial request accepts nothing.

  An agent acting for somebody carries many requests at once, so its words
  count only in the mission's own conversations, its tasks' or the
  requester chain's. A person's own decision counts wherever they said it,
  e.g. "close out the finished missions" in autolab's entrance.
- **The record** says whose decision it was and which shown result it
  followed: `[selfnote][acceptance] #<evidence> by <user> (<name>)
  after=#<shown>`, then `[state] done`. It is written by whoever recorded
  it, and its sender is the recorder.
- **Kept apart**: task acceptance (the review serving's close, bound to
  the checkpoint, p4), mission acceptance (this record), routine completion
  (the run's end record) and final reporting (delivery plus
  `[delivered]`) remain four records. Step 4 is about the last two.
- **Retries**: an acceptance note already on record is the one that
  counts. A retry finishes the first attempt's record with the first
  attempt's evidence, and a repeat by another recorder writes nothing.

What changed against step 1: Front's `agentchat accept` of its own
agreement is now accepted, and autolab recording the same post writes the
same record (the second recorder writes nothing). The refusal "#… is your
own post" is gone. A bystander's post, a post before the shown result, the
worker's own report and an agent's post from an unrelated conversation are
refused with the reason.

## Tools, guides and the completion door

- `agentchat accept` help and `agentchat --help` say who holds the decision
  and that the requester's own agreement counts. `agentchat reserve` is
  new.
- autolab's last close-out line and its introduction tell the requester to
  record its own entrusted decision itself, with its own post. It is not to
  post it into the `workplan-` topic, which buys a planning run. A
  reserved approval waits for the person.
- Front's `front`/`desk` guides have a short "Who accepts a mission"
  section: entrusted means Front's own agreement after the shown result;
  reserved means `agentchat reserve`, then ask the developer and record
  their words; the initial request is never the acceptance. The
  `routine_run` guide says the same for a runner the guide entrusts. The
  old sentence "the mission's acceptance has to be the developer's own
  words" is removed from both guides.
- `accept.flag` (autolab's planning serving) and `agautolab.mission_done`
  go through the same `accept_mission`, so they get the same rules. The
  first tests to hit the new conversation rule were `mission_done`'s: a
  person asking in autolab's entrance. That is why the rule binds only an
  agent acting for somebody.
- **The operation room's completion door** was reviewed and not changed.
  It records the pressing person's own decision with no post (`#0 by
  <them>`), which is the "holder in person" case. The door is the
  Developer's, and the Developer is the origin of the chain for real
  requests. It does not run the holder check itself (its writer fake in
  the relay's tests cannot trace); a door pressed by someone who holds no
  decision would still record. This is noted as a limitation, not
  exercised.

## Verification

- pyagag 1042: `test_acceptance.py` (15; the step-1 asymmetry test is now
  "the same record whoever records it", plus the bystander and in-person
  cases), and `test_failsafe_p5.py` step-3 tests on the synthetic study
  chain:
  - Front's agreement recorded by Front, then by autolab: once;
  - the task agreement as evidence;
  - the origin person holds it, a bystander does not;
  - the initial request, a pre-result post and the worker's report are
    refused;
  - evidence from an unrelated conversation is refused;
  - a reserved approval refuses Front's words and accepts the person's;
  - `agentchat reserve` records only the person's own post, where it was
    said.
- agautolab 331 (mission_done and accept.flag under the new rules), agfront
  186, agobserver 175, relay 363 on pyagag `1677a19`.

## Not live yet

The concurrent-study trials of step 6 are where Front, entrusted, records
its own acceptance. The reserved-approval scenario is exercised there by a
request that reserves it.
