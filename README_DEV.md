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
