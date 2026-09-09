# step1 — the public execution contract

**Deliverable:** `pyagag/docs/exec-options-v1.md` (`ag.exec-options.v1`).

## What was decided

The braindump's worry was coupling: a requester who wants `agy` should not
have to know autolab's `agents.toml`. So the contract separates two names.

- **Execution option** — a public name an agent advertises, with the usage
  pool it consumes and the work it covers. The agent's own vocabulary.
- **Profile** — the internal `agents.toml` name it maps to. Private; nobody
  outside the agent may name one. One-to-one is a valid mapping and is what
  agfront/agautolab will ship with.

### Discovery rides on the introduction

The `#agents` introduction is already the contract that travels as content
and is already re-posted after a behaviour change, so the options go there
as a fenced `agag-exec` block beside the existing `agag-roster` one, and are
generated from the running instance rather than written by hand — the same
argument that produced the roster block in `operation_room` p2.

The braindump asked whether a *separate* topic would be better ("implementation
detail disclosure"). It would not be: what is published is not implementation
detail — it is a small public menu, deliberately not a dump of `agents.toml`
— and a second topic is a second thing to keep fresh. The contract leaves the
door open (`option:` lines may point at a linked detail topic if a catalog
ever outgrows a post), but the first cut is inline.

`supported: yes|no` is carried so "asked and answered no" differs from "never
asked". **A missing block is `unknown`, never "unsupported"** — the roster
block's rule, for the roster block's reason.

### Selection is a topic-local command

    @**<bot full name>** use <option>
    @**<bot full name>** use default

Chosen shape and why:

- **A post, not a CLI.** `zulip_command` already proved the shape for the
  ComfyUI notifier: any agent that can reach Zulip can issue it, nothing has
  to be handed a binary, and a mention inside a code fence is not a mention —
  which is how the contract, the guides and these reports quote it without
  firing it.
- **Mention-addressed**, because that is what "targeting the topic's owner"
  means in a channel where several agents talk.
- **Whole-message only.** A mention embedded in a sentence is discussion.
  This is what keeps *"I asked forge to `use agy`"* from being a command.
- **`use default` is the reset**, resolving to *no selection* so it falls
  through to the existing defaults. An explicit "pinned to default" mode
  would be behaviourally identical and one more state to explain.

A message that is only commands is a **configuration-only post**: applied
without launching a model. The owner answers it with one deterministic line
rather than a reaction — a reaction would leave the poster as the last
speaker and the topic would match the sweep forever. (The notifier reacts
because it is *not* the topic's owner; here that reasoning inverts.)

### Freezing, precedence, inheritance

- **Frozen at each serving's start**: the newest directive at or below the
  serving's `processed_up_to`. A command posted mid-run has a larger message
  id and applies to the *next* serving, callbacks included. Nothing reaches
  into a running process.
- **Precedence** is two levels, not four: the newest directive in this topic
  (command or inherited snapshot, by message id), else the existing
  overlay/committed/project defaults.
- **Inheritance is a snapshot**, written by the parent into the child before
  its visible description as `[selfnote][exec] <option> from <ch>/<topic>#<id>`.
  A selfnote, so it buys nobody a run. Later parent changes reach new
  children only; a child overrides by posting an ordinary command, which is
  newer and wins by the same precedence rule. A callback's remote topic is
  never the execution context — the selection is read from home.

**Topic history is the durable store.** There is no selection file: the
command post and the snapshot note are both in the topic, so a listener
restart re-derives the same answer, and "why did this run on agy" is
answerable from the conversation alone.

### Rejection

Two failures kept apart, and neither ever downgrades silently:

- **not advertised** → one visible refusal post naming what *is* published;
  the previously effective selection continues;
- **not available now** → `E_UNAVAILABLE` at execution time, reported as a
  failed serving.

For a requester relaying a preference: no readable block means **unknown**,
which is reported as unknown — never as "unsupported", and never resolved by
trying a name to see what happens.

### The record

`ag.agent-run.v1` gains `exec_option`, `exec_source`
(`topic|inherited|default`) and `exec_message_id`, beside the existing
`profile`/`harness`/`model`. The public name is the request; the existing
fields stay the fact.

## Deliberate omissions

No approval machinery, no permissions, no audit trail beyond the topic and
the run record; no global switch; no profile name in any consumer's code.
The episode's plan asked for a small contract and this is the whole of it.

## Not done in this step

Nothing is implemented yet — this step is the contract. `agag.agent_config`
already has `profile_override` on `resolve_role`/`resolve_spec_role`, which
is the single seam step2 will drive.

## Commits

- pyagag `0dbc3e9` — the contract document.
- devdocs `ac7e299 (this commit)` — this report.
