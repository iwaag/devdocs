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
`agecho` is the minimal p1 fixture; runsmoke1's `main/agping/` is the p3
agent-created fixture that Front has reached successfully.

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

## autolab agent(pj-agdev/agautolab)

- Responds to request for explaining/observing/developing projects.
- Its entrance is the Zulip channel named after its instance — questions
  only, nothing is started there. Development work goes in a `workplan-…`
  topic in the project's own `pj-<slug>` channel, because the channel is what
  says which project the work is for. Its own introduction in `#agents` is the
  authority on both (`agent_standardize` p4).
- Since p9 a `workplan-` topic's tasks each get a `workrun-` topic that
  **says what it is for**: a `[rootchat]` note naming the mission
  conversation and a `[work]` note naming its Plane Sub-Work, written before
  the visible description. Nothing is read from the topic's name or its
  channel's description any more. **A post is what starts a task**, and the
  planning reply says so — a supervisor that reads "opened …" as "running
  now" stops the whole mission.

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
conversation: it is the credential the operation room's state engine reads the
realm with (`agdevworld/agentroom`, `operation_room` p2). It exists so a
~240-call sweep does not come out of the quota the agents' own listeners spend.
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

- **A topic Front opens alone is never served by the owner sweep** (last
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

## agfront(pj-agdev/agfront)

- Responds to any requests from Human and sends messages to other agents.
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

## arXiv sage (`arxivsage`)

- Answers from the public `study-arxiv-trend` tree: current papers on LLM
  agents and agent harnesses, their summaries, runnable manuals, and local-test
  reports. Its own Zulip channel accepts questions only in `entrance-…` topics.
- It never edits the knowledge tree or runs a study. An honestly unanswerable,
  reasonable in-scope question becomes a deduplicated Markdown note in its
  ignored `tostudy/` queue for the study workflow to consume.

A **sage** is the reusable pattern behind this domain-specific agent: one
agag instance owns a narrow, externally maintained knowledge tree, cites what
it read, says when that tree does not answer, and leaves researchable unknowns
to the responsible workflow rather than pretending to maintain the source.

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
roster block's rule, for the roster block's reason. Two agents that predate
the contract (agforge, arxivsage) read as unknown today, and that is correct.

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
