# Rule

- Local PC/cluster details (addresses, host names, credentials) are kept in
  ignored files; tracked files are read on other machines and by others.
- Agents work without process gates (workflow_editor p11): approvals,
  agreements, confirmations and acceptances are records, never permissions.

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

**Since `agent_guide` p2 ex1 the command is learnable from the board.**
The notifier posts its own introduction in `#agents`
(`comfynotify/params/intro.md` through `agag.intro.post_intro`):

- it is a tool, not an agent;
- the one command, quoted in a fence;
- what comes back;
- why a mention inside a sentence gets a refusal.

It is in every run's `tools/agents.md` and in `agentchat intro`. Its roster
block declares no channel and no prefixes, so the operation room (retired in
retire_old_ui p1) showed it as an instance with nothing to serve rather than
as an unreadable agent.
The daemon posts it at start-up when the board's newest one differs;
`comfynotify intro` posts it by hand. autolab's worker guide now only
points at it.

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

## cagent (managed workflow project, project_migration p5)

cagent is now the managed project `cagent` (`pj-cagent`, private repository
because its cluster records name hosts). Its entrances are
`wf-cagent.consult`, `.report`, `.scan`, `.change`, `.develop` and
`.freework`, through request topics, the WFE run view or delegation.

- **Backend (project_migration p6 ex2, 2026-10-17).** Every cagent workflow
  declares `execution.profile: luna`: autolab's `codex` harness on
  `openai/gpt-6-luna`, effort medium (`agautolab/agents.toml`). Claude
  declined the Developer's chosen platform instead of developing for it.
  The agent identity is still `cagent-agent`; Agent ≠ Model. Codex runs
  record tokens, never cost, so the gauge shows nothing for them.
  - `gpt-6-luna` needs a current codex CLI: 0.153.2 answered 400 "not
    supported with a ChatGPT account"; 0.163.0 (`codex update`) runs it.
  - pyagag runs codex with `features.shell_snapshot=false`. The snapshot
    replays `~/.zshrc`, whose PATH entry for the old
    `.local/workflow-editor-p2` area won over the executor's `wfe`; the
    first codex run could record nothing.
  - A held run moves to another profile by a person's resume
    (`wfe run resume --profile <name>`, or the run view's resume form;
    agdevworld `c6390a2`). Rollback is `profile: sonnet` in the workflow
    files, re-approved, and a resume with `--profile sonnet` for a held run.

- `develop` (project_migration p6 ex1) develops missing deployment and
  observation support in `backend/pj-clusterintent`, a run-owned copy of
  the superproject and its submodules (WFE owns nested submodules since
  agdevworld `b40378f`). Acceptance integrates the commits and pins; no
  host changes. A change then plans `sources_update: true`, and
  `cagentctl sources update` moves the live checkout to the integrated
  commit through the ledger. An unsupported platform is a `develop`
  request, not a reason to choose another host.
- A macOS host can serve NFS (`nfs_server` dispatches by OS). It cannot
  mount its own export: automountd substitutes a symlink. Its local
  processes use the backing path, and the share owner is their identity.
  macOS client mounts are marked fstab entries for the stock
  `/- -static` map, with the marker on its own line because `getfsent`
  drops entries with trailing comments.
- Known gaps (p6 ex1 report3):
  - nctl compares NFS placements by existence only, so a config change
    needs the allow-listed setup playbook;
  - one service cannot mix deployment profiles;
  - cagentctl has no ledgered one-off privileged effect.

- `consult` explains reported/observed/desired state with evidence and ages.
- `report` records installations, ownership and purpose with provenance;
  verification uses existing observations, not an implicit host scan.
- `scan` collects and ingests through `nctl observe`, records unavailable
  hosts honestly and publishes `snapshot.json` (`cagent.snapshot.v1`).
- `change` executes from an ordinary run context or `--context DIR`, with no
  plan agreement, approver, batch-hash, host or playbook allow-list
  (workflow_editor p11 step 3). `cagentctl` records external effects,
  actual inputs and outcomes, and recovers unknown outcomes from backend
  operation evidence.
- Destroy-class guest operations follow the same recorded execution path;
  the person handoff was removed in p11. nctl has no `allow_destroy` or
  imposed `allowed_hosts`; previews stay read-only unless `--yes`.
- Result acceptance and Git integration publish records; they do not undo
  external effects. An unknown outcome is never a reason to repeat it.
- `cluster:fetch` read the latest completed scan artifact through WFE's
  read-only file API into agdevworld's cluster views; both are retired
  (retire_old_ui p1). The independent refresh job uses observation only.

Front reaches cagent through its workflows (project_migration p8); the
argue host's consult invocation retired with the argues. Old HTTP/listener source, incidents and request evidence remain
history; the section below describes those retired paths.

## cagent (pj-clusterintent) — legacy implementation history

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

## autolab (pj-agdev/agautolab) — the workflow executor

autolab is the host that executes WFE workflow runs, and the Zulip transport
for their requests and coordination. It has no conversational entrance and
no introduction on the board.

- **Executor** (`agautolab.wfexec`, launchd `com.agdev.agautolab-wfexec`):
  the coordinator's local execution node `studio`, one slot of its own (see
  *Execution nodes* below). It claims a queued run of its node (`wfe exec claim`) and launches
  the agent the workflow names, in the run's own copy of the project. The
  prompt is the common contract (`agent/guides/wfrun/guide.md`), the run's
  scoped context and its brief. It reports the end (`wfe exec finished`),
  carries out stops, and reconciles vanished processes every minute. The
  session rules are in *Workflow run sessions*; every state rule lives in
  WFE.
- **Transports**, threads of the executor:
  - `wfnotify` posts each run's coordination to its `run-…` topic in the
    workflow's `wf-…` channel;
  - `wfintake` hands the posts of routed channels to `wfe intake receive`.
  
  Both use the autolab bot (`autolab-agstudio1`, user 11). Nautobot:
  placement `agautolab-agstudio` (`manual_toolchain`, process
  `agautolab\.wfexec`) and DesiredAgent `autolab-agstudio`.
- **Profiles**: a workflow selects one with `execution.profile` (`sonnet`,
  `luna`, …), from `agents.toml`; the role is `wfrun`. Agent ≠ Model.

**The old development system is retired (`retire/old_autolab`, 2026-10-19).**
These are gone:
- the Zulip listener and its `workplan-`/`workrun-`/`bmining-` topics;
- missions, tasks and their acceptance (`agag.acceptance`, `agentchat
  accept`/`reserve`, `mission_done`, `correct_state`);
- mission working copies, project init/archive and study setup;
- the `:8791` gateway (both placements) and the `autolab` CLI;
- the `autolab_node` Ansible role and agentroom's Project Room.

Development work for a project is a WFE workflow request. Old `pj-<slug>`,
`work-m<id>` and `workplan-…` conversations remain as history. Nothing serves
them, and the mission branches they name are all integrated.

## Execution nodes (`project_migration` p7, 2026-10-19)

WFE runs are performed by **execution nodes** (workers) of one coordinator
(the WFE service and its store on agstudio). Contract: agdevworld
`workflow_editor/docs/workers.md`; recipe for another PC: pj-clusterintent
`nctl/docs/add-a-wfe-worker.md`; episode: `devdocs/episodes/project_migration/p7/`.

- **Nodes now**: `studio` (local, the coordinator's host, default) and
  `gpupc` (agpc: Ubuntu, NVIDIA GPU, desktop `:0`, Godot 4.7.2, Claude Code,
  `agshare` at `/mnt/agshare`). Listed in `<area>/workers.json`; each has
  one slot, so two runs execute at once.
- **Choosing**: `wfe request … --node <id>` (or the browser route's
  `node`), else the workflow's `execution.node`, else the project's, else
  the default. Recorded in `run.create`; every attempt runs there;
  `wfe run assign` moves a run between attempts, never anything by itself.
  A job its node cannot serve waits with the reason; a lost node's runs wait.
- **Records stay central**: the coordinator's record copy holds the
  authoritative `run.json`. The PC works in a copy of its own
  (`~/wfe-worker/copies/…`), materialized from the published run branch;
  its `wfe` forwards every run operation to the coordinator (uploading the
  run folder and named files first), ships the attempt log, and publishes
  its copy at the end, after which the record copy follows. Questions,
  answers, decisions, integration, Zulip and Observer are unchanged and
  central.
- **Views**: the run view (Detail) names the node, its last contact and the
  attempt "on <node>"; Home has one chip per node; `wfe exec workers`,
  `GET /api/workers`, `wfe observe` `workers[]`.
- **Contact lost** is not a lost process: the attempt stays open; the
  PC's agents survive an executor restart (`KillMode=process`) and the
  executor adopts them (`wfe exec adopt`); a gone process ends with the
  exit its launch gate recorded (`<log>.exit`), or as `unknown`.
- **Deployment**: deployment profile `wfe_worker` (ansible_agdev), placement
  `agautolab-agpc`; installed by cagent `change/run-008`. A pin change of
  the placement needs the setup playbook (nctl compares by existence).
- **Lanes** (p8): a worker may declare extra slots per named lane
  (`workers.json` `lanes`); a workflow names its lane (`execution.lane`).
  Studio has `conversation: 1` for Front; remote workers refuse extra lanes
  for now.
- **Limits**: a worker offers every profile of `agents.toml` (only `sonnet`
  works on the PC); an external wait (`wfe run wait`) is woken only by a
  resume.

## Opsroom Observer (not an agent)

`Opsroom Observer` is a Zulip bot with no listener, no runs and no
conversation: it is the credential the agdevworld relay's **mirror** reads
the realm with (`agdevworld/agentroom`, `operation_room` p2; since
`better_zulip_call` p1 every read the relay makes comes from that one
persisted copy, and the Developer's credential only writes). Since
workflow_editor p15 the relay reads no Zulip at all (the Front Desk's
progress panel and the Arguing Room were its last readers), so this
credential is no longer used by anything running. It existed so the relay's
reading never came out of the quota the agents' own listeners or the
Developer spend.
It is **read-only by operation** — Zulip has no read-only API key — and its one
write to the realm is subscribing to public channels, which an event queue
requires and a read does not. It posts **only** in `#ops-testbed`, which no
agent is subscribed to, and never names a real agent bot even there: a mention
in a public channel reaches a bot that is not in the room, which is the
`zulip_command` lesson that cost one paid Front run per lap.

Since `operation_room` p2 ex2 the relay it reads with is **always on** —
launchd `com.agdev.agentroom` on agstudio. Its mirror is a running copy
rather than a per-request read, so a relay somebody has to remember to start is
a panel that is not there when it matters.

**One direct message is the exception** (`robust_workflow` p2): the relay's
watchdog over Observer's request monitor DMs the realm's owners — humans, no
agent named, so it buys no run — when the monitor goes into a failing state
and when it comes back. `#ops-testbed` has no human owner in it, so a post
there would have reached nobody.

**The observer still never posts.** The same relay writes on a credential of
its own (the Developer's) with no fallback in either direction. Since
retire_old_ui p1 its one write is closing an argue; `POST /chat` and the
routine run requests are gone.

## Routines (`refine_routine` p1) — retired with the legacy Front (project_migration p8)

Nothing runs a routine any more; the channels and guides stay readable. A
recurring need is a request to Front or a project workflow. History below.


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
- **A run is Front's to continue, adopt work into and end from anywhere**
  (`progress_panel` p1, reworked in `failsafe` p5): `agrun` (agfront,
  granted to `front`, `desk` and `routine_run`):
  - `agrun continue <run> --because <post>` has the run served again
    through the listener's own `[selfnote][start]` — Front's own post in
    its run serves nothing;
  - `agrun adopt <work topic> --run <run>` moves Front's root note in work
    opened beside the run (`[rootchat-moved]`), so its answers serve the run;
  - `agrun finish <run> --achieved|--not-achieved --reason … --report …`
    writes the end record and completes the close-out;
  - `agrun status` (and `tools/runs.md` in every desk/front serving) says
    which runs a conversation opened and where each one's end stands.

  **The close-out is resumable**: the end record comes first (a run's own
  serving posts it as its reply), and `agfront.routine.close_out` then
  delivers the report, writes `[delivered]` and resolves the run — each only
  if missing, after every run serving, from `agrun finish`, and at startup
  for ends of the last 24 h. An end record does not imply a delivery.
  Prose saying a run is complete, or a hand ✔, ends nothing.
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

Since `observer` p3 (2026-10-14) Observer watches **workflow runs**, not
conversations. The contract is `agobserver/docs/workflow.md`; the read side
it relies on is WFE's `docs/observation.md`. Episode record:
`devdocs/episodes/observer/p3/`.

- **What it reads.** `wfe observe --json` every 60 s — WFE's read-only
  snapshot of every run (its `uid` = `<project>/<workflow>/<run>#<creation>`,
  record, attempts, process group, log facts such as an open tool call with
  its declared timeout, queue position, waits with their responsible party,
  publication and integration problems) and of the executor's heartbeat.
  Observer reads no SQLite, no copy layout and no Zulip history to decide.
- **What it decides in code** (`wfclassify`): a person's hold, a question,
  an external wait (inside its `--deadline`), live work and a queue wait
  behind healthy work need nobody; `interrupted`/`unknown` holds, attempts
  whose process is gone, unowned continuations (an answer or ready work with
  nothing queued), stalled queues, an executor without heartbeat, failed
  publications/integrations and unreadable sources are suspicions with an
  identity `(run uid, family, episode)` — one incident per episode, whatever
  the wording, across restarts. A live attempt that is silent is
  `uncertain`: judged by the local model (`wftriage`), never given a second
  writer, told to the owners at its bound.
- **Recovery is a workflow.** A recoverable incident becomes a
  `request-<incident>` post in `#wf-agops.recover`; WFE's intake accepts it
  from Observer (`zulip.json` `intake.agents`) and the post becomes an
  `agops/recover` run. The recovery agent investigates and decides; to
  continue it records `wfe run recover` (interrupted/unknown holds or an
  unowned continuation; never a person's stop; no budget since p11). Observer records `restored` only when the run's agent reports
  work afterwards. A recovery run is never itself recovered; the executor's
  outage is told to the owners, not sent to recovery.
- **Records.** `agobserver/.local/workflow/observer.sqlite` (incidents,
  suspicions, judgments, assignments, outbox, events); posts go out
  journaled and are looked for by marker before any repeat; incident notes in
  `#agobserver-agstudio1 › incident-<id>`; owners by DM.
- **Processes.** `com.agdev.agobserver` (the monitor; writes
  `workflow-health.json` and `agag.status.v1` for nodeutils) and
  `com.agdev.agobserver-watchdog` (a separate process: Observer's freshness,
  loop failure and recovery backlog, and the executor's heartbeat through
  `wfe`; owners told once per change). The agentroom relay no longer
  watches Observer.
- **Retired.** The `#front` conversation monitor (traces, serving probes,
  Front asks, holds/dispositions on requests, developer reviews in Zulip)
  and the watch entrance's process. The watch modules remain a library; the
  old local state is archived under `agobserver/.local/archive/`.

## Delegation and detailed IO (`workflow_editor` p10, 2026-10-15)

**A delegate node entrusts its work to a run of another managed workflow.**
Contract: `agdevworld/workflow_editor/docs/delegation.md`; episode:
`devdocs/episodes/workflow_editor/p10/`.

- **Definition**: `type: delegate`, `project:` (absent = this project),
  `workflow:`, `accepts: person | parent` (who may accept the child's
  result; default person). The editor offers every registered project's
  workflows and validates across them (recursion included).
- **Who does what**: the parent's agent writes the request and dispatches
  it (`wfe delegate request`). Delivery is the notifier's: the request is
  posted in the target workflow's channel as a new `request-…` topic. The
  child is an ordinary run created by that workflow's intake.
  - Every workflow has that intake unless it declares `intake: {zulip: ""}`
    (workflow_editor fix2). Conversation and discussion turns (`converse`,
    `discuss`) do opt out. Before fix2, a file without a declaration loaded
    as "none" although creation wrote `requests`. Two aghangout workflows
    could therefore not be delegated to (front/converse/run-045).
  - The intake binds a post to a delegation only by the message id that a
    sent delivery recorded. The child's `request.md` is the parent's file,
    checked by hash.
  - The child's questions, failures and result are relayed to the parent as
    numbered facts (`wfe delegate show`). The parent's agent takes them up
    and answers (`wfe delegate answer`); the answers are posted in the
    child's request topic and recorded there.
  - It judges the result (`wfe delegate dispose satisfied|rejected|
    abandoned`). Only then does the node complete.
- **Waiting frees the slot**: the parent's attempt ends with the node
  waiting, and the child runs. A waking fact queues the parent (one job per
  fact). An attempt that ends without taking up a fact it was started for
  ends `interrupted` (a liveness signal, like work left ready).
- **Separate records**: the child's acceptance (its own decision from its
  confirmation), the parent's receipt (take-up), its disposition, and the
  parent's own result decision. None gates another since p11
  (`delegate-scope` was removed).
- **Controls**: a parent hold keeps facts and queues nothing. Stopping or
  cancelling the parent never touches the child (`wfe delegate cancel` is
  the explicit act). No replacement child is ever made automatically.
- **Observation**: `wfe observe` has `delegations[]` and
  `waiting.responsible = delegation:<child uid>` (or
  `delegation-unbound:`). Observer: `awaiting-delegation` is quiet;
  `relay-stalled` (300 s) and `unbound-dispatch` (900 s) go to the owners.
  A child's interruption is the child's incident only.

**Detailed IO: one Zulip channel per run** (`docs/io.md`):

- `wf-<project>.<workflow>.<run>.io`, topic `io`, fed from the attempt
  logs: agent text, tool calls and results, session start and end, attempt
  boundaries. Never reasoning or harness configuration; redacted (patterns
  plus `zulip.json` `io.secretFiles`). Long output links to the whole entry
  in the run view (`…/run/<wf>/<run>/io/<attempt>/<line>`, or `wfe run io`).
- Positions persist in `notify.sqlite` (`io_attempts`); a truncated or
  missing log is said to be a gap. Output only: the intake never reads it.
- The request receipt and the run topic's "Started" carry `Progress`,
  `Detailed IO` and `Run` links; a follow-up gives the IO link when the
  channel was not ready at the receipt. The run view's title band links
  Conversation · Progress · Detailed IO.

## Scoped context (`workflow_editor` p13, 2026-10-18)

What a run or an authoring session is given besides the run's records is a
scoped bundle: system / project / workflow, each a repository-owned
default (contract, `project.yaml` intent, goals and `guides`, workflow
intent and `execution.guides`) and a local override of the WFE area
(`<area>/.local/contexts/`, never in Git). Run input keeps its own records.
Contract: agdevworld `workflow_editor/docs/context.md`; episodes:
`devdocs/episodes/workflow_editor/p13/` (and `p13/ex1/`).

- Edit preferences where you work (p13 ex1): the Context section of Home
  (all projects), a project or a workflow editor, and a run's Context card,
  in Simple and Detail alike; or `wfe context set|clear` (revisioned,
  conflict-checked). Purpose, goals and guides are edited there too, with
  the file's own save; local instructions save separately. `#/context` is
  the cross-scope overview. A saved preference reaches the next attempt
  only; steering current work is an instruction (the run card's *Add an
  instruction for this run*, the conversation, or `wfe run instruct`).
- Each attempt records a snapshot id and hashes; the run view's Context
  section and `wfe context run <run>` compare it with the next attempt.
- Project guides are declared once in `project.yaml` (`guides`); a
  workflow's `execution.guides` hold only its own.
- Scope is breadth, not precedence: an explicit request refines a general
  preference.
- The authoring entry for an IDE agent is `<area>/authoring/` (generated on
  every release); never place `AGENTS.md`/`CLAUDE.md` above run copies.

## Context blocks and cagent's contributions (`workflow_editor` p14, 2026-10-18)

Local context is a set of **independently managed blocks** per scope
(system = this WFE area, project, workflow), each with a title, a kind
(`instruction` or `information`), a manager, a last editor, evidence,
details and its own revisions. Contract: agdevworld
`workflow_editor/docs/context.md`; episode:
`devdocs/episodes/workflow_editor/p14/`.

- **People** use the Context sections (Home, project, workflow, run card)
  in Simple (p14 ex1): each block is a row like a workflow in the
  Workflows list — title, manager, scope where needed, a short exception —
  and opens in a side panel for its full text, details, *Ask*, *Edit*
  (Remove is there) and *Evidence & history*; sizes and hashes are in
  Detail. Ownership is shown, not enforced; a stale save conflicts and
  keeps the draft. Episode: `devdocs/episodes/workflow_editor/p14/ex1/`.
- **Agents** use `wfe context list|get|set|check|refresh-failed|remove|history|detail`
  (`--base` = the content revision read; `--project` required for project
  writes; `--service <url>`/`WFE_SERVICE` from another host). `check` is
  bookkeeping and changes no prompt or session; everything in a block's
  delivered section (facts, evidence times and status, a failed refresh,
  details, consultation) changes the fingerprint.
- **Evidence** keeps publication, observation and verification times
  apart; observation is the source's own time, never the publication's.
  Details (`context:<scope>/<block>/<name>`) are read on demand, never in
  a prompt.
- **cagent** maintains `cagent-cluster` (system) and `cagent-resources`
  (per project) through its `context` workflow (`wf-cagent.context`,
  guide `agent/context.md`): an overview of what is relevant, the
  uncertainty that matters and where to read more; chronology, methods
  and provenance live in details and evidence (p14 ex1). A refresh is an explicit request; no
  recurring trigger exists yet (designed as the next step: scan/change
  results, new projects, project-intent changes, coalesced).
- The p13 single overrides were converted into Developer-managed
  `developer` blocks with their history.

## Workflow run sessions (`agent_session` p1, 2026-10-14)

A workflow chooses how its runs use harness conversations:
`execution.session: fresh | run` (Detail → Workflow settings → Execution;
not declared = `fresh`). The choice is frozen with the run.

- **`fresh`**: each attempt starts a new session from the records.
- **`run`**: later attempts resume the run's retained session by an explicit
  id. WFE mints the id and records it in `attempt.begin` before launch.
- **When `run` starts a new session anyway**: a changed configuration (agent,
  profile, backend, contract or guides), a retirement (`wfe run session <run>
  --retire`), or a conversation missing from the host's store. Each is
  recorded with its reason.
- **Unsupported profiles**: `run` on a profile whose harness keeps no
  resumable conversation is refused. Only `claude_code` keeps one.
- **Waits** still end the attempt and free the slot.
- **Observer** reads `session` from the record; incidents stay per attempt.

Contract: `agdevworld/workflow_editor/docs/sessions.md`; harness seam:
`agag.harness.Session`.

**Measured (p1 report5): `run` did not save money on RTS.** Two matched pairs
cost $8.07 for `run` against $5.99 for `fresh`. A retained session carries
its whole history into every request, and loses the prompt cache across a
wait longer than an hour. The bootstrap it saves is only 16–22 k tokens
written. The default stays `fresh`.

**Continuation (p1 ex2, 2026-10-15).** A resumed attempt is not given the
bootstrap again and is not asked to read the history. It gets the claim's
`continuation`:

- why it runs;
- the record entries since its previous attempt began that it did not
  write;
- the compact state;
- every answer, instruction and delegation fact not yet taken up, in full;
- instructions only for documentation steps it reaches for the first time.

`wfe run state <run> [--since <seq>]` is the compact read and the
node-boundary check. `wfe run show --history` stays a diagnostic. A
session compacted since its last bootstrap (or whose logs cannot show it
was not) is resumed with the whole bootstrap again, recorded as
`settings.session.reconstruct`. New sessions' briefs name the request and
the agreed `plan.md` on every attempt.

Measured on one RTS pair: no history reads on resume, continuations ≈1.7–1.9 k
chars, but the cost is still dominated by carried history: run $3.02 against
fresh $1.84, mostly one extra correction attempt. The default stays `fresh`.
A plan→implementation session boundary is designed, not implemented
(`devdocs/episodes/agent_session/p1/ex2/`).

**Larger tasks (p1 ex3, 2026-10-15).** Three distinct RTS features were run
once per policy: rally points, Shift waypoints and save/load. Each run
included a staged follow-up and up to one evaluated repair, and answers were
delivered only after the attempt's process had ended. All six runs were
accepted. Run cost against fresh: $6.00 vs $6.57, $3.02 vs $3.71, $6.08 vs
$5.57, so $15.10 vs $15.85 in total. Fresh attempts rebuild context: history
reads and about twice the tool output. Retained attempts carry 2–5× larger
prefixes. The two roughly cancel. The result is not a basis for changing the
default, which stays `fresh`; `run` is a reasonable choice
(`devdocs/episodes/agent_session/p1/ex3/`).

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
- **A run's health is its own record** (`failsafe` p2, pyagag
  `agag.execution`, `agag.health`). `run_harness(live=…)` keeps one JSON
  record per run:
  - the serving (ack, conversation), the pid and the deadline;
  - every event, and the tool calls not yet returned;
  - the end, written by the runner.

  A claude_code run with a record streams partial messages, so a long
  generation is progress. `python -m agag.health` reads the record, the
  process table and the listener's journal, and answers `running`,
  `waiting`, `stopped`, `ended` or `unknown`, with each fact's time and
  source and what it could not establish. A record of another serving is
  never applied. The trace's `Node` also says what the post ending a
  serving declared (`ending_intent`, `ending_to`).
- **Execution and holder are separate facts** (`failsafe` p1, pyagag
  `agag.trace.Node`).
  - A listener marks the reply that closes a serving with `end=<ack id>`
    (`ag.post.v1`). So `execution` is `open` (acked, not ended), `ended`, or
    `unknown`.
  - `holder` says who holds the next move of unfinished work:
    - a child conversation, or an agent the current serving named in the
      same conversation (the ComfyUI notifier watch), holds it until it
      answers;
    - a child waiting only on its requester holds nothing, so a circular
      wait reads as `none`.
  - `agentchat trace` prints both under every conversation.
  - A post Zulip cut (`[message truncated]`) is output, never an answer.
- **`agentchat recheck <anchor> --after <ack>`** (`failsafe` p4,
  `agag.trace.recheck`) re-reads the one conversation that holds stopped
  work and says whether its **owner** resumed it after the stopped
  serving:
  - FINISHED (its record), RESUMED, RESUMING, ASKED (a post waits for its
    owner), STOPPED or UNREADABLE, each with the next move;
  - the requester's own acknowledgement or activity elsewhere is never
    evidence;
  - Observer's stop requests name the exact command, and Front's guides
    act on its verdict. p4's three operational trials: two resumes on
    STOPPED, one abstention on RESUMING, no competing run.
    `compose` now cuts long posts before their line.
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

## The progress panel (`progress_panel` p1, 2026-09-27) — retired with the Front Desk (workflow_editor p15)

The Front Desk has a **`▦ progress`** toggle: a panel in the right column
listing every request in flight in *any* Front conversation, the one open in
the room pinned first, with a plan meter per plan, a run band under each
task, the stages still owed, whose move it is, and links to the evidence.
Record: `devdocs/episodes/progress_panel/p1/`.

- **One interpretation, shared** (pyagag `agag.progress`, `agag.progress.v1`).
  A card is a request (its origin's first post, `o<id>`, Observer's key);
  each conversation below it is a unit carrying **work** (trace state and
  record word), **execution** (the serving the conversation shows and, for a
  probed owner, the `agag.health.v1` check of *that* ack, with `evidence`
  `confirmed`/`stale`/`conversation`) and **recovery** (Observer's incident,
  a person's hold) — never merged. The display states are `planning`,
  `queued`, `working`, `waiting`, `awaiting_you`, `answered`, `completed`,
  `cancelled`, `stopped`, `unknown`.
  - An open serving is `working` only on a fresh check, or on the owner's
    own work within 30 min (conversation evidence, shown as such); past that
    it is `unknown`. A check about another ack is not applied; a check older
    than 120 s is `stale` and never animates.
  - The **plan meter** is tasks agreed / current total, with in-progress,
    awaiting-agreement, stopped and no-evidence tasks apart; each `[doc]`
    revision keeps the total it left, so a moved denominator says why.
    Planning shows no number. A run has no determinate unit in any record.
  - **Stages** stay pending until their record: tasks agreed, plan accepted,
    run ended (the finish block), report delivered (`[delivered]` home),
    knowledge refreshed (a `sagesync` for the study's project after the
    research's acceptance, wherever it was recorded). A card is `completed`
    only when every unit of work is finished by record and every stage is.
  - `queue_behind`: an unacknowledged post to an agent that is serving
    something else says what it waits behind — listeners serve one
    conversation at a time, so concurrent requests are not concurrent
    execution.
- **The relay** (`GET /progress[?current=<desk id>]`, agentroom
  `progress.py`): discovery off the mirror (`front-*` conversations; active,
  held or tracked ones, and results of the last 24 h, at most 16 cards), no
  Zulip call; boards reused for 5 s and while the mirror's revision is
  unchanged; health checks with Observer's own `health.toml` command, only
  for open servings of the owners it lists, cached per ack for 15 s, within
  a 5 s budget; Observer's records read as files (its directory is the
  parent of `AGENTROOM_MONITOR_HEALTH`, or `AGENTROOM_OBSERVER_DIR`).
  Nothing on this route writes or starts anything.
- **The trace learned from building it** (pyagag `agag.trace`):
  - a conversation its owner opens and serves itself (Front's
    `routinerun-`) reads `executing`/`awaiting_requester`, never
    `not_started`; the `ag-routinerun` block is the run's end record
    (`finished`/`ended`); `Node.records` carries `doc`, `acceptance`,
    `change`, `delivered`, `sagesync`, `state`, `finish`;
  - **an answer owed to a conversation's owner** (`Node.owed_to`) means that
    owner holds it: a routine run whose task result waits in Front's queue is
    not "held by nobody" (the p1 trial's false `unheld`);
  - **a requester's `[served]` mark anywhere in the request's tree** is its
    receipt: Front took task results up while serving the request's own
    conversation, and the mark there was never read (endless `undelivered`);
  - `silent` is asked about the deepest open unit only; a ✔ on work its
    record calls finished is not `resolved_live` (the owed delivery is
    `undelivered`'s).
- **Records added for it**: archsage's `sage sync`/`attach` inside a
  serving writes `[selfnote][sagesync] <sage> <revision> project=<slug>
  findings=<n>` (since `failsafe` p5 also `for=<conversation>#<anchor>` and
  `includes=`/`missing=<commit>`); `agrun finish` (was `agrunfinish`) ends
  a routine run from the conversation its request was completed in.
- **Known limits**: only autolab is probed (Front, archsage, forge, cagent
  are conversation-only and labelled so). Two limits of p1 are gone since
  `failsafe` p5: an entrusted runner accepts on its own agreement, and a
  study's refresh goes to a topic of the run's own.

## Waiting and finishing studies (`failsafe` p5, 2026-09-27)

Record: `devdocs/episodes/failsafe/p5/`.

- **A queued post is read, not judged by its age** (pyagag `agag.waits`,
  shared by the panel and Observer). A post an agent has not acknowledged,
  or an answer its requester has not taken up, is `behind` (the listener is
  busy with healthy work elsewhere), `blocked` (the serving ahead is not
  healthy), `unserved` (the listener is idle, passed it over, or gave up) or
  `unknown`. For autolab the facts come from its own listener journal
  (`python -m agag.health --queued --since <post time>`,
  `agag.health.probe_queue`: the entry, what runs meanwhile and that
  serving's own health); for others from the open servings every traced
  request shows (conversation evidence, 30 min at most). Observer defers
  `behind`, leaves `blocked` to the blocker's own request for at most
  `escalate_after`, reports `unserved` to the owners at once, and keeps the
  plain `unacknowledged`/`undelivered` rule for `unknown`; an incident on a
  wait later confirmed legitimate closes `excused`. A first post in a topic
  nobody has served belongs to the agent whose `#agents` roster serves it.
- **Acceptance is the decision holder's, whoever records it**
  (`agag.acceptance`; retired with the mission system in `retire/old_autolab`,
  so what follows is history — `agag.trace` still reads such notes). Holders: the mission's requester and everyone up
  its root notes to the request's origin — unless a person reserved it
  (`agentchat reserve --evidence <their post>`, `[selfnote][approval]
  reserved …`), then only them. Evidence: a holder's post after the last
  shown result (`+shown=`), never the mission's own agent's; an agent's
  words count only in the work's own conversations. The note is
  `[selfnote][acceptance] #<post> by <user> (<name>) after=#<shown>`.
  Front, entrusted by a routine's guide, records its own agreement.
- **A refresh is bound to its run.** Study guides ask archsage in a topic
  of the run's own, addressed to archsage itself (a post opening with
  `sage:<name>` goes to the sage), naming the integrated commit; `archsage
  sage sync <name> --require <commit>` records `for=` and
  `includes=`/`missing=`; the panel's `knowledge_refreshed` counts a
  refresh by that relation and revision, never by project and time.
- **Tools refuse the misroutes seen**: `agentchat send` refuses a topic
  whose answers would return to another request's conversation, and a topic
  spelled `<channel>/<topic>` naming a real channel; `agentchat recheck`
  says UNOWNED where no agent ever served, and ASKED for the queued post
  itself; `--to` suggests the account a name is part of.

## Completion, receipts and holds (`failsafe` p6, 2026-09-28)

Record: `devdocs/episodes/failsafe/p6/`.

- **Separate facts, one rule** (`agag.trace`, the same call for the panel,
  Observer and `agentchat trace`). A unit's execution record, the
  requester's decision, each answer's receipt and open questions are kept
  apart:
  - an answer without a receipt is owed (`awaiting_delivery`), whatever the
    producer's record says;
  - it is **settled** when a decision recorded after it covers it
    (`trace.decisions`):
    - the requester's `[state] accepted`/`done` in the unit;
    - `[change] accepted … +shown=<id>` (up to the shown result, not the
      close-out after it);
    - the mission's `[acceptance] … after=#<shown>`;
    - a cancellation above it.
  - Then the unit reads as its record does, and `Node.receipt` keeps the
    missing receipt as `settled` bookkeeping (a card's `settled_receipts`).
  - Speech at home is no receipt for anybody: Observer's old
    `receipts_from` leniency is gone.
  - Every answer above the served mark is checked, newest first, not only
    the newest.
  - A unit's requesters are all authors of its root notes, so the answer is
    the same from the mission or the request.
- **A citation adopts nothing**. Since `failsafe` p6 ex1 this is a recorded
  relation, not p6's first-post rule (`_reference`, gone): see *Work
  relations and dispositions*.
- **Receipts** (`agag.receipt`, `agentchat receipt <answer> [--repair]
  [--because <post>]`). It inspects:
  - the answer, and whether it names the caller;
  - the caller's home (`rootchat_home`);
  - the receipt: RECEIVED, RECONCILED, MISSING or NOT_OWED;
  - the evidence: the caller's listener journal (`AGENTCHAT_JOURNAL`, set
    for every run), a covering decision, or the caller's own later post.

  `--repair` writes one note into home:
  - with journal evidence, the listener's `[served]` mark — only when no
    earlier post naming the caller would be covered without having been
    given;
  - otherwise `[selfnote][receipt] #<answer> by #<evidence> (<why>) in
    <remote>`, which covers exactly that answer and claims no serving.

  Nothing is served, and a repeat writes nothing. The listener skips
  reconciled answers. Only the owed agent's own receipts count.
- **A trial fault**: `<instance .local>/faults/exit-before-receipt` (one
  shot, created only by a person) ends a listener right after a confirmed
  delivery, before its receipts. The restart writes them, with no rerun and
  no second report.
- **Holds** (`agag.holds`, `agentchat hold`/`release`, operator
  `python -m agobserver.hold`) are records in the request's origin
  conversation:
  - `[selfnote][hold] <acceptance|resume|decision|indefinite> a<unit> by
    <user> (<name>) #<their post> — <why>` and `[selfnote][hold-release]
    #<hold> …`;
  - `acceptance` settles with the unit's acceptance or cancellation;
    `resume` when its owner serves it again after the hold, or it ends by
    record; `decision`/`indefinite` only on the holder's words;
  - unrelated activity settles nothing;
  - a hold in force covers only its unit and what is below it, says what it
    waits for, and keeps its history.

  The panel shows it as `awaiting_you` with that text, and Observer leaves
  only the covered work alone. `held.json` is gone, and since p6 ex1 so is
  `retired.json` (dispositions, below).

## Work relations and dispositions (`failsafe` p6 ex1, 2026-09-28)

Record: `devdocs/episodes/failsafe/p6/ex1/`.

- **A root note says what its conversation is to its home** (`agag.relations`).
  `[selfnote][rootchat] <channel>/<topic> #<anchor> rel=work|reference`:
  - the note's home is always the **return address** (callbacks follow it,
    whatever the relation);
  - `work` (a delegation) or a deliberate move (`[rootchat-moved]`,
    `agentchat anchor`, `agrun adopt`) **adopts**: the conversation, its
    waits, acceptance holders and receipt decisions belong to the home's
    request;
  - `reference` (a comment, a citation) adopts nothing; the home lists it
    ("cites …");
  - no word and no record: **unknown** — adopts nothing, listed on the trace
    and the card with the command that resolves it.
  - Records: the author's correction `[selfnote][relation] #<note>
    work|reference — why` (newest wins), and one legacy record per author,
    `[selfnote][relation] legacy upto=#<id> reference=#… unknown=#…`, which
    classifies that author's older notes (written 2026-09-28 as #15450–#15454
    with `python -m agag.relations legacy --mirror <copy> --channel <own>
    --apply`, each with its author's credential).
  - No reader decides a relation from how much history it read.
- **Writers.** Every writer that opens a conversation for work states `work`
  (`rootchat_note`'s default). `agentchat send` reads a conversation's
  **beginning** oldest-first (`ZulipClient.topic_beginning`): work for a new
  conversation, one the sender began, or one opened for work; reference for
  one that began as somebody else's request (and says so); unknown when the
  beginning cannot be read. `--relation work|reference` says it; on a
  conversation with the sender's note already, it records a correction.
  `agentchat relation <channel> <topic> [work|reference]` shows and corrects.
- **Dispositions** (`agag.dispositions`) are decisions about a request's
  standing, records in its origin conversation beside its holds:
  `[selfnote][disposition] suppressed|completed|cancelled|withdrawn a<unit>
  upto=#<id> by <user> (<name>) #<evidence> — why`, and
  `[selfnote][disposition-reversed] #<disposition> …`.
  - `suppressed`: monitoring suppressed — the work stays open and visible
    (the card says why nobody chases it); Observer does not act on it.
  - `completed` / `cancelled` / `withdrawn`: the request (or unit) ended. The
    unit reads done or cancelled; unfinished work below it reads cancelled
    ("ended with …; its own record: …") and is listed as remaining — never
    completed. A missing receipt under it is settled bookkeeping.
  - The scope is the unit and what is below it; `upto` is the newest
    **substantive** post (speech, not an ack) there at the decision. A later
    substantive post uncovers the conversation it lands in (new obligations
    are shown and monitored); selfnotes, ✔, restarts, and the recorder's own
    reply closing the serving that recorded it change nothing.
  - Tools: `agentchat disposition [<msg>] [<kind>|reversed] --evidence
    <post> [--unit <anchor>] why`, operator `python -m agobserver.disposition`
    (`--list`, `--reverse`, `--by` in person). A repeat writes nothing.
  - Applied once by the trace, so the panel, Observer and `agentchat trace`
    agree; the relay and Observer discover requests with a disposition
    record like held ones.

## A reply that claims an act it never did (`failsafe` p7, 2026-09-29)

Record: `devdocs/episodes/failsafe/p7/` (report1 is the contract).

In agent_guide p2's live hold trial, Front's run-0183 answered "I release
the hold; stop following this request" in one turn with no tool call:
"Released hold #15837 … Recorded the disposition as `withdrawn`". Nothing
was recorded. Observer saw only the records, a hold in force, and would
never have asked. The Omni Agent noticed by hand (#15847). The fixture
recorded both acts 12 of 12 times: the event is rare and model-side, and
no guide sentence could be shown to help. Since p7 the listener checks
every delivered reply (pyagag `agag.claims`, in `agag.listen`, so the same
check runs for Front, autolab, forge, archsage, Observer and cagent):

- **A serving's records are its window.** A listener serves one
  conversation at a time, and a run's tools post with the agent's own
  credential. So every post the agent's bot made between the serving's ack
  (or its start, on the mention route) and its reply belongs to that
  serving, whatever the harness and wherever the post went. The window is
  read off the listener's mirror (no transcript, no Zulip call). Memo
  channels and the listener's own lines are left out.
- **The acts and their records**:

  | act | record |
  |---|---|
  | `hold` | `[hold]` |
  | `release` | `[hold-release]` |
  | `disposition` | `[disposition]`, `[disposition-reversed]` |
  | `relation` | `[relation]`, or a `rootchat` note carrying `rel=` |
  | `accept` | `[acceptance]`, `[state] accepted\|done` |
  | `reserve` | `[approval]` |
  | `receipt` | `[receipt]`, or a `[served]` note written by the run |
  | `send` | a post in another conversation |

  The check looks at who posted a record, and at its kind and target. A
  release on the Omni Agent's words is Front's own `[hold-release] … by 9
  … for 8` post, and counts.
- **A reader lists what the reply claims; code judges it.**
  - The reader is the host's local model (`~/.config/agag/claims.toml`: a
    `[reader]` table with `url`, `model` and `timeout`; today
    `qwen3.8:27b-mxfp8`, the model Observer keeps loaded). It gets the
    run's own reply words: the reply mark's text, without the mention or
    the `ag-post` line, which a reader once took for a send.
  - It returns `{act, target, where, quote}`.
  - A claim is true when the window holds a record of that kind, or when
    one is already on record: one naming the target (always required for
    `release` and `receipt`), one in the target's conversation, or, with
    no target, one in the reply's own conversation. So a restatement
    ("the hold #… is still in force", "recorded earlier") is no alarm.
  - A reader that fails, or no reader configured, leaves the serving
    `unchecked`, never clean. It is tried again while the executor is idle.
    A failure line or an empty reply is not read.
  - The outcome is kept in the serving record (`extra.claims`), and the log
    line is `claims of #<reply>: …`.
- **A mismatch buys one serving of its owner.**
  - The listener writes `[selfnote][claim] {json}` (reply, what is missing
    with the quote, what was found, attempt) into the reply's
    conversation. Beside it goes the owner's own `[selfnote][start] #<claim>
    for <requester>`, so the notice, not the stale decision, is the
    trigger.
  - Every conversational prompt carries the open claims
    (`prompt_with_guide(reply=True)`): the quote, the record missing, the
    tool that makes it, and "do it now, or say plainly the earlier reply
    was wrong". No guide text was added.
  - The serving that answers settles the claim:
    `[selfnote][claim-settled] #<claim> recorded #<reply>` when the records
    now exist, and `corrected` when the reply no longer claims them.
  - Claiming it again is attempt 2: no start note and no third serving.
    The trace's `claim` candidate, 60 s after an escalated claim or 600 s
    after an unanswered attempt 1, reaches Observer, which reports it to
    the owners at once and asks nobody (the `unanswered` rule). The
    incident recovers only on the settlement record.
  - A claim in a conversation its listener does not serve by itself gets
    no start note; the candidate covers it.
- **Shown as open until settled**:
  - `agentchat trace` prints `! claim #…: reply #… says release #…,
    disposition — no record (…)` and lists it under "owed now";
  - a progress card with an open claim reads `waiting`, with the claim as
    its reason (`claims` on the unit and the card).
- **Tools**:
  - `python -m agag.claims <reply> --mirror <copy> [--after <ack>]` checks
    one posted reply and writes nothing;
  - `python -m agag.claims --settle <claim> <why>` closes a claim a person
    decided to leave (`dismissed`);
  - `python -m agag.fixture reader [--runs 4] [--model m] [--url u]` measures
    the reader on 13 fixed cases (`agag/fixture/reader_cases.json`) with the
    listener's own prompt and `reply_words`: right readings and seconds per
    reading. Run it before switching `claims.toml` to another model; today's
    model reads 44 of 52 (the two over-readings the judge absorbs), 0.3–3.7 s
    (agent_guide p3).
- **Trial aids**:
  - agfront `faults/false-claim` (one shot, created only by a person): the
    next desk/front serving posts the file's reply and runs no model;
  - `agfront.trial hold-release --replies <file>…` replays fixed outputs
    through the real serving, then the model. The fixture's claims probe
    runs the same check on an overlay that keeps the run's writes.
- **Limits.**
  - Detection is only as good as the reader: a claim it does not extract
    is missed (pinned by a test).
  - Acts that leave no record in the realm (files, commits) are not
    checked.
  - autolab's task replies are literal sections, not model replies, and
    are not read.
  - Observer reports only claims inside requests that came through
    `#front`.

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

## How a guide is put together (`agent_guide` p1, 2026-09-28)

A conversational role's prompt is the placement lines and the conversation,
then its instruction, then the two sections every conversational role gets
from pyagag (`agag.reply` "How your reply is posted", `agag.continuation`
"Carrying the conversation forward"). In agfront the instruction is
`zulip_listener.role_guide(role)`: the role's own
`agent/guides/<role>/guide.md`, then the files `SHARED_GUIDES` names from
`agent/guides/shared/`:

- `board.md` (desk, front, routine_run, argue): what the developer assumes
  (every name on the board is yours to look up), that the board is Zulip
  reached by `agentchat` and not the filesystem, the `--help` index, that
  reading is free and a post runs its addressee, the working directory, how
  a post reads in the chatlog;
- `requests.md` (desk, front): what the developer asks for and whose work
  each kind is: work, argues, projects, studies, routines, references;
- `work.md` (desk, front, routine_run): each serving ends; judge on
  evidence; whose decision it is (proxy authority, acceptance, holds,
  dispositions, receipts); what to do when Observer says work has stopped.

A role's own guide is a short head (who you are, who you speak with) plus
what only that role does (routine_run's finish block, argue's `ag-argue`
blocks). **Tool usage lives in the tool's `--help`, not in a guide.**
`agentchat --help` is an index: the board, then one line per command. Each
`agentchat <command> --help`, `agproject`, `agrun`, `agrefs` and `agbudget`
says what it prints, what that means and when an agent wants it. A guide
names the tool and the fact the tool cannot hold. Other agents' guides point
at `agrefs --help` in one line, instead of copying its manual.

Every paragraph that exists because of a seen-live failure or a named trial
keeps its trial reference wherever it moved: a help text, a shared file, or
a role's guide. `devdocs/episodes/agent_guide/p1/report1.md` is the
inventory of where each came from, and `report.md` lists what was dropped
as Anxiety-Driven.

Guides are read from disk per serving, so an edit is live at the next post.
A change that also needs new code (`SHARED_GUIDES`) or a new help text (a
pyagag pin) is prepared on a branch and merged at deployment, followed at
once by the pin, `uv sync` and a listener restart.

### Text several agents share (`agent_guide` p2, 2026-09-28)

Text that more than one **agent** needs is written once in pyagag, as
package data in `agag/guides/*.md`, not copied into each agent's guides:

| file | what it says | who gets it |
|---|---|---|
| `board.md` | the board is Zulip reached by `agentchat`, what is on it, `agentchat --help` is the index, reading is free and a post runs its addressee (a question the person you serve asks you to put to a named agent is the request, not a poll), ✔ is finished (a question about it goes where the introduction says; agent_guide p3) | roles that hold `agentchat` and read the board: Front's desk/front/routine_run/argue, archsage, autolab's planner and worker |
| `callback.md` | each serving ends; ask with `send`, reply, finish; the callback brings the answer back | roles that delegate: Front's desk/front/routine_run, archsage, autolab's worker |
| `refs.md` | the `agrefs` pointer | every role that holds `agrefs`, argue participants by default |
| `entrance.md`, `entrance_default.md` | the entrance's fixed half, and the default vocabulary | every entrance (`agag.entrance`); an agent's `entrance_front/guide.md` holds only its vocabulary (autolab's does; forge has none) |
| `argue_participant.md` | what every argue participant is told | `agag.argue.participant_prompt` |

The agent's code names the sections per role:
`prompt_with_guide(lines, guide, reply=…, shared=("board", "callback", "refs"))`
appends them right after the role's own guide (always in that order),
before the reply and continuation sections; `shared_sections(names)` is
the same text for a composition that does not go through it (forge's
generators, cagent's operator, autolab's review serving);
`participate(…, shared=…)` takes a tuple or a function of the invitation
(archsage gives its sages none: they hold only `sagetree`). An unknown
name or a missing file is a `GuideError`. The choice follows the role's
grant: a section may only point at tools the role holds.

agfront's own `shared/` files keep what is true of Front alone: the
developer's assumption, Front's working directory and chatlog format
(`board.md`), what the developer asks for (`requests.md`), and whose
decision it is and what to do when Observer says work stopped (`work.md`).

The check that nothing is repeated across agents, over every guide of
every agent plus pyagag's files, is `cat … | awk 'length>40' | sort | uniq
-c | awk '$1>1'`; it printed nothing after p2.

### What a run reads the board through

A run's look-only commands — `agentchat read/topics/channels/intro/
options/trace` and `agproject status` — read the mirror store named by
`AGENTCHAT_MIRROR`, which `agag.agent.chat_environment` sets to the
listener's own `<instance>/.local/mirror/mirror.sqlite`
(`agag.mirror.reads.MirrorReads`, opened read-only). What the store cannot
answer (a private channel, an unread archived topic, users, every write,
and `recheck`) goes to Zulip. `agentchat read` takes several topics of a
channel, or `--latest N [--prefix p]`, in one call; `topics` takes
`--prefix`.

### Trying a guide change against the same board (`agent_guide` p2, p2 ex1, p3 ex1, p3 ex2)

**The board.** `python -m agag.fixture build <dir>` writes a synthetic
board as a mirror store marked `fixture`. It is built in code; nothing is
exported from the realm. A run pointed at it reads only it and can post
nowhere. `python -m agag.fixture probes` lists every probe, its pass rule
and the line that runs it.

**The board records what it says, and says which board it is**
(`agent_guide` p3 ex1).

- A mission is written as a live one is (m15750 was the template): its name
  is its `[mission]` note's id, and a finished one carries its tasks'
  `[task]`/`[rootchat]`/`[start]`, the shown result, the requester's
  agreement, `[change] accepted … +shown=`, `[state] completed`, ✔, then
  the acceptance record (`[state] accepted`, `[acceptance] … after=#`,
  `[state] done`, ✔). Routine runs end with their `ag-routinerun` block.
  Board 1 said "m20390 is done: accepted by Front" over a mission `trace`
  read as `queued`, and one run believed the trace.
- `python -m agag.fixture consistency <dir>` checks the built board: every
  mission named has its note, one called done traces done, a ✔ task traces
  done, every agent named has an introduction. It is also a test; board 1
  fails it on every mission.
- `agag.fixture.board.BOARD_VERSION` (2 since p3 ex1; 1 before) is written
  into the store and, from the store a trial actually read, into
  `outcome.json` as `board_version`. Board 2 renamed growbox's and
  ProtoPrey's missions (m20390→m20420, m20396→m20460, m20402→m20510,
  m20410→m20550, m20455→m20600); `board.EARLIER_NAMES` keeps the old
  names.

**The drivers.** Each agent's driver is in its own package. You run it from
that agent's checkout with its own venv:

```
.venv/bin/python -m agfront.trial | archsage.trial | agautolab.trial | agobserver.trial \
    <probe> --out <dir> [--store <mirror.sqlite>] [--guides <tree> | --guides-rev <commit>] \
    [--no-shared] [--dry-run] [--records <dir>]
```

They share pyagag's kit (`agag.fixture.run`). A trial builds its own board
under `--out` unless `--store` names one. `--dry-run` replaces the harness:
the prompt is written to `<out>/prompt.md` and no model runs. The outcome
goes to `<out>/outcome.json` and `reply.md`: the reply, the verdict,
required and observed facts, cost, turns, tool calls, and which guides
and board were used.

**A baseline is a commit.** `--guides-rev <commit>` extracts the
checkout's `agent/guides` at that commit (`git archive`, into `<out>`).
`--no-shared` leaves out pyagag's shared sections. For Front:

- p1's composition: `--guides-rev 447bb03 --no-shared`;
- before p1: `--guides-rev ba28e90^ --no-shared`.

For the other agents, before p2: archsage `df6e0af`, autolab `aab6810`.

**A trial touches nothing real.**

- The kit sets `AGAG_RECORDS_ROOT`, so every `AgentSpec` puts serving
  workspaces and run records under `<out>/records/<agent>/.local/`
  (`--records` to choose). The relay's cost read (history since
  retire_old_ui p1; the in-flight view is retired) reads only the agents' checkouts, so trial spend is never counted live. Each
  driver has a test that a dry run from the main checkout leaves that
  checkout's `.local/agent` and `.local/topics` as they were.
- `agproject status` on a fixture reads the fixture's own repositories
  (`gitea.fixture.invalid`), never the host's Gitea.
- A trial run's `AGENTCHAT_JOURNAL` is beside the board, not the live
  listener's.

**A delegation can be tried.** A probe with a responder script
(`agag.fixture.responder`) is served on an overlay, the trial's own copy
of the board:

- `agentchat send` there is recorded, and a post takes a scripted agent's
  next canned line only through that agent's **doors**, where its live
  listener would serve it (`responder.DOORS`, `agent_guide` p3 ex2):
  - autolab (`AUTOLAB_DOORS`) answers in `autolab-agstudio1` and in a
    mission's own `workplan-`/`workrun-` topics already on the board;
  - a **new** `workplan-` topic in a `pj-` channel is a new request. It
    gets a new-mission plan acknowledgement and no script line, and nothing
    scripted answers there again;
  - a mention elsewhere, a `workplan-` topic outside a project channel,
    and a `to=` alone reach nobody;
  - an agent not in `DOORS` is served by a mention or in its own channel;
  - each send's door is in `outcome.json` (`doors`), and `python -m
    agag.fixture doors <out>…` replays saved runs through today's doors;
- the line is posted after the serving ends, and the conversation is served
  again with the calling topic beside it, as the listener serves a callback
  (`Trial.converse`);
- a script line is `Canned(agent, text)`, where `{ask}` is the
  post it answers and `{asker}` the asker's mention: a plain answer, or a
  `response_request … ask=decision` back to the asker;
- every other write stays refused, so the board is the same between runs;
- rules judge the last serving's reply, every serving's tool calls, the
  posts sent (`sends_must`) and the number of servings.

The two probes are `delegate-answer` and `delegate-decision` (Front).
Results are in `agent_guide/p2/ex1/report.md`; the three-arm comparison
(before p1 / p1 / now) with the delegation fix in `agent_guide/p3/`; the
rates on board 2 (12 runs per arm, Wilson intervals, fd-wr attempts
counted over every run) in `agent_guide/p3/ex1/`; and the three arms
again on board 2 with the doors (72 runs per arm) in `agent_guide/p3/ex2/`.
Before the doors, a delegation that opened a new `workplan-` topic passed.
Under the doors it gets the acknowledgement, and in p3 ex2 all four such
runs answered it "start".
`guard-status` (as9: a status question about running work, nothing sent)
and `guard-finished` (fd-wr: no send into a ✔ topic's bare name; the rule
`sends_to_must_not` judges sends, not reads) keep that fix from undoing
the lessons it scoped.

**Judging again costs nothing** (`agent_guide` p3 ex1). An
`outcome.json` keeps what a rule reads (the last reply, every tool call,
the sends, the servings), and every verdict names its rule (`rule`, a
digest). `python -m agag.fixture rejudge <out-dir>… [--json <file>]`
applies today's rules to saved results without a model:

- a result from an earlier board is judged in that board's mission names
  (`probes.for_board`);
- one judged by another question (the first `delegate-answer`) is listed as
  `other rule` and left out of the counts;
- dry runs are skipped.

A rule change must be a spelling, not a changed standard: read the passes
it adds. p3 ex1 added "12 時間", named a study by any of its places, and
narrowed "教えてください" to asking which or what study is meant
(`ASKS_WHAT_THE_STUDY_IS`): p3's 108 runs went 86 → 90, exactly the four
passes in substance p3 named.

**A batch is one command** (`agent_guide` p3 ex2). The queue, the
classifier and the table live in `agag.fixture`, not in ignored folders:

```
python -m agag.fixture batch <plan.toml> --out <dir> [--jobs 2] [--probe p] [--list] [--dry-run]
python -m agag.fixture classify <dir>          # per run: verdict, delegation class, fd-wr, search, as9, unasked send
python -m agag.fixture table <dir> [--json f]  # pass counts with Wilson 95 %, process measures per arm
```

- The plan (TOML) holds:
  - `[arms]`: name → driver flags;
  - `[[probes]]`: `names`, `runs`, optionally `arms`;
  - `[drivers.<agent>] dir`;
  - `[budget]`: `harness`, `limits`, `poll_seconds`.

  `agag.fixture.batch`'s docstring has an example, and p3 ex2's
  `report2.md` has the three-arm plan.
- Jobs are interleaved per run number with the arms rotated. A job whose
  `outcome.json` exists is skipped, so the same command resumes.
- **The budget gate.** Trials spend the account the live agents use: in p3
  ex1, 4 parallel trials exhausted the 5-hour window. Before each job the
  gate reads `agbudget --json` and starts nothing while a limited window
  (e.g. `session = 60`, `weekly_all = 90`) is at its limit, or while the
  reading fails. The usage endpoint's 429s show up as short pauses.
  Pauses and readings go to `<dir>/batch.log` and `budget.jsonl`.
- A reply that is the harness's limit message is **cut**, not judged. It
  is kept under `<dir>/cut/` and queued again.
- p3 ex2's 216 runs (3 arms) took 97 minutes at `--jobs 2`, for $54.86,
  0 cut, and moved the session window 20 → 56 % and the weekly window
  81 → 86 %.

**A trial's filesystem is the host's.** Runs sometimes grep the projects
tree for a mission's name (p3 ex2: 8 of 12 p1 guard-status runs), and that
reaches other trials' outcomes and pyagag's pass rules. No reply used what
it found, but keep batch output where that matters to you.

**A shared-section change can be tried before it is released.**
`--shared-guides <dir>` reads pyagag's shared sections (`<dir>/<name>.md`)
instead of the installed package; with `--guides <tree>` a whole candidate
composition runs while the live agents keep reading their checkouts
(agent_guide p3 tried its fix this way). Judge the verdict *and* the calls:
a rule's spellings can miss correct answers (p3's four, since held), and
a delegation fails three different ways (nothing
sent, sent where no listener is served, or proposed and waited), which
only the calls tell apart.

**A probe can run the listener's claim check** (`failsafe` p7,
`Probe.claims`; `hold-release` does). It is served on an overlay that
keeps the records the run writes, and each reply is checked as the
listener checks it (`agag.claims.check_served`, with the host's reader).
A mismatch is written there and the conversation is served again with the
notice. The outcome carries `claims` per serving and `claims_final`.
`--replies <file>…` makes each serving's run answer the next file's text
instead of a model (its prompt goes to `<out>/prompt-<n>.md`), then lets
the model run. That is how run-0183 is reproduced exactly.

## Front (managed project `front`, project_migration p8, 2026-10-19)

**Front is the Developer's conversational entrance, and an ordinary managed
project.** `developer/pj-front`, workspace `front`, agent `front-agent`
(profile `sonnet`). It understands a request, works with the projects that
do the work (creating one when none serves), performs what agdevworld offers
on the Developer's behalf, and remembers preferences and coordination
lessons. Episode: `devdocs/episodes/project_migration/p8/`.

- **The entrance.** The Developer talks with Front in aghangout
  (`http://<host>:8093/hangout/?with=front`) or from Slack. Both are
  clients of WFE conversations (`api/conversations`; agdevworld
  `workflow_editor/docs/conversations.md`; `wfe conversation help`). Since
  workflow_editor p15 the Developer can also talk with forge and archsage
  directly; Front is one destination among several (*Conversations with
  project agents* below).
  - Each message of the person starts one bounded run of `front/converse`,
    by receipt, so a replay or restart starts nothing twice.
  - That run reads the conversation and its linked runs, and records what the
    message is to them (`new`, `follow-up`, `steer`, `answer`, …). It then
    answers, works, steers or asks.
  - Replies name their run and what they answer.
  - Zulip is not on the Developer's path.
- **Its work** is the run's own graph: delegate nodes added by adjustment
  entrust work to other projects' workflows; their questions and results
  continue the run.
  - It can answer a specialist itself when the request and preferences
    settle the question.
  - Other runs are steered with `wfe run instruct`. An instruction continues
    a run nobody executes.
  - From its attempt, Front records as itself (`operator`); the Developer's
    words stay theirs (`--from`, `--source-message`).
  - Agent requests: `wfe request --request`.
  - Projects, workflows and context: `wfe create`,
    `wfe workflow|project show|save --base`, `wfe context`.
- **Responsiveness**: a worker lane. The coordinator's worker has an extra
  `conversation` slot (`workers.json` lanes; `execution.lane`), so a turn is
  served while specialist work holds the default slot. It was measured at
  18 s with a cagent child running.
- **Knowledge**:
  - system context block `developer-preferences` and project blocks of the
    same id (managed by front-agent);
  - coordination lessons in `pj-front/knowledge/`;
  - run state only in run records.
- **Delivery links**: a run's file opens in the browser at
  `…/wfe/api/projects/<p>/runs/<wf>/<run>/raw?path=…`.
- Known limits:
  - one conversation slot, shared with forge's and archsage's turns
    (workflow_editor p15), which long work done inside a turn also holds;
  - a shell process left behind no longer keeps an attempt open once the
    session wrote its final result (p8 ex1); one left while the session
    still works does;
  - definitions Front edits are saved, not committed.

## The agdev site (`retire_old_ui` p1, `workflow_editor` p15, 2026-10-20)

**Home is the agdev dashboard** (`http://<host>:8093/wfe/`). A bare
`http://<host>:8093/`, any other path, and `:8090/` redirect there. Home
shows:

- projects and their runs, linked;
- execution nodes and what each runs;
- Context;
- Usage: subscription windows, and what workflow executions used with its
  coverage (agdevworld `workflow_editor/docs/usage.md`);
- User nodes;
- a link to aghangout.

agdevworld serves no screen of its own any more. The embedded Front Desk and
Arguing Room were retired in workflow_editor p15. An old Front Desk link
(`/?view=frontdesk&conv=<id>`) opens that conversation in aghangout. The
relay (`agentroom`, `:8094`) keeps only `/budget` and `/cost` for Usage, and
holds no credential. The world views, the operation room and ops board, the
routine dashboard and the cost gauge page were retired in retire_old_ui p1.
Their history stays in the episodes.

## Conversations with project agents (`workflow_editor` p15, 2026-10-20)

*Since challenge p2 a direct chat is a one-to-one room of the one
conversation model. The destination is now the default addressee. See
**One conversation (`challenge` p2)**.*

**A person talks with a project's agent directly, in a conversation whose
destination is that project's conversational workflow.** Episode:
`devdocs/episodes/workflow_editor/p15/`. Contract: agdevworld
`workflow_editor/docs/conversations.md`.

- **WFE owns conversations and execution.**
  - Each conversation records its destination (`front/converse`,
    `forge/converse`, `archsage/converse`). The destination is named by the
    first message's `target`, must be a configured target
    (`<area>/conversations.json`), and never changes.
  - Every message starts one run of the destination workflow, by receipt.
  - A destination that cannot take a turn leaves the message `failed` with
    the reason. It is retried there, never sent elsewhere.
  - `GET api/conversations` lists the targets and their availability.
  - The view gives each linked run's status, run view, file route and
    serving review pages.
- **aghangout** (managed project `aghangout`, Gitea
  `developer/pj-aghangout`, agent `aghangout-agent`) is the place to meet
  the agents: `http://<host>:8093/hangout/`.
  - It is a visual novel: a lobby of the partners, then each partner's room,
    portrait and dialogue, with History, Work and Talks panels.
  - It owns characters, rooms, playback, navigation, input, drafts and
    outbox. WFE owns everything else.
  - Its own `scripts/deploy.sh` installs releases into
    `<area>/hangout/releases/<commit>` and switches `current`. The agdev
    origin serves `current` read-only, so a redeploy needs nothing from
    agdevworld. aghangout-agent made and deployed a change this way in
    `change/run-001`.
  - Nautobot: service `aghangout`, profile `aghangout_web` (an HTTP check of
    `/hangout/`).
- **Other clients.**
  - Slack names `front/converse` for new DMs; its links open aghangout.
  - A future client (XR, …) uses the same contract.
- **The agents.**
  - `front/converse`: coordination across projects.
  - `forge/converse` and `archsage/converse` (each with its `converse.md`):
    answer within their role, and entrust real work to their own workflows
    (`provide`, `consult`, `study`, …) with a delegate node, which frees the
    conversation slot.
  - Shared facts for every conversational run are in the agautolab wfrun
    guide, *A run that serves a conversation message*:
    - replies name `--re`;
    - questions are posted in the conversation too;
    - a child's question is asked as the run's own question from its
      delegate node;
    - the person's later words go to the running work with
      `wfe run instruct|answer --from --source-message`.
- **Responsiveness.**
  - The three agents share the coordinator's one `conversation` slot. A
    turn waits only behind another turn.
  - Measured: Front 12 s while specialist work held the default slot;
    archsage 44 s when queued behind Front's turn.
  - Long work inside a turn delays everyone (forge's inline icon took 81 s).
- **Known limits:**
  - one conversation slot (the executor runs one claim loop per lane);
  - a message to a destination that stays unavailable is retried by every
    reconcile until the destination exists;
  - agents sometimes paraphrase the person's words when recording an answer
    (the source message reference stays exact).

## Discussions (`workflow_editor` p16, 2026-10-20)

*Since challenge p2 a discussion is a conversation that ponderer leads. The
invitation is the common response request, and `api/discussions` is gone.
See **One conversation (`challenge` p2)**.*

**A discussion is a person, a facilitator and project agents with their own
missions and contexts in one shared conversation.** Since argue p4 the
facilitator is ponderer's conductor run; since argue p4 ex1 ponderer also
keeps the records and the argue project is retired (*ponderer conducts,
keeps and learns from discussions* below). Episode: `devdocs/episodes/workflow_editor/p16/`. Contract:
agdevworld `workflow_editor/docs/discussions.md` (`wfe discussion help`).

- **WFE owns delivery and execution.** A discussion is the conversation
  store's second kind (store v5): members with a join/leave history,
  verbatim messages each under the participant WFE recorded (with agent,
  run, attempt and invitation), and **invitations — the only thing that
  starts a run** (receipt `discussion:<id>:invitation:<n>`; horizon;
  outcome `answered`/`unanswered`/`failed`/`withdrawn`/late).
  - The person's message invites whom it addresses; addressing nobody
    invites the facilitator. A reply naming a participant's question goes to
    **the run that asked** (as the answer to its WFE question).
  - Only the person and the facilitator invite; a contribution starts
    nothing. The facilitator's invitations are awaited: its node waits, and
    one instruction continues it when all are resolved.
  - Close withdraws what has not started, continues nobody and cancels
    nothing; reopening replays nothing. Unavailable participants fail
    visibly and block nobody.
- **Participants** (as configured in p16; now see the ponderer section):
  `argue/facilitate` (facilitator), `forge/discuss`,
  `archsage/discuss`, `aghangout/discuss`, and since argue p3
  `front/discuss`, `rts-vs-bot/discuss` (agent `rts-vs-bot-agent`) and
  `jevlab/discuss` (`conversations.json` `discussions`). Front's
  conversation-only facts live in `pj-front` `agent/converse.md`. Shared facts for every participant: agautolab wfrun
  guide, *A run that speaks in a discussion*. Real work is the project's
  ordinary workflow (forge's `discuss` entrusts `provide` by a delegate
  node and presents the result).
- **argue** (retired in argue p4 ex1; its records are in pj-ponderer,
  `HISTORY.md` there) was a managed project (Gitea `developer/pj-argue`,
  agent `argue-agent`): `discussions/<id>/{manifest.json,transcript.jsonl}` (the
  archive), `summary.md` and `knowledge/` (its runs' interpretation),
  `tools/argue` (list, show, validate, cite). Workflows `facilitate`
  (conversation lane), `synthesize` (summary, knowledge with *Revisions*,
  hand-off as an ordinary request with a pinned `argue@<commit>:…` source,
  linked back as `handoff`), `freework`.
- **The archive has one writer**: WFE's `DiscussionArchive`, in every
  executor reconcile and `wfe discussion archive`, in its own clone under a
  lock, into the configured archive project's repository (pj-ponderer since
  argue p4 ex1; each publication record names the repository it was
  verified in, store v8). It appends from the **published** manifest's horizon (the
  repository is the truth), so an interrupted export never duplicates;
  status (`current`/`behind`/`failed`) is in the view, aghangout's Records
  and Home.
- **Clients**: aghangout meeting rooms (`/hangout/?discussion=<id>`; since
  argue p4 ex2 overview-first with the verbatim conversation, the members at
  the table behind 再生表示, To:, Progress, Invitations, Members, Records,
  Details); WFE Home's *Discussions* card (ordinary discussions).
- **Measured**: a brief reached a two-project synthesis in 2 min 40 s on one
  `conversation` slot; a second participant queued up to 68 s; turns ran
  at once beside a `provide` run in the default lane. Capacity unchanged.
- **Commissioned work returns to review (argue p3, 2026-10-20).** The
  facilitator can await the result of a run already working (`wfe
  discussion await --run … --note …`; store v6 `invitations.awaits`):
  answered by that run's next `result`/`synthesis`, or settled when it
  ends. What a facilitator run awaited after it *completed* goes to one
  new facilitator turn (receipt-bound), so a finished turn no longer drops
  results. Contract: `docs/discussions.md` *Awaiting a result*. Episode:
  `devdocs/episodes/argue/p3/`. The trial itself (one RTS × Jev match,
  reviewed in the same discussion, 11 min 36 s) needed no await: the
  participants executed inside their turns, against their guides.
- **Known limits**: participants answer one after another on the one
  conversation slot; delegation answers still travel through Zulip (53 s
  seen across a restart); a requested `synthesize` run reports to the
  discussion only if it says so itself (guide since p16 step 4).

## ponderer conducts, keeps and learns from discussions (`argue` p4, p4 ex1, 2026-10-21)

**Since argue p4 ex1 ponderer owns discussions end to end: progression,
records, evaluation and the decision to change its practice.** argue is
retired. Episode: `devdocs/episodes/argue/p4/ex1/`.

- **pj-ponderer**:
  - `discussions/<id>/` holds the archive (WFE writes it) and
    `summary.md`;
  - `episodes/` holds evaluations and the comparison index
    (`ag.argue.episodes.v1`);
  - `tools/discussions --help` lists, shows, validates and cites
    discussions, and shows the pinned wedo;
  - `HISTORY.md` records the import of argue's history: pj-argue `main`
    merged without rewriting, run branches as tags
    `history/argue/wfe/argue/…`. `argue@<commit>:…` citations resolve
    in pj-ponderer.
- **Workflows**:
  - `conduct` (conversation lane);
  - `synthesize`: a summary, requested by the conductor through an
    ordinary delegate node. A participant is a workspace, and an
    invitation to `ponderer` reaches the conductor;
  - `evaluate`: episodes, research commissions, wedo revisions,
    adoption;
  - `freework`.
- **Knowledge**:
  - `developer/wedo-argue` holds reusable discussion findings with
    trials, contrary observations and revisions;
  - `developer/study-argue` holds literature research, written by
    archsage `study` runs that ponderer commissions. archsage binds it as
    a study destination.

  Both are submodules of pj-ponderer (`wedo/`, `study/`), and both carry a
  `knowledge.yaml` (maintainer ponderer). ponderer adopts a revision by
  moving its pin.
- **Subject findings stay with their owners**: asset provenance from
  p16-s4-provenance is forge's `knowledge/asset-provenance.md`. archsage's
  index has a `discussions` domain.
- **WFE** (agdevworld `153bdad`; aghangout `6210d30`):
  - `conversations.json` `discussions.archive` is `ponderer`;
    `argue/synthesize` is no longer a participant;
  - store **v8** `archives.destination`. A publication verified in another
    repository is `unverified`, never current; the writer clone is rebuilt
    on a destination change;
  - a run of an unregistered (retired) project keeps its name in views
    but gets no run-view link.
- **argue retired**:
  - `wfe project unregister argue`;
  - the workspace (with its ignored run store) is in
    `.local/agdev/.local/retired/pj-argue`;
  - the Gitea repository is left unchanged and unused.

  Historical speech keeps participant `argue`, author `argue-agent` and
  runs `argue/…`.

### As delivered in argue p4

**A discussion's progression is one conductor run of `ponderer/conduct`
(agent `ponderer-agent`, conversation lane), whose graph grows with the
discussion's activities.** Episode: `devdocs/episodes/argue/p4/`. Contract:
agdevworld `workflow_editor/docs/discussions.md` (*Conductor*, *Activities*,
*Annotations*); `wfe discussion help`.

- **Roles.** ponderer (managed project `ponderer`, Gitea
  `developer/pj-ponderer`) owns progression: the issue, who is asked what,
  sufficiency, the end and its account. Front owns the Developer's purpose
  and which projects take part (`wfe discussion join` from its `discuss`
  run). Participants work in their own workflows. (In p4 argue still kept
  archives, summaries, knowledge and the episode index and `argue/synthesize`
  was invitable; p4 ex1 moved all of that to ponderer, above.) Nobody's
  "go" is needed.
- **Activities** are node metadata (`activity: announce | ask | ponder |
  talk` on do/talk nodes), each bound to the discussion's records by its
  command (`wfe discussion announce|ask|settle|ponder|talk --node`).
  WFE completes an ask when its invitations are resolved and continues the
  run once; `ponder --next` records a decision with its sources and the run
  revision adding the next nodes; `talk` asks the requester (the person, or
  the run that delegated to ponderer). `wfe discussion state` is the
  conductor's compact read.
- **Routing.** A person's untargeted message goes to the open conductor run
  (the answer to its question, else an instruction); only without one is a
  new conductor run started. Late outcomes after a conductor completed
  start a new conductor run. `wfe discussion post` refuses an executing
  attempt.
- **Records.** Store v7: `activities` + `activity_events`, `annotations`
  (interventions, recoveries, evaluations, corrections — never speech),
  `conductor`, `requester`, `messages.node`. Archive transcript records
  `activity`, `annotation`. `announce --conclude` snapshots obligations
  (who owes what, who receives it). Reconcile completes a node whose
  message was delivered but left running (p3 rts-vs-bot) and lifts that
  hold. `contribute --completing <node>` speaks and completes at once.
- **Where integrated work is**: `wfe projects run <p> <wf>/<run>
  --integration` — run.integrate records, published branch, local workspace,
  kept apart (ex1's "not on main").
- **Views.** aghangout meeting room *Progress* panel (conductor, what it asks
  whom, decisions with sources, activities, what is owed; annotations in
  Records); run view activity badges; Home Discussions "now:/last:".
- **Episodes.** pj-ponderer (pj-argue until p4 ex1) `episodes/index.json`
  (`ag.argue.episodes.v1`) and one analysis per episode (`tools/discussions
  episodes [validate]`): argue-p3, argue-p3-ex1, argue-p4.
- **Trial (argue-p4-trial)**: progression, participation and delegated
  execution came from the agents with no hint (9 min 25 s); the conductor
  concluded without the protocol's results review, and its conclusion's
  "on branch" and "nothing owed" were stale against WFE's own records.
- Known limits: one ask carries one question text for all its addressees;
  aghangout's dialogue can show the previous line's typing after a jump.

## Reading discussions (`argue` p4 ex2, 2026-10-21)

**A person picks a discussion by what it is for and what came of it, and
enters it on that.** Episode: `devdocs/episodes/argue/p4/ex2/`. Contract:
agdevworld `workflow_editor/docs/discussions.md`, *Reading a discussion*.

- **WFE** (agdevworld `5d6f511`, conversation store **v9**):
  - **category**: `discussion` | `check`. It is set at creation
    (`create --category check`, API `category`) and corrected with
    `wfe discussion meta … --note`;
  - **display title**: recorded explicitly, never derived from the
    original;
  - both have an append-only `discussion_meta` history. A metadata write
    sends nothing, invites nobody, reopens nothing and does not touch
    `updated`;
  - **overview**: append-only revisions with purpose, outcome and
    unresolved points. Every part cites `[#n]`, and WFE refuses a part
    with no source. Each revision records `through` (the coverage),
    author, run and note. It is an interpretation, not speech or
    agreement;
  - the overview is written with `wfe discussion overview` (from ponderer's
    runs, or by a named author) or `POST …/overview`. Without one, readers
    get the **brief**, labelled as the request;
  - **status** comes from WFE's records: open/closed, working, owed,
    whose answer is awaited (`you` only when the receiver is the
    configured person; a `talk` question goes to the requester, possibly
    a run), failed invitations and held runs;
  - `GET api/discussions?category=&state=&limit=&offset=` returns counts
    per view and computes overview, brief, status and `newest` for the
    page's rows only;
  - the archive gains `meta` and `overview` records and the manifest
    fields `category`, `displayTitle` and `overview`.
- **ponderer** owns the overview (`agent/conduct.md`): written in Japanese
  for the readers, citing the original speech, revised when purpose,
  outcome or open points change. It is not required at every turn. In the
  ex2 check it published one unprompted (`argue-p4-ex2-check`).
- **aghangout** (`c66bd02`):
  - **the lobby's 議論 list**: one column on a plain surface. Each row has
    its title (wrapping), 結論 or 目的 (or 依頼文の抜粋), status words, the
    participating projects, the time and the action;
  - views 議論 and 動作確認 with 進行中/終了/すべて, counts and
    「さらに表示」. The list's place is kept in the address and restored;
  - **rooms open on the overview**: 目的/結論/未解決, `[#n]` opening the
    original, coverage, and 次に必要なこと;
  - then the verbatim conversation and the composer. The visual novel is
    「再生表示」, and 詳細 corrects the category and display title.
- **Classification and backfill**:
  - 6 historical checks: `p3-s2-fixture`, `p4-s3-check`, `p4-s5-a`,
    `archsage-topic-line`, `forge-media-gap` and
    `20261020-233458-r93d5v`;
  - 6 ordinary discussions given Omni-written overviews and display
    titles (handoff candidate for ponderer).
- **Known limits**:
  - an overview is only as current as its last revision. Readers see
    "以降に発言N件", but nothing refreshes it automatically;
  - the panels' inner texts (Progress, Invitations, Records) are still
    partly English.

## Workflows beside the speech (`argue` p4 ex3, 2026-10-21)

**A reader sees which workflow a discussion turn referenced, created, edited
or executed, at the version that turn had, beside the speech.** Episode:
`devdocs/episodes/argue/p4/ex3/`. Contract: agdevworld
`workflow_editor/docs/discussions.md`, *Workflows in a discussion*;
`wfe discussion help` (`workflow attach|link|list|show|repair`).

- **WFE** (agdevworld `2eb1976`, conversation store **v10**):
  - `workflow_refs` records, each with:
    - the relation and the role (`subject` / `progression`, the
      conductor's own graph);
    - the target (a definition file, or a run);
    - the versions before and after: byte sha256, semantic digest, commit
      or run revision with `seq`, and `at` (when that version came to be);
    - the turn (run, attempt, invitation, ponder activity, `origin` for a
      delegated or requested child), the messages, the evidence
      (`operation` / `claim` / `backfill`) and the operation;
  - every named version's bytes in `workflow_snapshots`, and later message
    associations in `workflow_ref_events`;
  - **recorded by the operations themselves** from a discussion turn:
    - `wfe workflow save|new` (journaled, recovered from the file);
    - `wfe request --request` (journaled by receipt);
    - `wfe run adjust` and `discussion ponder` (from the run's history);
    - `wfe delegate request` (once the child exists);
    - an explicit `wfe discussion workflow attach` for consulted material,
      which proves retrieval, not comprehension;
  - **not recorded**: reads by shell and edits in the editor;
  - reads at `api/discussions/<id>/workflows[/<n>[/current]]`: the pinned
    graph with replayed node states, the comparison by node id and edge,
    and the current target labelled apart. Nothing is executed or
    dispatched;
  - the archive carries `workflow` and `workflow-link` records and
    `discussions/<id>/workflows/<sha256>.yaml`;
  - **`/wfe/embed/graph.js`**: the editor's Canvas, read-only, in a shadow
    root (`src/embed/graph.ts`, `vite.embed.config.ts`), with change marks
    as words and patterns.
- **aghangout** (`96d1b99`):
  - cards under the messages WFE names;
  - a turn's operations with its speech (「このターンの操作」), or as
    「発言のない操作」 rows when the turn said nothing;
  - progression folded into 「進行の更新 N件」;
  - one ワークフロー panel (a sheet on a phone):
    - このターン時点 first, 現在 labelled as not the recorded version;
    - 変更後 / 変更前 with the change listed;
    - node details with the messages recorded about a node;
    - the state at the recorded point;
    - the way back to the turn, and 「エディタで開く（現在の内容）」.
  - Browser checks need `AGHANGOUT_WFE_EMBED=<agdevworld>/workflow_editor/dist/embed/graph.js`
    for the graph cases.
- **Guides**: the agautolab wfrun guide, *A run that speaks in a
  discussion* (agautolab `c08f117`), and pj-ponderer `agent/conduct.md`
  (`acb25ae`): one fact each.
- **Backfill**: `wfe discussion workflow repair --all`, run once at
  deployment.
  - 36 records came from proven sources only: conductor revisions with
    their ponder activity, delegated children, and hand-off links.
  - Each is `evidence: backfill`, with its source in the note.
  - The versions' `at` was added a few minutes later, verified against
    the runs. The archive's transcript lines for the first three
    (exported before that) lack it.
- **Known limits**:
  - no record of what an agent read by shell, or of edits made in the
    editor;
  - a definition saved before this episode has no captured history;
  - the graph's type labels and "Fit" are the editor's English.

## One way to ask, and catalog nodes (`workflow_editor` fix3, 2026-10-22)

**A question is one thing: the run's `wfe run ask`. WFE posts every open
question of a run that speaks for an open discussion there.** Episode:
`devdocs/episodes/workflow_editor/fix3/`. Contract: agdevworld
`workflow_editor/docs/discussions.md` *Questions*.

- Why: `ponderer/conduct/run-020` relayed gimmickimg's delegated question
  with `wfe run ask` alone. The discussion said it awaited nobody while the
  run awaited the person (`awaiting-person`).
- Posting is done after `wfe run ask` and by `wfe exec reconcile`, once per
  question. On the conductor's talk node the question is the talk
  activity. `status.awaiting` names whom the run asked.
- `wfe discussion talk` and `contribute --kind question` are refused.
- The node catalog's meaning (brief, `wfe run nodes`, wfrun guide):
  - perform a relevant activity as a node of its own **as far as
    possible**, not inside a talk;
  - not an absolute rule.

  gimmickimg/freework/run-001…004 did all their work inside freework's
  one talk.
- Completing outside the discussion's commands warns rather than refuses
  (`run complete` / `adjust --completing` on announce, ask or ponder, or
  ending a conductor run whose open discussion it never concluded). The
  warning records nothing; `--yes` accepts. run-020 had ended its
  discussion with nothing said.
- Deployed agdevworld `25c74c4`; agautolab `40f32e7`; pj-ponderer
  `b76fcad`. run-020 q1 was rescued as #301, and its conclusion by
  run-021 (#303).

## Character dialogue (`argue` p5, 2026-10-21)

**A discussion reads as short character lines: each agent message as one
line in its speaker's voice, with a portrait. Selecting a line shows the
original in place.** The lines are presentation only. The original speech
is what everything else works from. Episode: `devdocs/episodes/argue/p5/`.
Contracts:

- agdevworld `workflow_editor/docs/dialogue.md` (`wfe dialogue help`);
- aghangout `renderer/README.md`;
- pj-characters `docs/format.md`.

- **The cast is the managed project `characters`** (Gitea
  `developer/pj-characters`, agent `characters-agent`).
  - **Origin.** It was imported with its history from GitHub
    `iwaag/agdevworld-settings`; that repository is no longer read.
  - **Data.** `manifest.toml` (`ag.characters-manifest.v2`) lists each
    character with lore, a short speaking style, accent, portrait and
    provenance, plus the rooms.
  - **Tools.** `tools/characters list|show|validate|normalize|add|export`.
    Diagnostics name the file and the fix, and web derivatives are checked
    against the originals by hash.
  - **Workflows.** `character` (add or change one) and `freework`.
  - **No runtime identity.** Which participant a character stands for is
    each consumer's mapping; the old `agents` lists are in `HISTORY.md`.
- **aghangout consumes a pinned export.**
  - `app/public/cast/` is `characters.json` (`ag.characters.v1`) with web
    assets, refreshed by `npm run characters -- <rev>`.
  - `app/cast.json` maps WFE participants to characters, rooms and lobby
    lines: front, forge, archsage and ponderer. Everyone else is plain.
  - Adding or changing a character needs no UI source change.
- **The renderer is aghangout's** (`renderer/render.ts`, version
  `dialogue-9`, model `sonnet`).
  - **Input**: one snapshot of original speech. **Output**: one structured
    line.
  - **The model call**: Claude Code print mode, with no tools, no MCP and
    no setting sources.
  - **Rules**, each from a failure seen in the trial (report5):
    - stance and function are kept, and asks to the human are kept;
    - one register per speaker, and nobody calls themself by their
      nickname;
    - suggestions stay suggestions;
    - no message numbers;
    - the requested language, checked by script with one corrected retry.
  - **Installation**: `scripts/deploy.sh` installs it beside each release
    as `hangout/renderer/<commit>` and `renderer-current`, which is not
    served.
- **WFE keeps the lines apart from the speech** (agdevworld `35b3474`).
  - **Storage.** `<archive workspace>/.local/character-dialogue/dialogue.sqlite`,
    today pj-ponderer's. It is refused unless Git ignores it. It is never
    in the archive (the writer has its own clone) and never in a run copy
    (only regular `.local/` files are copied).
  - **Identity.** A line is keyed by renderer, model, the speaker's voice
    and the exact request: the target, the replied-to and four earlier
    originals, nothing later. A changed input is a new line; the old one is
    kept but never shown as current.
  - **Jobs** are durable rows with a holder (`host:pid:nonce`). A dead
    holder's job is reclaimed at once; otherwise it waits out its 5-minute
    lease. Each job has 3 tries, and result and completion are one
    transaction (store v2).
  - **The worker** lives in the WFE service process and holds no executor
    slot. It follows new agent speech (`ja`) from its first start and never
    renders history wholesale. A reader's read queues at most 40 missing
    lines and calls no model.
  - **Routes.** `GET api/discussions/<id>/dialogue?lang=`, `POST
    …/dialogue/retry` and `GET api/dialogue`.
  - **Untouched.** Discussion reads, `show`/`state`, overviews, the archive
    and run context are unchanged.
  - **Configuration.** `conversations.json` `dialogue`.
- **Reading** (aghangout `97806f5`):
  - **The line.** A portrait (or initials) and a bubble. The bubble is a
    button (click, Enter, Space) that opens the original in place, keeping
    the place and focus.
  - **Citations** open the cited original.
  - **The person's words** are always shown as written.
  - **Lines not ready.** Pending and failed lines leave the original, and
    a failed one offers a retry.
  - **表示.** 短い表示（日本語/English） / 原文, kept in the browser.
  - **The player** uses the same lines.
  - **Replies and cards** always name the original.
- **Measured** on p16-s3-room's 14 agent messages (deployed; costs as
  reported by Claude Code at list price):
  - about 6–8 s and $0.014 per line;
  - Japanese lines average 167 characters against 1,920 in the originals;
  - a new agent message became a line in 6–9 s, with no reader involved;
  - a service restart mid-render left no duplicates.
- **Known limits:**
  - adherence is probabilistic: in the final reading one message number
    and one run name remained, and English lines run longer (mean 381
    characters);
  - one renderer process per line (about 5 s of CLI start each);
  - characters other than front, forge, archsage and ponderer have no
    mapping, so their lines use a neutral voice;
  - lines are not shown in Slack or the WFE run views.

## Challenges (`challenge` p1, 2026-10-23)

**A challenge is a problem found during work that is worth keeping after
it: a durable, shared record any project can find, read and carry into a
later conversation, discussion or request.** Finishing the work resolves
none. Episode: `devdocs/episodes/challenge/p1/`. Contract: agdevworld
`workflow_editor/docs/challenges.md` (`wfe challenge help`).

- **Records**: one Gitea repository, `developer/pj-challenges` (a data
  repository; no agent, workflow or service of its own).
  - `challenges/<id>.md`: frontmatter `id` (`ch-0001`, allocated by WFE,
    immutable), `title`, `state` (`open` | `resolved` | `withdrawn`),
    `projects`; the Markdown body is free-form;
  - `evidence/<id>/` holds essential files (≤ 32 MiB each; Gitea has no LFS);
  - who wrote what, when, from which run, attempt and message is the Git
    history (author = actor; `Challenge-*` trailers), never form fields.
- **WFE writes, Git is the truth.**
  - Every create/edit goes through a journal (request identity, base
    revision) and a serialized writer clone; it is pushed without force.
  - A stale base is refused with the current record. A remote change after
    acceptance becomes `conflict` and keeps the draft.
  - A failed or unseen push is `unsynced`, retried by every reconcile. A
    request already in the remote history is recognised, never written
    twice.
  - The SQLite index is derived and rebuildable (`wfe challenge reindex`).
  - Backup: a mirror on agpc's disk, pushed after each publication
    (`wfe challenge status`); recovery is a clone of it plus `reindex`.
- **Reads**: `wfe challenge list|show|history|evidence`,
  `api/challenges…`. Current reads vs pinned reads (`--at <commit>`).
  Reading or recording starts no run.
- **Carrying into work** (conversation store **v11** `attachments`):
  - a person's message in a chat or a discussion (`challenges` on the post,
    `--challenge <id>[@<commit>]`), or a request (`wfe request
    --challenge`), carries challenges pinned to a revision;
  - the run it starts gets that revision's file
    (`attachments/<id>@<commit12>.md`) and `input.attachments`, named in its
    brief;
  - a reply delivered to a working run carries them in `source.attachments`;
  - archive transcripts keep them. A later edit never changes an earlier
    turn.
- **aghangout** `6e311e4`: the 課題 tab in chat and meeting rooms and its own
  page (`?challenges`, `?challenge=<id>[&at=<commit>]`):
  - a cross-project list with filters, a record's view (evidence, history,
    sources) and a simple editor whose draft survives failures and
    conflicts;
  - 課題に記録 on any message;
  - attaching to the draft (removable), the version sent vs the latest;
  - この課題で議論を始める.
- **Guides**: agautolab wfrun guide *Challenges* (facts, no mandatory
  steps); Front records "solve some day" feedback as a challenge as
  front-agent for the Developer (converse/run-055 had put it in a context
  block).
- **First record**: `ch-0001` — object extraction leaves the surrounding
  image at the edges of cut-outs (gimmickimg freework/run-006, the
  Developer's #310), with its evidence. gimmickimg's `improvement-feedback`
  context block now points to it.
- **Known limits**:
  - evidence upload over the agdev origin is capped by nginx's default body
    size (use the CLI for larger files);
  - `wfe challenge evidence --out` on an execution node writes on the
    coordinator (read evidence over HTTP there);
  - (resolved in challenge p2: direct conversations are archived, and chat
    and discussions are one conversation — next section).

## One conversation (`challenge` p2, 2026-10-23)

**A one-to-one chat and a facilitated discussion are one kind of conversation
with one service, one delivery mechanism, one archive writer and one room.**
Facilitation is optional and explicit. A room can gain members and a
conductor without changing its id, history, work or pinned challenges. This
supersedes the delivery parts of *Conversations with project agents* and
*Discussions* above. Episode: `devdocs/episodes/challenge/p2/`. Contract:
agdevworld `workflow_editor/docs/conversations.md` (`wfe conversation help`;
`wfe discussion` is the same command).

- **Store v12** (WFE `21a094e`). The fields are `default_to`, the
  facilitation records, `role`/`receipt`/`held` per request, and
  `archives.path`. The migration was pure SQL: direct rooms got members, a
  default addressee and one `converse` request per message, with the legacy
  receipt `conversation:<c>:<m>` kept. Discussions got an active
  facilitation. Nothing was dispatched or replayed. An older release refuses
  v12.
- **Routing**, first match wins:
  1. `to` (participants);
  2. `runs` (an instruction to work under way; an ended run fails visibly);
  3. `re` naming an open question (the answer goes to the asking run);
  4. the active conductor;
  5. the default addressee;
  6. otherwise 409 `choose-recipient`, and nothing is recorded.
- **Response requests.** The former invitation is the only thing that starts
  or continues a run. Each request records the workflow selected for it:
  `converse` for the only project when nobody leads, `discuss` otherwise,
  plus `conduct`, `deliver` and `await`. The same token gives the same
  message and requests, and the same receipt gives the same run. A failed
  request is retried at the same recipient and never redirected. A question
  is always `wfe run ask`, which WFE posts.
- **Facilitation.**
  - `start` takes a brief and records the horizon.
  - `end` changes routing only; nothing stops.
  - Joining a member starts nothing.
  - A continuation that comes due while the room is closed or no longer
    facilitated is held and shown. `resume` runs it once; reopening
    replays nothing.
- **Archives for every room** in pj-ponderer, written by one writer.
  Rooms that began as discussions keep `discussions/<id>/`; every other room
  is at `conversations/<id>/`. The path is recorded per room and never
  moves. The manifest format is `ag.conversation.v1`. Export never starts a
  run. At deployment all 62 existing rooms were current (42 direct rooms
  backfilled). A fresh clone of pj-ponderer restores the records. Pinned
  challenges need `pj-challenges` beside it. Restoring never replays
  delivery.
- **aghangout** `9604695`: one Room (`app/src/room.ts`) and one list. The
  shape (one-to-one, several projects, led by ponderer) is presentation
  only. The composer shows who will receive the words before sending. The
  room offers ＋ 参加, 「Pondererに進行を頼む」, 進行を終える, 再試行 and
  再開. `?conv=<id>` is the room and `?with=<project>` is a one-to-one room;
  `?discussion=<id>` resolves to `?conv=`.
- **Clients and guides**:
  - Slack posts `with: front` and follows the room's routing;
  - the agautolab wfrun guide (`ba9e2c1`) has one section, *A run that
    speaks in a conversation*;
  - Front, forge, archsage, aghangout and ponderer guides are updated;
  - ponderer has a `converse` workflow for direct consultation without
    leading.
- **Verified** (report6): a deployed check room went one-to-one → member
  joined → ponderer led → ended → closed → archived. ch-0001 was pinned in
  both run inputs. A repeated token gave the same message and request. The
  LAN check from agpc passed 7/7.
- **Known limits**:
  - the agpc worker still pins WFE `aadee43` and executor `227ef1c`. The
    prepared nctl batch was not applied (report7). Conversation turns run
    on studio's `conversation` lane;
  - a `?conv=` room without a partner briefly shows 宛先を選んでください
    until its first read returns;
  - turns still share one `conversation` slot.

## Waits and run status (`project_migration` p8 ex1, 2026-10-19)

**A wait is the run's record, never a process, and every view says in a
few words why a run stands still.** Episode:
`devdocs/episodes/project_migration/p8/ex1/`; contracts: agdevworld
`workflow_editor/docs/runs.md` (*Status in short*, *Why a hold was
derived*), `docs/delegation.md`, `docs/observation.md`.

- **Delegation waits.** An attempt dispatches, ends and frees its slot; the
  child's facts start new attempts, minutes or days later, through
  restarts. Every other node it started must be completed, failed or
  waiting by then — run-027 left `talk` running and was held twice. Front
  completes `talk` at the adjustment that adds the delegate node and
  handles the result in a node after it; `wfe delegate request` names
  nodes still running. A child's completion is relayed once, after its
  completing attempt ended.
- **Executor**: a harness that wrote its final result but stays alive for
  something it backgrounded is stopped after 30 s and its attempt ends as
  the session did (trial converse/run-029 held the conversation slot ten
  minutes for a `find /`).
- **Holds keep a cause** (`left-running`, `left-ready`,
  `nothing-recorded`, `missed-event`, `process`, `lost`) beside the
  sentence; an attempt that records nothing of its work is an interruption.
- **One status reading** (`shared/status.ts`): `Waiting · Your answer` /
  `· jevlab study result` / `· <external>`, `Queued · Waiting for an
  execution slot`, `Paused · <reason>`, `Interrupted · Talk node left
  running at exit`, `Unknown · Process lost; outcome unconfirmed`. Run lists,
  run views (details hold the long text), `wfe run list|show`,
  `wfe conversation show` and the Front Desk (run links, the conversation's
  unfinished work) use it. `situation()` splits `awaiting-person|
  -delegation|-external` and adds `due`.
- **Repairs**: reconcile requeues lost instruction/resume jobs; the notifier
  re-queues a missing delegation post row and searches its marker before
  posting. Observer leads an interruption's fact with its cause, gives the
  recovery agent every delegation by identity, sends the same cause after
  two recoveries to the owners, and says (not concludes) an unreadable
  child. Recovery runs must record their `investigate` node.
- Known limits: the Slack bridge's `due` polling change is committed but its
  release was not redeployed; the repeat rule applies to holds recorded
  since this release.

## Front from Slack (`slack_connect` p1, 2026-10-19)

**The Developer can also talk with Front from Slack** (the iPhone app
included): direct messages with a private Front bot in one workspace. It is a
second door to the same WFE conversations, not a second Front. Episode:
`devdocs/episodes/slack_connect/p1/`; code and operations: agdevworld
`integrations/slack/README.md`.

- A new top-level DM is a new conversation; replies in its thread continue it.
  Each accepted message is one WFE message (`via: slack`) and so one Front
  turn; 📥 marks it accepted. Front's replies, questions, progress and results
  come back in the thread with their run.
- Words the person writes in aghangout in a Slack-linked conversation are
  mirrored into its thread; aghangout shows Slack words with `via slack`.
  New DM conversations name the destination `front/converse` (workflow_editor
  p15), and result and question posts link the conversation in aghangout.
- Edits, deletions and attachments start nothing; the bot explains.
- Separate process `com.agdev.slack-front` (Socket Mode, outbound only), its
  own release (`scripts/slack-release.sh`) and transport store; WFE and its
  runs do not depend on it. Nautobot: service `slack-front`.
- Text needs no home network or VPN; run, artifact and desk links need the
  phone's Tailscale.
- Automated trials post as `Omni Agent (for Developer)` via `slack-trial`
  (`slack-front trial`), never as the Developer.

## agfront (pj-agdev/agfront) — retired 2026-10-19 (project_migration p8)

The listener `com.agdev.agfront-zulip` is booted out (plists in
`agfront/.local/archive/project-migration-p8-20261019/`), the retirement
introduction is `#agents` #19888 (✔), and Nautobot has the service retired,
the placement disabled and the agent retired. The checkout, its run records,
mirror and journal, and every Zulip conversation stay as history.
- The relay's Zulip writes that only it served answer 410: desk posts,
  re-voicing, argues, routine chat and "Ask Front to run it".
- Routines, argues and character re-voicing are retired with it.
- What follows is history.

### agfront as it was

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

## lowerpoly (managed project `lowerpoly`, project_migration p9, 2026-10-20)

**An existing repository onboarded by Front, not created.** Gitea
`trustedagent/pj-lowerpoly` (history kept from `a31efb8`), workspace
`lowerpoly`, agent `lowerpoly-agent` (profile `sonnet`). Mesh simplification
/ LOD research for Meshy GLBs. Local areas `study/`, `wedo/`, `lab/`
(ag.project.v2 `areas`) live in the root's history. Episode:
`devdocs/episodes/project_migration/p9/`.

- Workflows `freework` (exploration) and `lod-chain`: one input GLB on the
  share → Phase 2 default LOD chain, evaluation, comparison with the
  recorded table, `wfe run publish`, review page. The workflow's intent says
  which trial run each node came from.
- **Toolchain** (Blender with Metal bake, Python venv, gltfpack, msimplify,
  mdt_lod, KTX-Software, gltf-transform) exists only in the original working
  copy on the coordinator host. Runs link it read-only
  (`lab/first_experiment/tools/check_toolchain.sh --link`), so lowerpoly runs
  only on worker `studio`.
- **Review pages stay up until the Developer closes them.** Front's
  `knowledge/review-pages.md` holds why: p9's first page was stopped by a
  "relayed" answer, and a promised link had no run to carry it.

**Gitea as a named principal** (`wfe gitea whoami|repo|git --as <name>`;
`gitea.json` `principals`). The service and every other `wfe` operation keep
the service principal (`developer`). An agent pushes as another principal
only through this command. The token is never printed or stored, and force
pushes are refused. p9: Front's preparation pushes to `pj-lowerpoly`
(project.yaml, guide, freework) were trustedagent; registration, the
workspace clone, run checkpoints and integration are `developer`. Run commits carry the host's Git user name;
the run record names the agent.

## Project access (`wfe projects`, since `project_migration` p4, 2026-10-16)

**Reading any managed project is a system capability, not an agent's.**
`wfe projects list|show|tree|read|search|runs|run` and the read-only
`GET api/projects…` routes (agdevworld `workflow_editor/docs/access.md`)
read every registered project and registered repository (`repo:<owner>/<name>`),
with or without a workspace on this host, from system-managed caches
(`<area>/.local/access-cache/`): committed `main` by default, a submodule at
the commit its project **pins** or `--upstream`, `--rev`, `--require`, the
working tree only with `--source worktree`. Every result names project,
repository, commit, the commit that pins it, and fetch state; failures are
`absent` / `inaccessible` / `uninitialized` / `fetch-failed`. Runs show the
**live** record beside the **published** snapshot. Every run's agent sees it
(agautolab `wfrun` guide); Front can use it as is.

Two additions came with it:

- `execution.guides: ["<project>:<path>"]` — a guide of another managed
  project, read at claim from its committed `main`.
- A binding's `integrate: if-unchanged` — integration stops as
  `target-changed` when the target changed under the binding since the run
  built on it; reassessment is a successor run.

## Shared areas (`shared://`, since `project_migration` p6, 2026-10-17)

**Large inputs and outputs travel on a shared area, by reference and
manifest, never through Git.** A person puts a folder on the share and
names it in their own words (`/agshare/x`, `agshare/x`, `x`, "the x folder
on the NFS"); since `workflow_editor` fix1 the agent turns that into a
reference with `wfe share resolve` (one stat per reading, inside configured
roots) or `wfe share find` (bounded name search that says when it was
incomplete), and the records keep the reference. A `shared://` reference
written in the request is still recorded by `run.create`. `wfe run input` records the exact files used as an input set
(hashes; staged into the run copy's `.local/inputs/` by default), `wfe run
publish` puts an output folder on the share as `o<n>` of a named candidate
(`.partial-*` then renamed; never partially visible), and `wfe run ask …
--output <o>` binds the confirmation to that output's manifest. Contract:
agdevworld `workflow_editor/docs/shares.md`.

- Where a share is on a host is that host's `<area>/shares.json`; the share's
  root carries `.wfe-share.json` naming it, so an empty mount point reads
  `unmounted`, not "empty".
- `wfe share list|resolve|ls|find|verify` work without a run; `wfe share verify
  <manifest.json>` is how another PC checks a delivery it got by reference.
- Downloads: `GET api/shares/<share>/<path>` (byte ranges) on the WFE
  service, through the usual LAN address.
- **Operating it as a person.** Put a folder anywhere under the share (from
  the server host or a client's mount) and name it in the request with any
  explanation; there is no upload form. An accepted output is the `o<n>` the
  run's confirmation names: open its download link from the run view, or
  `wfe share resolve` it and copy it off the share before using it (tools
  that import or compile write next to their files; p6 step 7 wrote
  `__pycache__` into a published output that way). Dot entries are never
  carried: a delivery must not list one.
- **An unavailable share** shows as `unmounted`/`unavailable` ("absence is
  not confirmed") or a hang in `wfe share resolve`, not as an empty folder
  or an absent item. The share is a cagent-managed
  service (`agshare`, changes `run-005`–`run-007` in pj-cagent's records):
  report the state and the host to cagent as a `change` request; do not
  repair mounts by hand outside it.
- **Retention** of inputs and accepted outputs belongs to the person or the
  project. Rejected, superseded or unaccepted outputs also stay until the
  person decides; no run or cleanup deletes them incidentally.

## archsage (workflow-editor project `archsage`, since `project_migration` p4, 2026-10-16)

**archsage is a managed project, not an agag agent.** `developer/pj-archsage`,
workspace `archsage`, agent identity `archsage-agent` (profile `sonnet`). It
advises from what the managed projects have learned, performs requested
studies, and supplements a project's wedo only where that project assigned it.

- Workflows `consult` (answer with sources by project/repository/commit/path,
  unknowns kept apart; `advice.md`), `study` (agreed scope and destination,
  into a shared study bound as a submodule), `backfill-wedo` (shared wedo,
  guarded), `freework` (every knowledge destination readonly). Channels
  `wf-archsage.*`.
- `index/` — where knowledge lives (`sources.yaml`, `domains/`), not a copy
  of it; `questions/` — proposed research questions. The old sages' domains
  and queue notes were imported with provenance.
- **Maintenance is declared** by a `knowledge.yaml` at a knowledge target's
  root: `maintenance: project | supplemented` (+ `supplementer: archsage`);
  absent = unknown = not eligible.
- **Knowledge inside another project's root** is written by that project's
  run following archsage's guides (`execution.guides:
  [archsage:agent/guide.md, archsage:agent/backfill-wedo.md]`): rts-vs-bot
  `backfill-wedo` writes its root-local `wedo/`. A shared repository is
  written by archsage's own run; integration moves the shared branch and
  archsage's pin, never another project's pin.

The autolab studies (`autodev/*`) stay autolab's; their routines (v4 guides,
2026-10-16) no longer ask archsage to refresh. Front's legacy guides named the
old `@**archsage**` / `sage:` addressing; the managed Front (project_migration p8) reaches
archsage through its workflows.

## archsage — the knowledge council — retired 2026-10-16

Listener `com.agdev.archsage-zulip` booted out, plist archived, retirement
introduction in `#agents` (#16705), Nautobot service/agent `retired`,
placement `disabled`; the checkout, its records, the sage store
`autodev/archsage-sages` and the conversations stay. What follows is history.

### archsage — the knowledge council (`archsage`; `argue` p1, 2026-09-16)

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

#### archsage establishes studies (`sage` p2, 2026-09-26)

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
  posts each guide version as the caller, **read back** (a post Zulip cut
  before p4, or around the library, ends in `[message truncated]`).
  Registering starts nothing. The board and
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
synced to that revision). Since `progress_panel` p1 the last is a record:
`archsage sage sync`/`attach` inside a serving writes `[selfnote][sagesync]
<sage> <revision> project=<slug> findings=<n>` into the conversation served. Research with the setup means Front runs the
study's routine afterwards, and the guide archsage writes ends with asking
archsage to refresh the sage. `devdocs/episodes/sage/p2/`.

## Argues (`argue` p1, 2026-09-16) — retired with the legacy Front (project_migration p8)

The Arguing Room reads history only. Discussions with several project
agents returned in workflow_editor p16 (*Discussions*, above);
nothing of the Zulip argues was imported. History below.


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
  (humans, Front; its own folder) and posts the goal; repositories and
  workflows belong to a WFE managed project (`wfe create`), and the old
  autolab setup handshake is gone (`retire/old_autolab`); **a new study is asked of archsage** in its
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
by the notifier's two intake paths and (until retire_old_ui p1) by the relay's ops board. **A consumer
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

**The Project Room** (`project_room` p1) is gone with the old autolab
mission system (`retire/old_autolab`, 2026-10-19).

## forge (workflow-editor project `forge`, since `project_migration` p3, 2026-10-14)

**forge is a managed project, not an agag agent.** `developer/pj-forge`,
workspace `forge`, agent identity `forge-agent` (profile `sonnet`). It makes
or changes media assets (text included) with capabilities proven here, says
plainly when it cannot, and keeps the demand.

- Workflows: `provide` (assess in the plan, make or explain, present,
  confirm), `develop-capability` (demand → research → implement → verify →
  catalog), `refresh-catalog`, `freework`. Channels `wf-forge.*`.
- `catalog/` — one entry per capability: state (`verified` / `unverified` /
  `planned` / `elsewhere` …), sources as `<source>@<rev>:<path>`, *Proof
  here* kept apart from knowledge. `requests/` — one demand record per run
  that could not be (fully) served, with `Later` lines from the runs that
  act on it.
- `tools/forge --help`: images (SwarmUI), edits, ComfyUI jobs with a record
  written before queueing (submit once, collect once, survives an
  interrupted attempt), durable delivery by content hash to the old agforge
  bucket (`forge/<h2>/<sha256>`; any old key still signs), speech (`say` +
  ffmpeg, from develop-capability/run-001).
- Host settings are the workspace's ignored `.local/` (`forge.env`,
  `sources.toml`, `comfy/`, `localenv.md`); tools find it from a run copy's
  location.
- **No documentation step after `confirms: result`.** An acceptance binds
  the result as presented; a report written after it stops p9 integration
  (`needs-agreement`, seen on provide/run-001–003). agsfx, ppt and
  rts-vs-bot still have that pattern.

## forge agent(pj-agdev/agforge) — retired 2026-10-14

Listener and request service stopped, plists archived, retirement
introduction posted to `#agents`; the checkout stays (comfynotify's venv).
Front reaches forge through its workflows since project_migration p8.
What follows is history.


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

- The run puts what it wants said between a `<ag-reply …>` line and a
  `</ag-reply>` line (**since `failsafe` p3**; it was a fenced block, and a
  reply opened with three backticks ended at the first bare fence of a code
  block inside it — an ambiguity no rule could resolve, so the fenced form
  is retired and a run that writes it is repaired with that reason). Inside
  the mark code fences are ordinary Markdown, paired as CommonMark pairs
  them. Several blocks are one post, in order. An unreadable attribute on
  the opening tag (`to=Omni Agent`) is dropped and named; it never costs the
  reply. Everything outside the mark is the run's own, kept in the
  transcript and never posted. Machine blocks (`ag-argue`, `ag-routinerun`,
  `ag-continue`) are read from the whole output by their own splitters and
  never posted. **Nothing is cut** (`failsafe` p4): the realm keeps
  100 000 characters per post (`SETTING_MAX_MESSAGE_LENGTH` in the Zulip
  deployment's override). `ZulipClient.max_message_length()` learns it
  from `/register` once, keeps it in `<credentials>.limits` for 6 h, and
  falls back to the last value learnt, else Zulip's default 10 000. Every
  send refuses a longer post before sending (`MessageTooLong`, a
  rejection), and a run's reply over its room (the limit less the
  listener's lines and 300 for the mention and `ag-post` line) is a failed
  reply: repaired once for size, then owed, the words kept whole in the
  journal (`extra.reply_unposted`).
- No mark, an empty mark or an unclosed one is a **failed reply**, not
  silence and not "post it all": the run is asked once more for the reply
  alone with its previous output and the exact reason in front of it
  (`repair_prompt`; every action and machine block has already happened).
  **A reply that still fails is owed** (`failsafe` p3): the failure line is
  `progress` without `end=`, no receipt is written for what the serving was
  given, the journal keeps the reason and the run's own output
  (`extra.reply_owed`), and the listener serves the same input once more
  after `REPLY_RETRY_SECONDS` (20 s) with that output in the prompt
  (`prompt_with_guide` appends the notice: do not repeat anything). A second
  failure is the last: a closing `report`, receipts written, and the
  request's own conversation ending in its agent's failure notice is the
  monitor's `unanswered` kind, reported to the owners.
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
- The rooms (Front Desk, Arguing Room, routine chat) label each
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
instance left the agent room and the operation room (both retired in
retire_old_ui p1), its channel stops being
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
