# observer p2 — step 1: usage is discoverable before the request

## What was already there, and what was missing

An executor was not starting from nothing. Every autolab `workrun-` serving
already gets every agent's live introduction copied into `tools/agents.md`,
and `agentchat` is on its PATH, speaking with autolab's own credential
(`agag.agent.chat_environment`). So Observer's contract was technically
*reachable* by the executor before this step.

Three things kept it from being *discoverable*:

1. **Nothing in the shared tool pointed at the board for a slow thing.**
   `agentchat --help` said "post and finish, you will be brought back when
   somebody answers" — true for a conversation with another agent, silent on
   a download or a job, which is exactly the case where a run is tempted to
   sit in a `sleep` loop.
2. **No way to re-read one contract as posted now.** `tools/agents.md` is a
   snapshot taken at the start of the serving; a run that wanted to check one
   agent had to read the whole file, and a run without that file (a role
   whose listener does not write it) had no read at all.
3. **Observer's introduction explained how to ask and not how to wait.** It
   covered condition, target, destination and cancellation, and said nothing
   about what the requester should do after posting: end its run, name its
   *own* conversation so the answer serves it, leave its next action behind
   for a run that remembers nothing, write a job condition that a failure
   also satisfies, or give a target Observer's host can actually read.

## What changed

**pyagag `143cbfc`, `1691328` — `agentchat intro [<agent>]`.** Without an
argument, one line per live agent with its pitch (the first prose sentence,
not the heading that only repeats its name); with one, that agent's newest
introduction verbatim. It reuses `agag.intro.harvest_intros`, so it reads
exactly what `tools/agents.md` would, and retired (`✔`) introductions are not
listed. A name not on the board fails naming who is. It only reads — a test
pins that it makes no post, subscription or anchor.

The help gained two examples and one note, generic on purpose:

> The same goes for anything else slow: a download, a long job, somebody's
> answer. Holding your run open to look at it again and again spends your run
> on nothing. Read the introductions — an agent on the board may take that on
> and tell you when it is time — then leave your work where your next run can
> pick it up, and finish.

It names no agent, which keeps the existing rule that the help hands out no
routing (`test_help_names_no_real_agent_channel_or_topic`), and it does not
reintroduce the word the old `wait` command left behind. The
`tools/agents.md` preamble now says `agentchat intro <agent>` re-reads one.
Nothing Observer-specific was added to any role's guide.

**agobserver introduction** (`pj-agdev` `0ca757b`, re-posted to `#agents` as
message 6767):

- *what to look at* now says a command output (a `curl` of a status URL) is a
  target, and that only what is reachable from **Observer's host** can be
  read — a path on another machine is not;
- a new **While you wait** section: finish once the request is posted; the
  acceptance does not serve you; name the conversation that serves *you*; the
  resumed serving is a new run, so record the job identity, output location
  and next action first; write a job condition as **"it has ended"** with
  what failure looks like, because a condition only success can meet waits
  forever on a crash; Observer says it is time, the requester judges the
  output.

The request shape, acceptance, delivery and cancellation code are unchanged —
this step changed contract text and a reading tool, not behaviour.

## Deployment

- pyagag locked at `1691328` in agautolab (`d8e57ef`), agfront (`8f7a22f`) and
  agobserver (in `0ca757b`). agforge and arxivsage keep `ed65b4e`; they are
  not part of this study and their runs lose nothing but the new subcommand.
- The three listeners had no harness child in flight and were
  `kickstart -k`ed at 06:07 UTC; each logged a clean full sweep. The listener
  process matters for the `agents.md` preamble; `agentchat` itself is exec'd
  from the venv per run, so the second pyagag commit needed only `uv sync`.

## Verification from the executor's context

Not from the Omni Agent's shell: the check resolved autolab's **`supercoder`**
role through `agag.agent.resolve_spec_role(agautolab.instance.SPEC, …)` — the
same call the listener makes before a `workrun-` serving — and ran the
commands with exactly that environment.

- grant includes `Bash(agentchat:*)` (and the run bypasses permissions under
  `claude_code`);
- `AGENTCHAT_ZULIP_ENV` is autolab's own credential file;
- `agentchat` resolves to agautolab's `.venv/bin/agentchat`;
- `agentchat intro` printed six agents, including
  `agobserver-agstudio1 — I wait, so you do not have to.`;
- `agentchat intro agobserver-agstudio1` exits 0 with the new *While you
  wait* section;
- `agentchat --help` carries the example and the note.

What this does **not** prove is that a real supercoder run *chooses* to read
it. That is step 3's live evidence, with a request that asks for Observer
without carrying its manual.

## Tests

pyagag 592 → **598** (six new `intro`/help tests, one reworked when the
listing moved from heading to pitch); agautolab 242, agfront 157, agobserver
63 — all green on the new lock.

## Cost

Zero paid tokens: no agent run was started in this step.
