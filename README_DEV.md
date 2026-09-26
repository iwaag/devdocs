# Rule

- Don't expose local pc/cluster info in non-ignored files.

# Adopted Deveopment Style

Standard AG Style

# refs
devpolicy/terms.md ... Read only when you need to check terminologies

# In-System Agents

## ComfyUI notifier

`comfynotify` is a host-local tool, not an agent: it persists a ComfyUI
ticket and the dedicated notifier bot posts its terminal result into the
requesting Zulip topic. A generation run tickets and exits; the normal topic
callback mechanism serves the receiving agent again.

Since `zulip_command` (2026-09-01) the ticket is opened by **posting one line
in the topic** — `@**Comfy Notifier** watch <prompt_id>` — so nothing has to
be handed the CLI, and the notifier acknowledges with a reaction rather than a
post, because a bot post in a run topic would serve that run early. Public
channels only — and `#front` turned out to be one of them, so a command it
cannot read is refused **once per topic** and then not again, which is what
keeps a refusal from waking an agent that answers by naming the bot. A mention
inside a code fence is not a mention, which is how the command is quoted
without firing it.

A new agent is `agag init <agent> --yes --provision --like <sibling>`
(pyagag): it generates a project on the shared skeleton (`agag.agent`),
copies compatible local machine facts, creates its Zulip bot and channels,
and writes its ignored credentials. Since 2026-08-30 it also files the
instance's own channel in the `agents` channel folder and subscribes the
realm's organization owners to it — a new agent's entrance is watched by the
humans who own the realm, never by the account that provisioned it. The owner-class identity is the **path**
in `AGAG_ZULIP_ADMIN_ENV`; autolab receives the dedicated provisioner path
and can run the whole chain from a project workplan (`agag_builder` p3).
`agecho` was the minimal p1 fixture and runsmoke1's `main/agping/` the p3
agent-created one that Front reached successfully. **Both are retired**
(`refactor` p3 ex1 for agecho): they proved a generated agent can be found and
talked to, and an agent kept alive only to keep proving that costs every
board a row and every sweep a read.

## Reading Zulip through mirrors (`better_zulip_call` p1, 2026-09-14)

Every process on this host that reads the realm holds a **mirror**
(`pyagag` `agag.mirror`): a persisted, event-updated copy of the realm's
public conversations on that process's own credential — one event queue
registered for all public channels, one paged read of the realm to fill the
store (62 calls, a second, for this realm), and events from then on. The
relay's boards and completion previews, every listener's intake and
recovery, cagent's topic listener and the Observer's schedule are answered
from the copy; Zulip is asked for what the copy cannot say (a topic in an
archived channel, once), for the check right before a write or a
notification, and for the writes themselves. A restart within Zulip's queue
lifetime resumes the queue and reads nothing.

Listening is `agag.listen`: an intake thread follows the mirror's change
feed into a durable queue beside the store (`listener.sqlite`), one executor
serves the queue, and neither waits for the other — a long run no longer
stops the polling, a burst on one conversation is one serving, and downtime
is recovered from the index rather than by sweeping every channel. The
served note and the root note keep exactly the meaning they had; the
listener's recovery reads them off the index.

Rate limits are honoured **per credential** at the transport
(`agag.zulip.Budget`): a 429 to any client on a credential pauses every
client on it, in the process and — through `<credentials file>.ratelimit`
beside the env file — in every process built from that file. Tools outside
`agag.zulip` (a browser, `curl`) are outside that.

The ComfyUI notifier's command intake is an event queue too; the
`is:mentioned` narrow it used to poll every five seconds (78 % of the
realm's API traffic on 2026-09-14) is read once at start and after a queue
expiry. The whole measurement is `devdocs/episodes/better_zulip_call/`.

## cagent (pj-clusterintent)

- Responds to requests for explaining/observing/changing desired state and/or actual state of the cluster.
- In-System workflow shold be designed so that cagent receive a report when cagent's explanation of the cluster found invalid.
- That report path exists since `devdocs/episodes/better_communication/zulip_cagent_receive`:
  a DM to the **Cagent bot** in Zulip, or `POST /window` on cagent's
  unauthenticated window, records the report as a local file and does not try
  to repair it in that turn. Ask the window "what has been reported lately"
  to read them back.
- Three doors today — node (mTLS), human (bearer token), window
  (unauthenticated). The window is where they are heading; see the
  cross-project `devdocs/todo_done.md` entry.
- **Since `refactor` p3 cagent has a channel of its own and its change
  record is the conversation in it.** A request to change the cluster used
  to become a Plane issue in `ClusterAdmin`; it is now a
  `change-<stem>-o<the id of the asking post>` topic in cagent's instance
  channel, holding the statement of the change as an ordinary post and
  `[selfnote][change]`, `[origin]`, `[doc]` and `[state]` for what a program
  must resolve without guessing. **Identity is a message id** — the
  `[change]` note's own id *is* the request (`c5867`) — so a rename, a
  resolve or a reused topic name never redirects one request's record into
  another's, and a deleted anchor is *absent*. The origin is told where its
  record is with `[selfnote][changerec] <id>`, which is a selfnote and
  therefore buys nobody a run: registering the same request again restates
  it in the same conversation instead of forking it.
- Its listener serves every topic in that channel and the `cagent-`/`change-`
  prefixes in any public channel (through `agag.listen` and a mirror on its
  own credential since `better_zulip_call` p1), and cagent posts its own
  `intro-` contract in `#agents` like every other agent (`uv run --project
  cagent python -m cagent_api.intro`).
- **Recording a change is not making one.** That distinction is why the
  front writes a file rather than calling `nctl`, and p3 relocated the
  record without adding reconciliation to it.

**Plane is gone from this system entirely (`refactor` p3, 2026-09-10).** It
was the external task manager three agents kept their work records in; p1
moved autolab's records into its conversations, p2 forge's, p3 cagent's, and
then removed the client (`agag.plane`), the Nautobot agent identity fields,
the `tasks / plane` screen, the provisioning credentials and the deployment
itself. Nothing in the realm reaches a second system for a work record any
more: **a conversation is the record**, and a message id is its name.

## autolab agent(pj-agdev/agautolab)

- Responds to request for explaining/observing/developing projects.
- Its entrance is the Zulip channel named after its instance — questions
  only, nothing is started there. Development work goes in a `workplan-…`
  topic in the project's own `pj-<slug>` channel, because the channel is what
  says which project the work is for. Its own introduction in `#agents` is the
  authority on both (`agent_standardize` p4).
- Since p9 a `workplan-` topic's tasks each get a `workrun-` topic that
  **says what it is for**: a `[rootchat]` note naming the mission
  conversation and — since `refactor` p1, where the Plane Sub-Work it used to
  name went away — a `[task]` note naming the mission by the message id that
  *is* it, written before the visible description. Nothing is read from the topic's name or its
  channel's description any more.
- **autolab progresses an authorized mission itself** (`robust_workflow`
  p1). "You may start" starts task 1, and the requester's acceptance of task
  N starts task N+1 — autolab posts one visible line naming nobody and a
  `[selfnote][start]` (pyagag `agag.selfnote.start_note`), which is the one
  note that makes its own listener serve a conversation only it has spoken
  in. The per-task start post Front used to relay is gone, and with it the
  stall class behind adventure_game p3's 24 minutes. `hold.flag` (the
  requester asking the next task to wait) marks it `held`; a post there
  starts it. A task's report reaches the requester through the parent
  conversation's root note (`agag.zulip.parent_rootchat`) even though the
  requester never posted in that task.
- **A task closes only on its requester's agreement, and the mission's
  close is the requester's record** (`robust_workflow` p3). The close-out
  runs only when the serving's processed input holds a requester's post —
  autolab's own start note never counts (p3 trial G closed a task nobody had
  accepted). When the last task closes, the mission is done once its
  requester **records** the acceptance: `agentchat accept <mission id>
  --evidence <the post that accepted it>` (pyagag `agag.acceptance`) writes
  `[state] accepted` on each finished task, `[selfnote][acceptance] #<post>
  by <user>`, `[state] done`, and ✔ — selfnotes only, so nobody runs to
  acknowledge it, and a repeat or an interrupted attempt ends on the same one
  record. It refuses, writing nothing, while a task is open, without
  evidence, or on evidence older than the mission or written by the recorder
  or the owner. A person without the tool says it in the `workplan-` topic
  (`accept.flag`, the same operation); `mission_done` and the operation
  room's completion door write the same record. Posting the acceptance into
  the plan for autolab to "note" — two paid runs and no record — is what
  this replaced.
- **Each mission works in its own copy of the project** (`robust_workflow`
  p3 ex1, `agautolab.missionspace`). The copy is
  `.local/missions/<slug>/m<id>/`: one git worktree per repository of the
  project folder, on branch `autolab/m<id>`, found again by the mission's
  id. Its pushes are refused. Task runs work there, and planning stays in
  the project folder, which holds integrated work only. What a planning run
  writes there is committed as its notes.
  - A worker's commit is a **checkpoint**. After each serving that changed
    the copy, `[selfnote][change] checkpoint` records its exact content.
  - The requester's agreement closes a task: everything in the copy is
    committed and bound to exact commits (`[change] accepted <repo>=<commit>
    #<post>`) before anything shared moves.
  - Then comes integration under a per-project lock. A shared branch that
    has not moved is fast-forwarded. One that moved is merged when the two
    sides touched different files. The same file on both sides, conflicting
    or not, is `returned` and the task stays open: the worker merges the
    shared branch into its copy and the requester reviews the combined
    result.
  - `main`, `direction` and `devlog` are published by the push that
    integrates them, a compare-and-swap that is never forced. Other
    repositories are published only when the worker names them in
    `publish.flag`.
  - Only then is the task `completed`, so a mission's acceptance never
    leaves integration outstanding.
  - A repeat is recognized by ancestry. A close-out cut short is finished
    from its `accepted` note without another run.
  - Cancellation, replacement, the last close-out and archiving release the
    copy: uncommitted work is committed to its branch, the worktrees are
    removed and the branch is kept. Unattributable dirt found in a project
    folder at rollout went onto `autolab/set-aside-20260924`.

**A project channel files itself (2026-09-04).** A `pj-<slug>` channel is
still opened by a human, but every serving of it now files it in the
channel folder of its own name, minting the folder if needed, using the
provisioner credential the node holds. One folder per project is the
standard; `work-` channels inherit the parent's folder, which is why a
project channel filed wrongly by hand used to carry its whole fleet with it
(`devdocs/episodes/agautolab/channel_folder/report.md`).

**Since `agent_standardize` p10 an instance's own channel answers about its
work.** Ask forge or autolab in its own channel where its plans stand and it
surveys its board — its own topics for forge, every `pj-` channel and their
derived `work-` channels for autolab — and says so. Told to close out what is
finished, it verifies by reading, resolves those topics, and (autolab) marks
the mission Work Done with `python -m agautolab.mission_done`. **It never
tidies on its own**: that is contract, not shackle. Each serving keeps a
streamed transcript, because an answer that skipped a project reads exactly
like one that found nothing in it — which is how a whole project went
unreported for one round of p10.

## Opsroom Observer (not an agent)

`Opsroom Observer` is a Zulip bot with no listener, no runs and no
conversation: it is the credential the agdevworld relay's **mirror** reads
the realm with (`agdevworld/agentroom`, `operation_room` p2; since
`better_zulip_call` p1 every read the relay makes — the agent room, the ops
board, the routines, the Front Desk, every completion preview — comes from
that one persisted copy, and the Developer's credential only writes). It
exists so the relay's reading never comes out of the quota the agents' own
listeners or the Developer spend.
It is **read-only by operation** — Zulip has no read-only API key — and its one
write to the realm is subscribing to public channels, which an event queue
requires and a read does not. It posts **only** in `#ops-testbed`, which no
agent is subscribed to, and never names a real agent bot even there: a mention
in a public channel reaches a bot that is not in the room, which is the
`zulip_command` lesson that cost one paid Front run per lap.

Since `operation_room` p2 ex2 the relay it reads with is **always on** —
launchd `com.agdev.agentroom` on agstudio. `/ops` is a running reconstruction
rather than a per-request read, so a relay somebody has to remember to start is
a board that is not there when it matters.

**One direct message is the exception** (`robust_workflow` p2): the relay's
watchdog over Observer's request monitor DMs the realm's owners — humans, no
agent named, so it buys no run — when the monitor goes into a failing state
and when it comes back. `#ops-testbed` has no human owner in it, so a post
there would have reached nobody.

**The observer still never posts.** Since `operation_room` p3 the same relay
*can* post — `POST /chat`, into a routine's own `guide` or run topics, and
a run request into a Front Desk conversation — but on a third
credential of its own (the Developer's) with no fallback in either direction. A
relay without it reads routines and cannot answer in them, and says so on the
screen rather than at submit time.

## Routines (`refine_routine` p1)

A routine is a **process guide kept in Zulip** and the runs made of it.
The `routine` channel folder holds one public channel per routine,
`#routine-<name>`; its fixed `guide` topic is the guide, and **the newest
post there is the whole guide** — a new version is a new full post, never an
edit, nobody runs or reports there, and a ✔ on it retires the routine. There
is no schedule and no dispatcher any more: a routine runs when somebody asks
Front to run it, with whatever conditions they give in words ("until the
5-hour window is 50 % used", "one repository only").

A **run** is `#routine-<name>` › `routinerun-<id>`, opened by Front with one
opening post — the request as it was made, the conditions as Front read
them, the guide post it read (its message id), and the requesting
conversation — and then owned by Front: the `routine_run` role serves it
with its own guide, delegates from it (root notes in the delegates' topics
name the run, so their answers resume the run and nothing else), records
each serving as an entry, and ends it with one fenced `ag-routinerun` block
(`achieved`, `reason`, `report`). Front's listener delivers the report into
the conversation that asked, names the run, and resolves the run topic.
**Ending a run and reaching the routine's goal are two sentences**:
`achieved` is the goal, `reason` is why the run ended.

Three things the realm taught about starting and stopping:

- **A topic Front opens alone is never served by the owner route** (last
  speaker is Front), so the listener starts a run right after the serving
  that opened it, and at startup for one opened just before a crash. A run
  is *unstarted* while it holds only Front's speech and no serving ack; an
  ack is what says "started", so a run never starts twice and a progress
  entry never restarts anything. A run opened by hand is served like any
  other topic.
- **The root note in a run topic means "opened from"**, never "serve that
  instead": an owned topic is always served as itself. The report is posted
  into the origin directly by the listener, so no root note ever points from
  the requester to the run.
- **The report is Front's own speech, so delivering it silences the requester
  too** — and a request made of more than one routine then had nobody left to
  notice (`routine_tests` p1). The delivery is followed by
  `[selfnote][delivered] <run>` in the requester's conversation; a note buys
  nobody a run, but it makes "a report landed here and nothing has served this
  conversation since" a question the chat can answer, which
  `continue_deliveries` asks after every run serving, on both routes, at
  startup and after every recovery the listener makes. The serving it buys is
  ordinary, and
  what happens next — open the next run, tell the developer, stop because the
  work failed — is decided by Front in the conversation; no stage is encoded
  in the listener. An ordinary one-routine request does not loop, because
  Front's reply is speech and speech spends the note. An **ack** is not speech
  enough, so a crash between a serving's ack and its reply leaves the handoff
  owed rather than silently spent.
- **A finished run is never reopened, and a late answer is not dropped.**
  Ending a run resolves its topic, and resolving *renames* it, so the callback
  route used to read the bare name the root note recorded, find nothing, and
  post — forking a twin without the origin note, whose report could reach
  nobody. Now a resolved home is left alone; when it is a run, its origin is
  told an answer arrived after the end and gets the delivered note, so the
  conversation that can decide about it is served. Both ends closed means
  there is nobody to tell, and the log says so.
- **The `ag-routinerun` block ends the run — it is not a progress note.**
  Three runs in one trial wrote one on their first serving, seconds after
  delegating, each contradicting itself ("so the run continues", "the run has
  not ended"), and the listener did what the block says: delivered and
  resolved, with the work still in flight. `achieved: false` means *this run
  is stopping without its goal*, never "not done yet". The `routine_run` guide
  now separates a serving ending from the run ending and gives the test —
  is there anybody I am waiting for? — and a re-measurement found no
  premature block at all.
- **Usage conditions are judged on an observation**, `tools/budget.md` at
  every run serving and `agbudget` on the run's PATH (the relay's `/budget`,
  or a fixture file via `AGFRONT_BUDGET_URL`). "Until N % used" is the
  current window reaching N whatever consumed it, met at once if already
  there; a failed or stale read is neither reached nor 0.

On the screen (`agdevworld` operation dashboard and routines view) a routine
is its guide and its runs; "Ask Front to run it" posts the request at Front's
ordinary entrance (a Front Desk conversation) and the run appears when Front
opens it. Completing a run closes that run and its work only — never the
guide, the channel or another run; the requester conversation is its parent
and stays open.

## observer (`pj-agdev/agobserver`)

- **Waits, so nobody else has to.** One watch is one topic in its own
  `agobserver-agstudio1` channel: a condition in ordinary words, what to look
  at, and where to notify. Observer looks every minute with the **host's own
  local model**, posts once into the conversation the requester named when the
  condition holds, and then stops. Waiting therefore costs no account at all —
  which is the point, and why the polling role is deliberately **not** on its
  execution menu.
- **Its introduction is the contract**, like every agent's. `watch-<something
  short>` is the recommended topic name; every topic in the channel is a watch
  and there is no second door. **A ✔ on the watch topic cancels it** — that is
  the whole cancellation gesture, checked before each evaluation and again
  before notifying, because the ✔ may land inside the evaluation meant to be
  cancelled. Since `p1 ex1` that check is a lookup of the watch's **own anchor
  id**, not a comparison of remembered topic names: a renamed watch is
  continued under its new name, a renamed *then* resolved one cancels, a
  deleted anchor ends it locally, and a lookup that got no answer concludes
  nothing at all.
- **A watch is a message id.** `[selfnote][watch]`'s own id *is* the request
  (`w6676`), with `[accepted]` (condition, target, destination, requester) and
  `[state]` beside it, and one ordinary visible post saying the same in words.
  The same rule the rest of the realm arrived at the hard way: a name is
  reusable, an id is not.
- **The acknowledgement names nobody**, and that is load-bearing rather than
  polite. The requester is waiting for the *notification*; naming them in the
  acceptance would buy them a paid run to read "understood". Measured live on
  2026-09-13: Front opened a watch, Observer accepted it, and Front's listener
  did not stir until the notification arrived.
- **The answer is three-valued.** `met`, `not_met`, and **`unable`** — a
  statement about the *look*, not about the world. A target that cannot be read
  is never reported as a condition that has not held yet; the watch keeps its
  schedule and says so in its own topic once, at the third consecutive failure.
  Routine polling is otherwise silent.
- **A destination stops being a name at intake.** Since `p1 ex1` both
  spellings — a message link and `<channel>/<topic>` — are resolved *while the
  request is being accepted* to a message id in the conversation the requester
  meant, and that id is what the Zulip record carries, so it survives a lost
  local store. Delivery follows the id: a rename, a ✔ or somebody taking the
  freed name cannot move the notification. A destination that is gone or
  already ✔ is a terminal `undeliverable` outcome recorded in the watch topic —
  **Observer never opens a conversation of its own to deliver into**, and
  leaves that watch topic open because a human has to see it. A delivered watch
  is resolved.
- **Uncertainty is never terminal, and that took a change in pyagag.**
  `ZulipClient.message()` flattened every failure into "absent", so a timeout
  reading a destination was indistinguishable from its deletion. `call()` now
  raises `ZulipRejected` for an answered 4xx and a plain `ZulipError` for a
  5xx, a timeout or a dropped connection, and `message()`, `conversation_of()`
  and `topic_history_across_resolve()` take `strict=True` (pyagag `f613feb`;
  the lenient default is unchanged). Observer stops only on Zulip's own "no":
  a failed destination read, a failed read-back and a failed send all keep the
  met result and the pending notification and retry at the ordinary interval,
  without re-judging the condition.
- Exactly-once rests on three things in order: the store's delivery record
  (which is why polling never reaches delivery twice), the watch id inside the
  notification (which a read-back recognizes after an ambiguous send), and
  writing the state note *after* delivery, so a crash between them leaves the
  watch owed rather than silently finished.
- The listener's own triggers cannot wait — the listener reacts to posts
  and its recovery runs at startup and after a resync — so the due-watch
  trigger is a worker
  thread started beside the listener, not a plist setting. See
  `devdocs/episodes/observer/p1/`.
- **The introduction says what the requester does while it waits** (`p2`
  step 1): finish the serving once the request is posted, name the
  conversation that serves *you* as the destination, and leave the job's
  identity, its output location and the next action behind, because the run
  the notification wakes is a new run that remembers nothing. A job
  condition is written as **"it has ended"**, with what failure looks like,
  since a condition only success can meet waits forever on a crash. Only
  what is reachable from Observer's host can be looked at — a path on
  another machine is not, and an HTTP endpoint usually is.
- **One watch per thing you are waiting for.** A condition covering two jobs
  was judged `met` while its own evidence said one of them was still
  running (`p2` step 3, `w6866`): correct by luck of timing, and a slower
  job would have woken the requester to collect a result that did not exist.
  The `observe` guide now says every part must hold in the same look, that
  the parts are enumerated in the evidence, and that a verdict must agree
  with its own evidence — a "but" in the evidence has already answered the
  question. Two later two-job watches judged correctly across seven looks,
  but a conjunction is still the hardest thing a small model is asked to do
  here.
- **An evaluation's cost follows the condition, not just the queue.** p1
  measured 11–22 s on single-target conditions; a two-job condition reading
  two histories and a queue took 30–120 s. Sequential evaluation is the
  ceiling, so a complex condition makes every *other* watch wait too.

- **It also watches requests nobody registered** (`robust_workflow` p1,
  `agobserver.monitor`; p2 made it durable and evidence-backed). Every couple
  of minutes it traces, off its mirror (no Zulip call), every request that
  came in through `#front` recently **and every request it is already
  tracking**, and lists in code what is owed and overdue: a task with no
  start after its predecessor finished, a post nobody's listener
  acknowledged, an answer the asker was never served, a failure notice, a ✔
  on live work, the request's own conversation ✔'d with work still open
  (`origin_closed`), a long silence. The two judged kinds (a ✔, a silence)
  go to the `triage` role on the local model — on a worker of its own, so a
  30–130 s judgment never holds the look at everything else.
  - **Identity is ids.** A request is its origin's first post (`o<id>`); an
    incident is (request, stalled conversation's anchor), the kind an
    attribute — a rename, a ✔, a restart or a different blockage of the same
    work is the same incident with the same two requests; a reused name is
    another request. A lost store adopts the incident from its
    `[selfnote][incident]` note.
  - **Retention is not discovery.** A request stays tracked
    (`.local/incidents/tracked.json`) while anything opened for it is
    unfinished or an incident of it is open, whatever its age or name.
    Unfinished means not `done`/`cancelled` **by record** (p3): a ✔, a quiet
    wait and a `legit` verdict end nothing. A dismissed incident (a wait
    judged legitimate) closes as *finished* when the work records its outcome,
    *cancelled* on a recorded decision, and stays dismissed otherwise.
  - **A verdict is about the evidence it read** (p3). Each judgment carries a
    snapshot identity (kind, state, ✔, the newest relevant post in the
    stalled conversation and in the request's own); a verdict whose snapshot
    no longer matches is discarded and the current state judged, and a kept
    `stall` is judged again when its evidence moves — the requests already
    made still count. The health record says `pending`, the oldest wait,
    `invalidated` and `churning`; the watchdog reads a backlog or churn as
    `degraded`. The judgment reads the request's own conversation too (its
    newest spoken posts) and a long post by both ends; every judgment keeps
    its exact `prompt.md` and `input.json` beside its transcript.
  - **Recovery is a transition on record, never an absence.** *Rescued*
    needs a fresh look that reads the stalled conversation in the state its
    kind waited for; a recorded `cancelled`/`replaced` closes it *cancelled*;
    an unreadable conversation is said once and *reported* after 30 min; a
    stale mirror concludes nothing and asks nobody. An `undelivered` ask
    carries `[selfnote][owed] <remote> <answer id>`, and the requester's
    listener turns it into the served mark once the serving that read the ask
    has replied — the receipt the next look verifies.
  - Asked about **in the conversation the request came from** — Front owns
    it — at most twice, ten minutes apart; with nobody to ask (origin ✔ or
    gone, its own agent silent) or no progress, reported to the realm's
    owners by name. `python -m agobserver.withdraw <why> <incident>…` is the
    operator's correction for an incident opened in error.
  - **The monitor never reports its own failure.** It writes
    `agobserver/.local/monitor-health.json` at every look and judgment; the
    agdevworld relay evaluates it every 30 s (`ok`, `idle`, `disabled`,
    `missing`, `stopped` — saying whether Observer's process is alive —,
    `stalled`, `judgment_stalled`, `unable_to_observe`, `degraded`), shows it
    on `/ops`, the ops board and `/healthz`, and DMs the realm's owners on a
    change into or out of a failing state.
  - Trial fault hooks, created only by a person, in
    `agobserver/.local/faults/`: `monitor-stop`, `triage-stall`,
    `mirror-stale`.
  The bot keeps itself subscribed to every public channel, because a ✔ in a
  channel it has not joined never reaches its mirror.

## Request progress, operation failures and resolving (`robust_workflow` p1–p3, 2026-09-24)

- `agentchat trace [<message id>]` (pyagag `agag.trace`) follows a request
  from any message through every conversation opened for it — the topics
  whose root note names it, whoever wrote the note — and gives each one
  state from its posts: `not_started`, `queued`, `executing`,
  `awaiting_requester`, `awaiting_delivery`, `awaiting_human`, `answered`,
  `failed`, `done`, `cancelled`, `unobservable`, plus "owed now". Since
  `clearer_chat_ui` (2026-09-25) `awaiting_human` means an **explicit**
  response request is pending in a person's conversation; the agent having
  merely spoken last there is `answered` (see *What a post is for*). Without an id it
  traces the conversation the run is serving. An answer is `awaiting_delivery`
  until the requester's **served mark** covers it, whatever the owner's own
  record says and whatever the requester said since (p2: speech at home is no
  receipt). A task its owner started itself is owed to the requester of the
  conversation it was opened for (the parent hop callbacks follow). A root
  note means the conversation its `#anchor` is in; without one, never a
  conversation that began after the note was written, and through
  `[replaces]` to a retired predecessor. Every node carries a stable
  `anchor`. `MirrorReader` answers the same reads from a mirror at no call.
- A refused or uncertain `agentchat` write leaves `[selfnote][opfail]` in the
  run's home conversation, so the failure is one read away from the request
  instead of only in a transcript.
- `agentchat resolve` refuses while the caller's own newest post there is
  unanswered (a resolve renames; it stops nothing) — `--anyway` for a
  conversation known to be finished. `agentchat unresolve` undoes a ✔,
  folding back a stray post made under the old name and refusing only a
  conversation of its own. The `send` refusal names `unresolve` instead of
  telling the caller to open a new topic, which is what forked p3's twin.
- A listener serves a mention in **somebody else's** ✔'d conversation: a
  task's closing report is followed at once by its ✔, and until this the
  report reached the requester only if the listener looked before the rename
  arrived. Since p2 the mention route judges the post that triggered it **by
  id** first, so a ✔ that moves the topic between two reads cannot hide it.
- **A receipt follows what a serving was given, on every route** (pyagag,
  p3). Each thread handed to a run is recorded in its serving journal
  (`agag.serving.note_input`: the span of ids, and whether the read reached
  the beginning); after the reply is confirmed delivered, the newest post
  naming the agent inside each span gets its served mark — owner route and
  mention route alike, so an answer an owner serving relayed is not served
  again for "nothing new". An answer that arrives after the thread was read
  stays owed; a restart between delivery and receipt writes the receipt
  without a rerun. A task its owner started for the agent (the parent hop)
  is one of home's threads. Startup recovery also finds a callback in
  somebody else's ✔'d conversation, newer than the listener's **horizon**
  (the newest message its mirror held when this rule first ran), so the
  realm's older ✔ history is not replayed.
- **Served marks are matched by the post they name** (pyagag
  `agag.identity`): `[served] <remote> <id>` covers the conversation post
  `<id>` is in now, so a renamed callback topic is not served again after a
  restart. A callback serving is anchored in **home** (`AGENTCHAT_HOME_ANCHOR`
  and the reply's destination are a post in home, not the post that named
  the agent elsewhere), and a prepared reply redelivered after a restart is
  re-located by its anchor.

## How a run finds all of this

`agentchat intro` lists every agent on the `#agents` board with its own
one-line pitch, and `agentchat intro <agent>` prints one introduction
verbatim as it is posted now (pyagag `1691328`). A serving already gets the
whole board snapshotted into `tools/agents.md`; this is the same content
read live, and the entry point for a run that only wants one contract.

`agentchat --help` also tells a run what to do about anything slow — a
download, a long job, somebody's answer — namely that an agent on the board
may take that on, and that holding a run open to watch spends the run on
nothing. It deliberately **names no agent**: which agent waits is what the
board says, not what the tool says, the same rule that keeps routing
vocabulary out of every consumer's guide.

## agfront(pj-agdev/agfront)

- Responds to any requests from Human and sends messages to other agents.
- Since `argue` p2 it also **renders**: a presentation role and a job worker
  of its own turn recorded speech into character dialogue in memo topics
  (see *Memos* below). No new account: the memo posts are Front's.
- It can also be asked to **supervise**: stay with a request until the agent
  doing the work finishes, answering what it asks along the way. Since
  `agent_standardize` p7 that is not a long run but several short ones — Front
  posts, ends, and is called again when the answer names it.
- **Its reply always goes to the developer.** Since p8 a run called back from
  another agent's topic still answers in its own `front-*` conversation;
  anything it says to that agent is a deliberate `agentchat send`. That is
  what ends an exchange between two agents, and it is the whole of p8's
  answer to p7's "nothing decides when a conversation is over".
- **Since `refine_routine` p1 it runs routines**: a `routinerun-` topic in
  a routine's channel is a conversation of Front's own, served by the
  `routine_run` role (see *Routines* above).

## archsage — the knowledge council (`archsage`; `argue` p1, 2026-09-16)

**One deployed agent, one Zulip account (`archsage`, user 24), many logical
sages.** archsage runs on the frontier profile (Claude Fable 5.1 through
claude_code) and designs the knowledge and research a desire needs: which
existing knowledge bears on it, which research questions would take it
further, which domain nobody covers — in which case it defines a new sage
(`archsage sage add`). A **sage** is a directory `sages/<name>/` holding
`sage.toml`, a domain `guide.md` and the clone of one study's knowledge
repository (`mainstudy/`, usually the study's internal `main`, refreshed with
`archsage sage sync`); it runs
on the shared `sage` role (Sonnet 5) with **one tool**, `sagetree`, a
bounded reader that refuses every path outside its tree and writes only
into the sage's study queue (`tostudy/`). An empty tree is a valid state
for a newly planned study: the sage says so instead of inventing findings.

Addressing is a **selector**: `@**archsage** sage:arxiv …` in an argue, or
`sage:arxiv …` as the first word of a post in `#archsage-agstudio1`; a bare
mention or post asks the council. Every sage reply begins with
`**[sage:<name>]**`, added by the posting layer. A request for a sage costs
no archsage run; a name nobody publishes is refused in one line. The
introduction lists the sages, rendered from the directory at post time.
Dispatch between roles never goes through Zulip (a listener ignores its own
posts): archsage consults a sage from inside its run with `archsage ask`.

**The boundary is a bounded reader, not a sandbox.** Under claude_code a
`Bash(sagetree:*)` grant admits a compound command; the transcript shows
an escape as a command that is not `sagetree`. archsage's broader access
is explicit in its `agents.toml`.

**arxivsage is retired** (`argue` p1 step 4): its listener, plist and
Nautobot rows are gone and its `intro-` topic is ✔; `sages/arxiv` is its
knowledge (`study-arxiv-trend`) in the new structure, with the same study
queue contract (`sage` p1). The channel and repository stay as history.

### archsage establishes studies (`sage` p2, 2026-09-26)

**A study is archsage's to establish; running it is Front's.** A request to
create a study, to connect an existing study to a sage or a routine, or to
give a sage a study goes to archsage in a topic of its own channel
(`study-<slug>`, opened by the asker with `agentchat send`, so the asker's
root note is the return path). archsage decides the substance — the research
plan, the sage's domain, the routine guide — and the tools make it:

- **`agproject`** (pyagag `agag.project`, was agfront's) opens or
  **continues** a `pj-<slug>` channel: members resolved explicitly (owners,
  the routine runner and workspace agent from the board, the caller, the
  board reader `Opsroom Observer`), the plan, and a setup request carrying a
  fenced **`ag-setup`** block (`ag.project-setup.v1`). A repeat does only
  what is missing. `agproject status` reads `absent … setup-pending /
  answered / ready`; a structured setup is `ready` only by autolab's answer
  line `study layout established: main/ = <repo> at <commit>` (an ack is not
  an answer). Channels are created with `AGAG_PROVISIONER_ENV` (archsage's
  comes from its ignored `.local/listener.env`).
- **autolab lays a study out from the block** before `init_project` could
  scaffold a plain project: `README_PROJECT.md` generated from the block
  (`autolab project establish <slug>` rebuilds it), `main/` = `autodev/<slug>`
  with the plan, `README.md`, `methods/`, `reports/INDEX.md`. No
  `direction/`/`devlog/`, no `publish/`.
- **`agroutine create|update|show|list`** (pyagag `agag.routine`) registers
  `#routine-<name>` in the `routine` folder with the exact description and
  posts each guide version as the caller, **read back** (Zulip truncates at
  10,000 characters silently). Registering starts nothing. The board and
  running listeners see a new channel without a restart.
- **Sages** (`archsage sage add|update|attach|sync|remove|show`,
  `archsage queue …`, `archsage intro`): a sage records `project`, `source`
  (`main` — the internal repository, the default — or `publish`) and the
  repository. `attach` checks the repository, syncs at once and reports the
  revision and whether the study has findings; `sync` replaces a tree cloned
  from another repository (only after the new clone worked). Definitions
  are runtime state in a private store, `sages/` = `autodev/archsage-sages`
  (ignored in the public repo; `archsage store restore` rebuilds it); every
  change is pushed and the introduction re-posted. A queued question is
  removed only when files in the refreshed tree answer it.
- **archsage is called back where it delegated**: a mention carrying its
  root note serves the home conversation (a ✔'d home under its ✔ name) with
  the answering topic as a thread. Own-channel replies name the asker; a
  reply declaring `intent=progress` names nobody
  (`TopicResult.quiet_progress`), so waiting on autolab buys no run.

Reports keep three states apart: **setup complete**, **research complete**
(a routine run accepted and integrated), **knowledge refreshed** (the sage
synced to that revision). Research with the setup means Front runs the
study's routine afterwards, and the guide archsage writes ends with asking
archsage to refresh the sage. `devdocs/episodes/sage/p2/`.

## Argues (`argue` p1, 2026-09-16)

An **argue** is `#argue › argue-<stem>`: a conversation in which a human
develops a desire — vague and far-reaching at first — with every agent.
Front opens one from an ordinary conversation (`agentchat argue open`) and
is the only agent served automatically there, with the `argue` role and
**no hand-off mention**; every other agent takes part only when **named**,
and answers in the same topic. The contract is `pyagag` `agag.argue`, so
every participant means the same thing by it:

- **A mention is an invitation** that costs a run; nobody names the last
  speaker by reflex. `@**<bot>** <kind>:<name>` addresses one logical
  speaker of an account. An invitation is outstanding until a reply under
  the matching speaker header answers it — judged from the conversation, so
  several invitations in one post, other posts in between and a restart all
  get the same answer. The participant writes `[selfnote][served]` into the
  argue topic afterwards, and `agag.listen` judges every mention route by
  the newest unanswered mention above that mark, not by the last post.
- **The human's desire** is `[selfnote][desire] <message id> by <user id>`,
  written by Front's listener only for a human's own post in the argue —
  Front's draft is not the submission, a human's "yes, that is it" is.
  While it is missing Front asks when the human speaks; nothing polls.
- **The argue's identity** is the id of its `[selfnote][argue] from
  <channel>/<topic>` note. Resolving the topic ends discussion dispatch and
  nothing else.
- **Participants**: cagent, Observer, autolab, forge and archsage each have
  a mention route that answers an argue invitation and ignores every other
  mention, a read-only `argue` role, and `agent/guides/argue/role.md`. Observer's
  is its first mention route; a `front-` topic is not an argue, so Front
  thanking it still buys no run.
- **How it ends** is Front's judgement, from the conversation and the
  advice — study first (a new study, or a research plan in an existing one)
  when exploring would widen or firm up the idea, else a project. `agproject
  open <slug> --kind project --doc …` creates the `pj-<slug>` channel
  (humans, autolab, Front; its own folder), posts the goal, and asks
  autolab in `workplan-setup-<slug>` to prepare the workspace — setup only,
  no `workrun-`, no routine; **a new study is asked of archsage** in its
  own channel since `sage` p2 (it answers the argue when the study, its
  routine and its sage exist); `agproject plan <study> --doc …`
  posts one plan into an existing study. Front's outcome reply ends in an
  `ag-argue` block (`outcome`, `target`, `complete: true`) that the listener
  **checks against the realm** — channel, document, autolab's answer —
  before it writes `[selfnote][outcome]`, tells the origin conversation and
  resolves the argue. Running the study or developing the project is the
  next chapter and belongs to whoever the outcome names.

## Memos, and how a discussion is shown (`argue` p2, 2026-09-18)

**A memo is a conversation that is read and never answered.** A conversation
whose channel is `memo` (or `memo-…`) is presentation only (pyagag
`agag.memo`): nothing written there starts anything — not a post in an
owned-looking topic, not a real `@**bot**` mention, not a notifier command,
not a copied `[selfnote]`, not a rename or a resolve, not a restart with work
already queued. The channel is the distinction because it is decidable
before the first message, survives every rename, and costs no read; a `✔`
could not be, since the ComfyUI notifier accepts commands under `✔` on
purpose. The one rule is asked at `agag.listen`'s intake, mention route,
recovery and again at execution time, in `serve_topic`, in the mirror's note
index and `own_notes` / `mentions`, by `agentchat send` (no root note there),
by the notifier's two intake paths and by the relay's ops board. **A consumer
with an intake of its own must ask it too.** A memo names what it presents
with `[selfnote][memosource] <message id>` — never `rootchat`, which takes
part in callback routing, `threads/` and completion discovery.

**The discussion carries no character; the character dialogue is a
rendering of it.** Until p2 the Front Desk run (`character_talk`) was handed
every character's lore and wrote an `ag-dialogue` block into its own reply,
so lore sat in the context of every judgement and delegation. Now:

- Front's discussion roles — `front`, `desk` (the Front Desk), `argue`,
  `routine_run` — get no settings and no dialogue contract; a reply is posted
  as written. Human input is verbatim, under the human's own account.
- Front's **presentation** role (`present`; `agfront.present`) re-voices
  recorded speech: its input is a snapshot (the posts, the context before
  them, who each speaker is, one pinned settings revision), its output one
  validated block, and it has no `agentchat`, no shell and no way to write
  into a discussion. `archsage` and `sage:<name>` are different speakers on
  one account; a speaker without a character is a `plain` turn, shown as
  written under its own label.
- `agfront.render` is a worker beside Front's listener, on the same mirror,
  with its own checkpoint and store (`.local/render/`): triggered by new
  agent speech in a Front Desk conversation or an argue, coalesced after a
  quiet period, one durable job per content (anchor + message ids + content
  fingerprint + settings revision + renderer version). The result is an
  `ag-memo` record in `#memo › <source topic>-s<anchor>`, and **the memo is
  the truth**: a job whose record is already there is finished without a
  run, which is how a crash between the post and the local "done" heals.
  Three attempts, then one `failed` record; nothing re-arms it but a request.
  The first start renders nothing that already exists.
- **Another interpretation is explicit**: `[selfnote][render] <settings
  revision>` written into the *source* by a human account (the relay writes
  it as the Developer). Earlier interpretations stay, with the revision their
  portraits are retained under. An edited source is visible as `stale` by
  fingerprint and is re-rendered only on request.

**The Arguing Room** (`agdevworld` `/?view=argue`) is where a human starts,
reads and continues an argue; the Front Desk is the same scene with another
adapter. The relay's `/argues` routes name an argue by its anchor message id
and locate it where that message is now; every read — source, memo results,
status — is the mirror's. The composer always posts into the source
conversation, whichever view (dialogue or original) is showing, and a post
with no rendering yet is shown as written, so nobody waits for a rendering
to read or reply. Resuming a ✔'d argue resumes the discussion and nothing
downstream.

**The Project Room** (`agdevworld` `/?view=project`, `project_room` p1,
2026-09-19) is where a human follows a project or a study from its purpose
through its plans and runs and talks in the selected one. The relay reads
it all off the mirror (`/projects`): the channel is the project, its kind is
read from the description or stays `unknown`; `goal` / `researchplan-`
topics are documents (not executable work, never assigned a mission by
name), `workplan-setup-` is setup that plans no mission, a `[mission]` note
makes a mission and a `[task]` note a task, by anchor id. Two states are
shown side by side — autolab's recorded `[state]` word and the ops engine's
reply verdict — with task counts and an explicit *incomplete* when a work
channel is not mirrored. A comment goes into the mission's planning
conversation or the task's execution conversation, located from the anchor
at send time; a document is refused with the path to Front; a ✔'d target is
resumed only on a second, explicit press.

## forge agent(pj-agdev/agforge)

- Responds to requests for providing media assets with characteristics specified in the requests.
- Its entrance is the Zulip channel named after its instance; an asset
  request is an `assetplan-…` topic there (`agent_standardize` p1).
- Since p8 **forge opens the run topic itself** when it registers the plan,
  and says so in the plan topic. The requester posts there to start it, and
  what they post is read. Nothing is chosen from a queue any more.
- Since 2026-09-01 **a long generation no longer holds the run open.**
  `agforge video submit` / `music submit` queue a ComfyUI job and return its
  `prompt_id`; the generator writes it into `pending.json` and finishes. The
  *listener* then posts the notifier command — the generator has no chat tool
  and deliberately keeps none — delivers nothing, and leaves the Work open.
  The notifier's callback is itself a post in the run topic, so it triggers
  the run that collects the outputs with `agforge comfy fetch`. Image stays
  synchronous: it goes through SwarmUI, which has no `prompt_id` to give, and
  returns in seconds.
- Since p9 **one result names the requester once** — in the `assetplan-`
  delivery, which is what they were waiting for. The `assetrun-` copy is the
  record of the run and names nobody. "The requester" is read from the
  conversation as it stood when the run was served, not from the topic as it
  looks afterwards, because a generation takes minutes and anybody may post
  meanwhile.
- **Since `refactor` p3 ex1 forge publishes execution options**: `agy` and
  `agy-claude` beside its default, covering *everything a request is made
  of* — the entrance reply, asset planning, generation, and the run that
  collects after a ComfyUI callback. The `assetplan-` topic's selection is
  snapshotted into the `assetrun-` topic it opens (`[selfnote][exec]`), so a
  callback's collecting run continues on what the request was asked for:
  home decides, never the notifier's topic. **An option is not the media
  model.** It chooses the harness that reads and plans a request; which
  image, video or music model makes the asset is decided in the plan and
  named by the toolset, and the menu says so.
- **Since `refactor` p2 (2026-09-10) forge's record is its conversations.**
  There is no Plane issue behind an asset request: the plan is a visible post
  in the `assetplan-` topic, and `[selfnote][asset]`, `[doc]`, `[tools]`,
  `[assetrun]`, `[state]`, `[result]` and `[replaces]` carry what a program
  must resolve without guessing. **Identity is a message id** — the `[asset]`
  note's own id *is* the request (`a5814`) — so the run topic is named
  `assetrun-<stem>-a<id>` and a requester's stem can be reused without two
  requests merging, and the delivery follows the anchor home rather than a
  name, surviving a resolve, a rename and a retirement. A deleted origin is
  *absent* rather than whatever took its name.
  `python -m agforge.retire <channel> <topic> [--replace]` is the one
  retirement route; a job still running is collected by the old run topic and
  delivered to the request that asked. `collected.txt` beside the run's
  workspace makes a repeated notifier callback a no-op, so one job is never
  generated or delivered twice.

- **Since `study_import` p1 (2026-09-20) forge finds its own knowledge.**
  `agforge knowledge list | show | search | path` reaches the sources named
  in its ignored `.local/knowledge.toml`: mediagen's `main/` (general,
  publish-ready) and mediagen's `localize/` (what runs here, one INDEX row
  per capability with a state). Every index row is shown, unverified ones
  with their state; the planner reads what it likes and cites
  `<source>/<path>` in the plan, which is recorded with a
  `[selfnote][knowledge] mediagen@<rev>, localize@<rev>` note beside
  `[tools]`; the run's workspace gets `knowledge.md` saying whether a source
  moved since. Precedence when inputs disagree: the chat's words >
  `required_items.md` > local > general. `localize/` is a study-pattern
  extra folder with no `publish/`; its README says how to re-verify a
  capability and where each kind of problem goes (environment → cagent,
  script → autolab workplan, missing general finding → mediagen tip).
  First capability: `hud_icons` (drawn Pillow route, verified; the generated
  and whole-icon routes recorded as weaker or failed).

## Human-authored references (`adventure_game` p2, 2026-09-22)

The human keeps the things agents build *from* — stories, images, templates,
runnable examples — in a repository of their own, edited in a folder outside
every agent workspace and published by `git push`. Agents never write there.
A reference is named `<source>@<revision>[:<path>]` and every agent reads it
with the shared `agrefs` (pyagag, `agag.refs`): `list`, `sync`, `show`,
`path`, `search`, `revision`, `changes`. A snapshot is `git archive` of one
commit, kept immutable under the instance's `.local/refs/<source>/<sha>/`, so
several revisions coexist and a running task keeps the one it adopted; the
source name → URL map is the ignored `.local/refs.toml`, and a run is handed
`AGREFS_HOME` beside `AGENTCHAT_ZULIP_ENV`. `show` describes a binary (kind,
pixel size, bytes, path) rather than pretending to read it; the harness's
own image reader over `agrefs path` is the visual access. Front carries the
identity into `GOAL.md` and workplans, autolab records the adopted commit in
`direction/REFERENCES.md` and names paths per task, forge can steer SwarmUI
with a reference (`agforge image generate --init-image`), and archsage reads
references apart from its sages' trees — a project's input, never a study's
finding. Creative direction from a reference outranks technical knowledge;
the requester's words outrank both. Originals are the human's; agents write
derivatives in their own workspaces and say where they departed.

**Since `give_context_easier` p1 (2026-09-26) which sources exist is one
shared catalog**, not a file per agent. `developer/context-catalog` on Gitea
holds `catalog.toml` (`agag.refs-catalog.v1`): per source a stable `id` (the
`<source>` in references, never renamed), a display `name`, a `description`
agents read to choose, `repository` (owner/name, resolved against whatever
host the catalog was read from), `branch` and `active`/`archived`. The
catalog URL is a host fact, configured once per host in
`~/.config/agag/refs.toml` (`agag_agent` writes it on remote nodes from
`AGREFS_CATALOG_URL`), so every instance on a host — and every instance
provisioned there later, whose generated roles grant `agrefs` — discovers
the same set with no per-agent edit. `agrefs list` shows it (with the
catalog's state: current / last-known / unavailable); a name not yet known
re-reads the catalog at once, so a newly registered source needs no restart;
archived sources stay resolvable for old references. cagent's shell-less
window reads the same library through a `contexts` tool.

The Developer manages contexts from the Front Room: the `▤ contexts` panel
beside the composer (Front Desk and Arguing Room) lists and searches the
catalog, and a click inserts `<id>@<full commit>[:<path>]` at the caret —
resolved at that moment, so a later publication never changes a reference
already written; a draft's reference moves to a newer version only on
request. The same panel creates a repository (first README commit, then the
catalog entry), registers an existing one, edits name and description,
archives and reactivates, edits Markdown, uploads files and publishes — relay
operations (`/contexts…`) on the Developer's own Gitea token, no model run.
A publication names the revision the editor started from and is pushed
without force: if anyone published in between, nothing is written and the
panel shows what moved. Ordinary `git push` stays another way to publish.

## Explicit replies and journaled servings (`explicit_reply` p1, 2026-09-20)

**An agent's reply is what its run marks, not whatever it printed.** Until
this phase a reply into the served conversation was the run's whole final
output, and the only boundary between the agent thinking and the agent
speaking was a guide sentence ("no notes to yourself"); Front's #7222 went
to Zulip as two paragraphs of thought and then the reply, and the Arguing
Room voiced both. Now (`pyagag` `agag.reply`):

- The run puts what it wants said inside a fenced `ag-reply` block; several
  blocks are one post, in order; a reply may carry code fences (open the
  mark with four backticks). Everything outside the mark is the run's own,
  kept in the transcript and never posted. Machine blocks (`ag-argue`,
  `ag-routinerun`, `ag-continue`) are read from the whole output by their
  own splitters and never posted.
- No mark, an empty mark or an unclosed one is a **failed reply**, not
  silence and not "post it all": the run is asked once more for the reply
  alone with its previous output in front of it (`repair_prompt`; every
  action and machine block has already happened), and if that fails too
  the conversation gets one visible line saying this run produced no reply.
- `TopicResult` separates model `output` (under the contract), literal
  `sections` (deterministic lines, a canonical block echoed as the record)
  and system `notices` (appended after the reply). Structured-output roles —
  Front's `present`, generators, `observe`, `workrun_supercoder`, Observer's
  intake — keep their own contracts.
- The mark is described once, after every conversational guide, by
  `prompt_with_guide(…, reply=True)`; the prose prohibitions came out of the
  guides. Every consumer's conversational roles use it: Front's four,
  autolab's director and bmining, forge's plan front, cagent's front,
  archsage and its sages, the shared entrance and argue participants.

**A serving is a journaled record, and delivery is exactly-once**
(`agag.serving`, `agag.delivery`, in each listener's `listener.sqlite`):
received → acked → executed → prepared → delivered, with `failed` and
`interrupted` as the exits. The reply text is on disk before it is sent; an
ambiguous send is settled by reading the conversation back for the bot's
own post of that text; a `DeliveryError` leaves the text prepared and the
listener retries the *delivery* with bounded backoff, never the run;
exhausted or refused, the entry stays in the queue as `failed` with its
reason (visible in `entries("failed")`, the status file's `last_error` and
the log) and is re-armed by the next post. **One completion rule**: speech
by others past the input boundary the last delivered serving processed —
asked after the reply, before a resolve (input that arrived during a
cancelling run is answered first), at restart recovery and by `owed()`;
the bot's own ack is never evidence that anything was answered. A restart
marks an unreplied record `interrupted` and hands it to the next serving as
`context.previous`. The reply and delivery outcome are written beside the
run identity (`reply` in the `ag.agent-run.v1` record).

**Handoffs are bound to requests.** The reply names the requester read from
the processed input, never a speaker looked up at send time; a third party
who posts during the run is answered by the next serving. A `Conversation`
may carry an anchor (`front/front-1 #7225`, the id of the post the serving
was started for); runs get it as `AGENTCHAT_HOME_ANCHOR`, `agentchat send`
writes it into the root note, and `agag.zulip.locate` finds a conversation
by it before its name — the reply's destination is located at delivery
time, so a topic renamed or resolved mid-run is answered where it is now
(a `✔` name included; no twin), and one that is gone is a terminal failure
out loud. The served mark on the mention route is written by the listener
after a *confirmed* delivery, bound to the mention that triggered the
serving, so a mention arriving mid-run stays owed and a restart between the
home reply and the mark writes the mark without a rerun.

**The context to continue is carried, not reconstructed**
(`agag.continuation`). Every serving of a conversational role gets a block
between `===== BEGIN CONTINUATION =====` and `===== END CONTINUATION =====`
derived from the record: what arrived since the last delivered reply, the
agent's own goal / conditions / next as it last wrote them in an
`ag-continue` block (kept as `[selfnote][continuation]` in the served
conversation; any newer post overrides it and the view says so), where each
request made elsewhere stands (awaiting, answered and not yet dealt with,
dealt with, finished, unreadable — from the root notes and the served
marks), an interrupted previous serving of the same input, and what the
carried conversation left out. Front builds it from the thread snapshots
it already reads for `threads/`.

The phase's fixtures are `pyagag/tests/test_serving_lifecycle.py`,
`test_reply.py`, `test_handoff_binding.py`, `test_continuation.py` and
`test_end_to_end.py` (request → delegation → callback → final response with
a restart: 0 lost, 0 duplicated, one run per serving, recovery in well
under a second). `devdocs/episodes/agentchat/explicit_reply/p1/` is the
record.

## What a post is for (`clearer_chat_ui`, 2026-09-25)

A mention says whose turn it is; it never said whether anybody waits for an
answer. Every post can now say so itself, with one machine line at its end
(`ag.post.v1`, pyagag `agag.post`, `docs/post-intent-v1.md`):

```
`ag-post intent=response_request to=8 ask=question seen=11422`
```

- `intent` is `progress`, `report` or `response_request`; a request names
  the **user id** it waits for (`to`), optionally `ask=question|confirmation`,
  and the listener adds `seen=` (the input the serving had read). `re=<id>`
  names the request a post answers — or, from the asker, withdraws. **No line
  means unclassified, and unclassified is never "waiting".**
  `answer=none` says a post answers **no** request (a person's aside while a
  question waits); it contradicts `re=`. Leaving both out lets the next-post
  rule decide (ex1).
- A run declares it on its reply fence (```` ```ag-reply intent=report ````);
  the shared reply guide teaches it once, and a request without `to=` goes to
  the requester the serving recorded. A handler's own `response_request`
  (`TopicResult.meta`, e.g. autolab's task waiting for its requester's
  agreement) is a requirement: it stands over whatever the run declared, with
  the handler's `to` and `ask` (`agag.post.combine`, ex1); any other handler
  intent is only a default. `agentchat send --intent … --to …
  --ask … --re … | --not-answer` is the same for direct posts. The line is in the same
  message as the words, so the journal, redelivery and read-back carry it.
- Agents never see the raw line: chatlogs, `agentchat read` and Front's
  evidence say `(asks Developer to answer (question); request #9120)`.
- **What is still asked** is a read model over the history, the same for
  every room and consumer (`agag.outstanding`): a request is its message id;
  its post keeps its intent, its state is `pending`, `overtaken` (the person
  spoke after `seen`, so their input is owed a serving first), `answered`
  (by reference, quote-and-reply, or the next post when exactly one is
  pending), `withdrawn`, `superseded` or `closed`. Only the recipient's
  speech settles one; progress, acks and third parties never do. An answer
  is a receipt — approval and acceptance stay where they were.
- The rooms (Front Desk, Arguing Room, routine chat, Project Room) label each
  post with an icon and words, list what waits for *you* above the dialogue,
  and let you pick which question your next post answers (the relay writes
  `re=`) — or mark it "not an answer" (`answer=none`), which leaves every
  question waiting. The choice is shown before sending and kept on a failed
  send. An old question in the history keeps its label and says it was
  answered; it never reappears as waiting.
- Known gaps: the Observer monitor's "please get it moving" posts and the
  ComfyUI notifier's callback stay unclassified (they address agents or
  trigger runs, never a person).

## How agents remember each other

Since `agent_standardize` p8 an agent that speaks in another agent's
conversation writes a hidden line into it first:

```
[selfnote][rootchat] <channel>/<topic>
```

naming the conversation of its own that it is there on behalf of. That post
is the whole memory — no agent keeps a file of who it is talking to. When the
answer names the agent, the topic tells it which of its own conversations to
serve, and it serves that one and replies there.

`[selfnote]` posts are machine-to-machine. They are hidden from every
`chatlog.md`, every `threads/` file and `agentchat read` — from their author
too — and, crucially, **a selfnote is never counted as somebody speaking**,
so writing one never buys another agent a run. The convention is
`agag.selfnote`; agents add their own tags (agforge and autolab both anchor a
run topic to its Work with `[selfnote][work]`).

Since `agent_standardize` p9 there is a second shared note:

```
[selfnote][served] <channel>/<topic> <message id>
```

written **into home** once a callback has been answered. Recovery needs it
because the answer goes home: the agent never becomes the last poster where
it was named, so "somebody else spoke there and named me" is true forever and
every restart would re-serve every exchange the agent ever had. Both recovery
routes consult it.

**Every one of these lookups follows Zulip's `✔ ` resolve rename.** The post
that names an agent is very often the post that finishes the conversation,
and a lookup that cannot see past the rename reads an empty topic and drops
the callback silently — p9 lost a task's completion report to exactly that,
and the mission stopped for 26 minutes with nothing in any log.

**Retiring a plan renames its conversation, and a rename moves every agent's
notes.** That sentence has now cost two episodes, so it is here rather than
in anybody's code comment. `retire_conversation` renames the whole topic —
every message in it moves, including the root notes *other* agents wrote
there — and the replacement then takes the freed display name, which is the
point of retiring. A third party anchored in the retired conversation finds
nothing of its own under the reused name. Nobody may repair that by copying a
note: a root note is identified by its sender, so a copy would make the
record lie about who is party to what.

Since `routine_tests` p2 ex1 the **reader** carries it, through a relation
the replacing agent already writes:

```
[selfnote][replaces] <message id>
```

naming the retired work's anchor **by id**, because an id is the one
identifier a rename does not touch. A lookup that finds no note of its own
reads that pointer, resolves the id to the conversation it is in *now*, and
looks there — **one hop, then stop**. The note found must still be its own; a
missing, malformed or deleted pointer is *no anchor*, and nothing is guessed
from the reused display name, which is the one conversation the pointer
certainly does not mean. The relation is read **whoever wrote it** — it is
written by the replacing agent for the benefit of agents that wrote nothing
in the replacement, and filtering it by sender would leave exactly those
agents unable to read it.

There is also now a way to say an anchor is simply **wrong**:

```
[selfnote][rootchat-moved] <channel>/<topic>
```

One rule reads both notes, everywhere: **the newest valid explicit move
written by this agent wins; otherwise its earliest ordinary root note wins.**
A later ordinary repeat still never redirects a topic — that default is what
protects a live conversation from a careless second post, and p2 confirmed it
by repairing a mis-anchored delegation with an ordinary note and watching the
callback go to the old home anyway. Only an agent's own move moves its own
anchor. `agentchat anchor <channel> <topic>` writes it; `agentchat --help`
says when to use it and when not to. It is a selfnote, so it is invisible in
every chatlog and buys nobody a run, and because one rule reads it everywhere
a corrected delegate moves in `threads/` and in restart recovery too.

**A serving's conversation is in its prompt, not only in `chatlog.md`.** Also
`routine_tests` p2 ex1: twice out of that trial's first two requests Front
answered a brand-new conversation with *"I don't see a message or request
from the developer yet"* — one turn, no tool calls — with the request sitting
verbatim in the file. A conversation that is only a file is a file a run has
to decide to open, and no wording removes the possibility of a one-turn
reply. `agag.topics.conversation_context` carries the rendered bytes (the
same ones written to the file) into the prompt, bounded, saying what it left
out; Front's three roles, the shared own-channel entrance and forge's plan
front all use it.

## How an agent is found

Each agent posts its own introduction to the shared `#agents` channel, under
an append-only `intro-<instance>` topic. **That post is the contract**: it is
where another agent learns the entrance, the topic prefix and what is safe to
do — so routing vocabulary travels as posted content and is never compiled
into a consumer's guide. Re-post after a behavior change; a stale
introduction is acted on as if it were current.

An introduction is also where an agent says what it needs *from* the
requester. autolab's says a task is not closed until the requester agrees it
is done — a contract that lived only in its code until p5, where a supervisor
had no way to learn its own part in it.

**A ✔ on the `intro-` topic retires the agent** (`operation_room` p2 ex2).
It is the realm's only way to say an instance is gone: a project can be
deleted from every machine and Zulip never hears about it, which is how
`agping-agstudio1` — `runsmoke1`'s agent-created fixture — sat on the
operation room board as an unroutable *unknown* for a whole phase. A retired
instance leaves the agent room and the operation room, its channel stops being
walked for open work, and both name what they retired rather than letting a
card quietly stop being drawn. It is a flag, not a deletion: un-✔ the topic
and the agent is back.

**A retirement has a node half, and `refactor` p3 ex1 found out the hard way.**
Removing a placement from desired state stops the production inventory
carrying an agent, so nothing redeploys it — and the systemd user unit it
already installed stays *enabled*, so the next boot starts the listener, which
re-posts the introduction under the freed bare name of the resolved topic and
opens a **twin**. Two topics, one resolved and one not, and the un-resolved
one is what the boards read: a retirement quietly undone by a deployment.
That is how `agecho` came back. `ansible_agdev/playbooks/agent/retire_agag_agent.yml`
is the inverse of `setup_agag_agent.yml` — stop, disable, remove the unit —
and it stops there, touching neither the bot account nor the flag. Folding an
existing twin needs `resolve_topic` on the twin's own last message;
`agentchat resolve` answers "already resolved" and leaves it.

Since `operation_room` p2 the post also carries a fenced **roster block**
(`ag.agent-roster.v1`, `pyagag/docs/agent-roster-v1.md`): the Zulip name the
instance is mentioned by, the channel whose every topic it answers, and the
prefixes it sweeps elsewhere. It is generated from the running instance when
it posts, so it cannot drift from the listener. It exists because none of that
is readable from outside — prefixes are compiled into an `AgentSpec`, the
instance name lives in that node's ignored `.local/instance.toml` — and
`operation_room` p1 guessed both and produced 66 phantom stalled rows. **A
consumer must keep a missing block as *unknown*, never as "no prefixes".**

## How an agent is asked to run a particular way

Since `runtime-profile` (2026-09-09) a request may name a *way of executing*
— "run it using agy" — and there is a public name for one that is not the
recipient's internal profile. `ag.exec-options.v1`
(`pyagag/docs/exec-options-v1.md`) keeps the two apart:

- an **execution option** is public: a name an agent advertises in its own
  introduction, with the **usage pool** it consumes and the **work** it
  covers;
- a **profile** is private, the `agents.toml` name it maps to. Nobody outside
  the agent may name one, and no consumer's code contains one.

The introduction carries a fenced `agag-exec` block beside the roster's,
generated from the running instance — so it is a small menu, never a dump of
`agents.toml`, and it cannot advertise a profile that is not configured.
**A post with no block is *unknown*, never "does not support it"** — the
roster block's rule, for the roster block's reason. **Since `refactor` p3 ex1
every agent in this realm publishes one except cagent**, which predates the
contract and correctly reads as unknown.

**The `pool` in that block is derived, not written down** (p3 ex1 step 3,
`agag.execpool`). Filtering option *names* against the configured profiles
kept a name from being advertised without a profile behind it; the pool beside
the name was a string in a tuple that nothing compared to a harness. Four
agents said `pool: anthropic` for their default and were right *by
coincidence* — one line in a machine's `agents.local.toml` moving a role to
`agy` would have left every published default lying while the code stayed
correct. So the pool is resolved for **every role the option covers**
(`AgentSpec.exec_roles`) through the option-to-profile mapping, `agents.toml`
and the instance overlay, and what is published is the resolved value. Three
rules follow: several pools are joined with `+` rather than rounded to one,
because an agent whose planning and task work resolve differently really does
spend two accounts and a threshold must be judged against both; a harness
that is not installed is a runtime failure of that one option, never an
unpublishable contract, so derivation never checks availability; and the
agent's own declaration is compared against the derivation and every
disagreement is logged at startup and before an introduction is posted.

**Selection is a topic-local command**, addressed to the topic's owner:

    @**<their Zulip name>** use <option>
    @**<their Zulip name>** use default

posted **on its own** — a mention inside a sentence is discussion, and a
mention inside a code fence is not a mention, which is how this paragraph
quotes it without firing it. A message that is only commands is
**configuration**: the setting is applied and *no model runs*. The owner
answers with one deterministic line of its own — a reaction would leave the
poster as the last speaker and the topic would match every sweep forever
(the ComfyUI notifier reacts for the opposite reason: it is not the owner).
A confirmation names nobody; a **refusal names the poster**, because an agent
that asked for a name nobody publishes will otherwise ask again. A refused
name never becomes the topic's setting.

`agentchat options` prints what every agent publishes; `agentchat use
<channel> <topic> <option> --to "<name>"` posts the command. Both are how an
agent discovers and asks — an option name is one agent's vocabulary, so a
further delegation means discovering *that* agent's options and translating
the intent again, never forwarding a name.

**The selection is frozen at each serving's start.** A command posted while a
run is in flight lands on the next serving, callbacks included. The store is
the topic itself: there is no selection file, so a listener restart
re-derives the same answer and "why did this run on agy" is answerable by
reading the conversation. A child conversation an agent opens for work
(autolab's `workrun-` topic) carries a **snapshot** of what the parent was
set to, `[selfnote][exec] <option> from <channel>/<topic>#<id>`, written
before the visible description — so a later re-plan reaches the tasks it
opens now and leaves the running ones alone, and a child overrides by
carrying its own command. **A callback's remote topic is never the execution
context**: the selection is read from home.

The **pool** is the provider whose account the harness spends
(`anthropic`, `antigravity`, `openai`, `google`), which is what lets "until
agy's usage exceeds 70 %" be matched to a window: `agfront.budget` prints the
same name beside each harness in `tools/budget.md`. A harness whose account
follows its model (agcode) says `pool unknown` and matches no option — an
unobservable condition, not one at 0.

Proven live on 2026-09-09: Front read the board, selected `agy` in a
`workplan-` topic, and autolab planned *and executed* the mission on it —
`superdirector` and `supercoder` records both `profile: agy`, the task's
`exec_source: inherited` from the plan's message — while another topic of the
same agent, served the same minute with no command, ran on the
`claude_code` defaults. See `devdocs/episodes/pyagag/runtime-profile/`.

# Adopted Policies for Development of In-System Agents

## Favorable

Easier Next Time
Single Entrance
Tool Giving
Evidence-Driven Guidance
Failure Farming

## Unfavorable

Anxiety-Driven Guidance
Tool Implantation
Unexplained Chainsaw

## Agent ≠ Model
- The backend model/harness is a swappable parameter of an agent, never its identity.
- Every agentic run records which backend served it. See devpolicy/agent_records.md for the common record.
- Since `runtime-profile` a run also records **what was asked for** — the
  public execution option and where the selection came from — beside what
  actually ran. The option is a request; `profile`/`harness`/`model` stay the
  fact. Which way an agent is asked to run is a parameter of the
  conversation, never of the agent.

# Other policies

## Deus Ex Machina note
- When the Omni Agent performs work that belongs to an in-system agent, leave a one-line note in the episode doc: "did X for agent Y — handoff candidate".
- Perhaps positive for the mission, perhaps negative for workflow growth; the note is the whole obligation.

# Ansible Commands

Run from `pj-clusterintent/ansible_agdev`:

- **Update autolab nodes**:
  ```bash
  uv run --project ../nctl nctl render production --out inventories/generated
  ansible-playbook -i inventories/generated/production.yml playbooks/agent/setup_autolab_node.yml --limit agautolab1
  ```
