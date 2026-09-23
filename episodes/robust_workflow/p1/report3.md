# Step 3 report — the per-task start relay is gone

Plan: [plan.md](plan.md) step 3. Evidence: [report1.md](report1.md) (F1–F3, D2–D4),
[report2.md](report2.md). 2026-09-24 (JST), by the Omni Agent. Two listeners (autolab, Front)
restarted on the new code; autolab's introduction re-posted. No task was run.

## The boundary chosen

report1's handoff map: 28 of Front's 33 posts to other agents in p3 needed no judgment, and
the only one whose omission was silent was the **task start** — 10 of them, plus the separate
"may start" post. Every p3 stall and at least six earlier ones sat there. The requester's
decisions around it are real (the plan may run; this task is accepted); the post that turns
those decisions into a start is not.

**New ownership: autolab progresses an authorized mission itself.**

| Moment | Before | Now |
|---|---|---|
| Requester says the mission may start | autolab marks it started; **requester posts into task 1** | autolab marks it started **and starts task 1** |
| Requester accepts task N | autolab closes N; **requester posts into task N+1** | autolab closes N **and starts N+1** |
| Requester wants N+1 to wait | (implicit: not posting) | says so when accepting; autolab marks N+1 `held`; a post there starts it |
| Requester posts "start" anyway | a start | recognised: "already under way", nothing doubled |

Front keeps interpretation, coordination and the human conversation; acceptance stays the
requester's (autolab still closes nothing without it). For p3's first request this removes
6 of Front's 12 task-related posts (5 starts and the separate permission post), and every
Front run that existed only to make them.

## Implementation

### pyagag `99b938d` — an owner may start its own conversation

- **`[selfnote][start] #<because> for <user id> <name>`** (`agag.selfnote.start_note`,
  `owed_start`). Until now a conversation could be started only by somebody else's post,
  because every listener skips a topic whose last speaker is itself. The start note is the
  owner saying "I owe this conversation a serving", and naming who the answer goes to.
  - `agag.listen`: intake enqueues the owner's own start note in a conversation it owns;
    `owed()` counts a start no serving has answered (on the journal path too, by the input
    boundary); recovery at startup and after a resync re-queues one. An ack or progress lines
    after the note do not spend it, so **a crash mid-run leaves it owed** — the rule every
    other serving already follows.
  - `agag.topics`: the empty-topic guard counts a start as input, and `requester_of` falls
    back to the note, so the report of a self-started task names the requester.
- **Callbacks find home through the parent** (`agag.zulip.parent_rootchat`). A task autolab
  started holds no note of Front's; its report names Front; Front's mention route now follows
  the opener's root note (task → `workplan-`) one hop and finds Front's anchor there. Own note
  first, then the `replaces` hop, then this — never an override.
- **Resolve is request-aware; un-resolve exists** (D2–D4):
  - `agentchat resolve` refuses while the caller's own newest post is unanswered ("your post
    #8292 is the newest word here and autolab has acknowledged it and is working on it. A
    resolve only renames …"), `--anyway` for a conversation known to be finished. The check is
    the request's state, not the topic's age.
  - `agentchat unresolve <channel> <topic>` renames `✔ x` back to `x`, refusing when a twin of
    the bare name exists (the rename would merge them). Checked live on `#memo` with Front's
    bot credential: resolve and un-resolve both work, including other senders' messages.
  - The `send` refusal no longer says "it is finished … a new request goes in a new topic";
    it names `agentchat unresolve` as the correction and a new request only if the work is
    really finished. That sentence is what forked p3's twin.
- `agag.trace` reads start notes (`queued` / `executing` / answered-for) and `held`.

### agautolab `85f53a3`

- `start_first_task` on `start.flag`; `start_next_task` at the end of a task's close-out,
  after the devlog record. Each posts one visible line **naming nobody** (a mention would buy
  the requester a run to read it) and the start note naming the requester who authorized or
  accepted. Written inside the close-out, so the start is durable the moment the task closes;
  no scan of old missions is ever needed (several accepted missions still carry `started`).
- Nothing starts when the mission was never started, the task is already under way (any other
  speaker, our ack, or an earlier start note), the task is `held`, or nobody accepted by name.
  Each case is one close-out line saying so.
- `hold.flag` (supercoder) → the next task is marked `held`.

### Guides and the public contract

| Where | Change |
|---|---|
| autolab introduction (re-posted) | "When you say the mission may start, I start task 1 myself, and after that each next task when you accept the one before it … nobody posts a start"; how to hold one; reports from tasks it started reach the asking conversation |
| autolab `workrun_supercoder` | closing starts the next task; `hold.flag`; a start request for work already done or running is answered with where it stands |
| autolab `workplan_superdirector` | what `start.flag` now does |
| Front `front` guide | **deleted** the p3 paragraph (post id or post now; never resolve a fresh topic; a resolve is a rename) — the tools now carry both; replaced the retired `agentchat wait` with `agentchat trace`. 1,790 → 1,657 words |
| Front `desk`, `routine_run` | unchanged: neither asked for per-task starts |

Shared contracts stay in one place each: progression in autolab's introduction, resolve
semantics in `agentchat --help`.

## Verification

| Check | Result |
|---|---|
| pyagag | 730 passed (+19: `test_progression.py` 8, listener start-note 3, agentchat resolve/unresolve 6, trace 2) |
| agautolab | 256 passed (+9: `test_progression.py` 7, close-out wiring 2); the start-flag test rewritten to the new contract |
| agfront | 182 passed on `99b938d` |
| Interruption / retry | listener tests: a start note is served once and not again after a restart; a start acked and then crashed is re-served by recovery; another bot's start note is not ours. autolab: a repeated close-out, and a start that arrived by post first, start nothing twice |
| Live permission | un-resolve by Front's bot over another sender's message, `#memo` probe (inert channel) |
| Deployment | agautolab and agfront locked to `99b938d`, listeners kickstarted 16:44:25Z, startup recovery queued 0 in both (no old task was started) |

**Not yet verified live:** a mission progressing through its tasks. That is step 5's first
case, run through the ordinary entrance.

## Remaining in this area

- **Mission close is still a relay.** After the last task autolab says the mission waits for
  acceptance in the workplan; Front relays it; nothing writes `[state] done` (p3's four
  missions still read `started`). Left: acceptance of the whole mission is a separate review
  point in p3's record, and the missing `done` does not stall anything.
- **forge's plan approval → run start** stays one post by the requester: there the approval
  *is* the permission, not a relay.
- agforge, archsage, agobserver and cagent still pin older pyagag; their runs lack
  `unresolve` and the resolve guard until step 6.
