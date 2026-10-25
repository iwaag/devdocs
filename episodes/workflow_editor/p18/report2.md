# p18 step 2 — durable continuation and graph growth

Plan: [plan.md](plan.md), step 2. Representation: [report1](report1.md).

## What changed

### Routing and delivery (WFE `server/conversationService.ts`)

- **One conductor per room.** A request for the conductor is delivered to
  `conversations.conductor` while that run stands, whether it is working,
  waiting, at rest, or finalized while the room was closed.
  - The answer to its question continues the talk node. Other words are an
    instruction.
  - Only a room without a conductor gets a new run, which is created
    standing (`exec.request … standing`).
  - Creation runs under a per-conversation lock (`conductor-<conv>`).
    Messages racing into an empty room make one run, and the other
    requests are delivered to it.
- **Never replaced silently.** A cancelled conductor's request fails with
  the reason, as does a delivery that finds it ended.
  `wfe conversation conductor <id> --replace --reason …` (HTTP `POST
  …/conductor {action: replace}`) starts a new run whose `standing.replaces`
  names the cancelled one. Retrying the failed request then reaches the new
  run.
- **Late outcomes** (`afterResolution`) continue the same run:
  - a waiting conductor gets an instruction;
  - a resting one is woken;
  - a resting one while nobody facilitates is held until the person
    resumes;
  - a conductor finalized by a closed room that has since reopened is
    reopened first.

  The "new facilitator turn" path is gone. A completed conductor that
  never stood holds the outcome until the step 5 transition gives the room
  a conductor.
- **Room transitions** are activity events (`room` closed/reopened, beside
  `facilitation`). `syncStanding` records the matching standing operation
  on the run, under the receipt `standing:<conv>:<op>:e<event>`.
  - Callers: `close`, `reopen`, a post that reopens the room, `start` and
  `end` facilitation, and `create` from the conductor's attempt
    (`standing.establish`).
  - Reconcile repairs any run whose standing differs from its room.
  - `close`, `reopen` and `endFacilitation` are now async; the CLI, HTTP
    routes and tests were updated.
- **Settlement.** A request delivered to the conductor, or the request
  that started it, is settled `unanswered` once a turn has taken it up
  (an attempt that began after it arrived has ended) without a reply.
  Settlement does not wait for the run to end, since it no longer ends. A
  conductor's reply answers the delivered requests whose messages it names
  in `--re`; words that arrive while it speaks stay open for their own
  turn.
- **Resume** releases what the conductor's standing held. It also wakes a
  conductor that stands suspended, for the outcomes the person resumed.
- **Obligations and status** leave a resting conductor out: the room's
  `working` count is 0 at rest. A standing run receives its own outcomes
  ("later to" is the run itself).
- `ponder --topic <label>` (store **v13**: `activities.topic`) labels a
  stretch for readers.

### The executor (`server/exec.ts`, `server/standing.ts`)

- `afterStanding` is the one rule for continuing a conductor with words no
  attempt was briefed with:
  - with work ready, an ordinary job;
  - with nothing ready or running, a **wake**: `adjustRun` with a
    `derive` callback decides under the run's lock whether the run still
    rests and appends `ponder-w<n>` after the latest completed ponder.
    Unrelated asks, reviews and children are not waited for. Then one job
    `wake:<ws>/<run>/w<n>`.

  It is called from `afterInstruction`, at the end of every attempt
  (words that arrived during it) and from reconcile (a lost wake or job).
- **Integration** is queued when the run comes to rest, as at completion.
  A long-lived conductor's turn reaches `main` without the run ending.
- **Claims** drop a job for a resting or finalized conductor, since there
  is nothing to do.
- **Delegation gating** (`delegate.continuation`): a child's fact does not
  continue a conductor whose room is closed. A fact held at a transition
  waits for release, does not count as `due`, and is not a missed event at
  the next attempt's end.

### Reminders and observation

- `endsWork` (`shared/run.ts`) makes a turn that leaves the run at rest
  count as "completes the run" for fix5/fix6 reminders, so finished-work
  and delegated-result checks still apply. Only fix3's `discussion-open`
  ("a completed run can no longer speak") no longer applies, because the
  conductor speaks again at the next message.
- `wfe observe`: a resting conductor reads `responsible:
  conversation:<id>`. If words arrived and it was not woken, `executor` is
  responsible. agobserver classifies `conversation:` as the new quiet
  condition `idle-input`, never a person wait or a stall.
- A finalization (room closed at rest) is announced as such, not as
  "every node completed".

## Long-lived run audit

| Concern | Finding and decision |
| --- | --- |
| Attempt context | The brief adds the standing facts: whose conductor it is, this turn's wake and its events, and held events "not to be taken up unless the person asks". Delegations, presentations and review rounds list what is open plus the 3 latest closed, and count the rest with the command to read them. `wfe conversation state` shows the current stretch and open work in full and folds earlier completed stretches (`shared/stretches.ts`; `--all` lists everything). |
| Guides | The guide was read from the run's copy, which would freeze it at the conversation's start. A standing run now reads its own guides from its project's committed main, with the commit recorded. If main cannot be read it falls back to the copy and says so. |
| Checkout ownership | One copy for the run's life, as before. Only one attempt runs at a time (fencing and the slot). Integration runs at each rest, between attempts (it refuses while an attempt is open). |
| Workflow version pinning | The definition stays frozen at creation and grows only by recorded revisions. A changed `conduct.yaml` applies to conductors created later; replacement is explicit. |
| Revision storage | Every wake or `--next` writes a revision file, and `run.json` keeps each revision's compiled graph, so size grows quadratically with turns. Measured (one revision per turn): 20 turns 144 KB, 60 turns 780 KB, 120 turns 2.6 MB. Reading was quadratic as well, because the reducer deep-copied every revision per entry. The reducer now shares the copy-on-write `definition` between replayed states: 120 turns check in 0.24 s (1.8 s before). File size stays a known limit. |
| Sessions | `conduct` keeps the `fresh` policy: each attempt starts a bounded session from the records. No model context lasts across turns. |
| Old nodes | Completed nodes are never started again. A wake appends a node, and the evidence of earlier nodes stays. |
| Cleanup | Copies are kept as before. Nothing new to clean up. |

## Checks

- `test/standingFlow.test.ts` (7, through real claims and attempt ends):
  - several messages and replies keep one run; the conductor rests with
    no job while another room's conductor uses the freed slot; a message
    during an attempt wakes the run after that attempt; the graph grows
    `ponder` → `ponder-w1` → `ponder-w2`; integration is queued; the
    observation names the conversation;
  - three posts racing into an empty room make one run; five concurrent
    wakers make one node and one job; a resubmission records nothing new;
  - restarts: a lost wake and a lost queue row are repaired once by
    reconcile; a delivery cut short is delivered once by its receipt;
  - facilitation end/start and close/reopen keep the same run; a child is
    not cancelled; an outcome during the closed period is held, reopening
    replays nothing, and resume continues once;
  - a closed room's finalized conductor is reopened and woken by the next
    message;
  - a cancelled conductor: the request fails with the reason, and the
    explicit replacement names it; retry reaches the new run;
  - a silent turn settles its request `unanswered`, and a reply answers
    only the words it names.
- `test/standing.test.ts`: +1 test (a child's fact arriving while the room
  is closed is held across reopening and released by resume).
- Six earlier tests asserted "a completed conductor is replaced by a new
  run" or store version 12. They now assert the standing behavior: the
  same run is continued or woken.
- WFE: 615/615 tests pass and `tsc` is clean. agobserver
  `tests/test_wfmonitor.py`: 38/38, including the new idle-input case.

## Commits

- agdevworld `f4def9b` (step 2; step 1 was `24ade85`).
- pj-agdev `9e9e767` (agobserver idle-input).
