# unshackle_agent / agdevworld turn1 — remove the forced paths

Goal: strip agdevworld of Tool Implantation. Every place where code forces the
in-system agent down a deterministic path is removed; what remains is the
*fact* that snapshots, endpoints and services exist, told once, in the fewest
words that still let the agent find them. Then run the same desires that
worked before and see what the agent still does on its own.

Source of the survey: [../../../../../shackle_list.md](../../../../../shackle_list.md)
(agdevworld section). Direction: [../../../../../memo.md](../../../../../memo.md).
Prior turn, whose findings this plan inherits:
[../../agforge/turn1/report.md](../../agforge/turn1/report.md).

This is an experiment, not a hardening pass. A capability that quietly
regresses is the finding we are here to collect — do not re-add a guard to
prevent it. Failures leave evidence; that is enough.

## The shape of the problem here

agdevworld's in-system agent is the assistant behind `POST /api/chat`. Its
situation differs from agforge's in one decisive way: **it has no tools at
all.** It is a single `messages.create` with no tool loop. Everything it can
do, it does by emitting a JSON object into its prose that a regex lifts out
and an `if` statement executes; everything it knows arrives as a plain-text
digest that code wrote for it.

So this turn is not mainly a deletion turn. Deleting the forced paths without
replacing them leaves an agent that cannot act at all. **The work is Tool
Giving: hand the agent the doors, then delete the rails.** F1 of the agforge
turn is the governing lesson — grants widen in the same commit as the pointers.

## What stays (the complete list — everything else goes)

Not exempt because they are useful. Exempt because they guard against
irreversible or cross-boundary harm, guard a resource, or are pure
observation that takes no judgment from the agent.

1. **No unauthenticated write-through to autolab nodes.**
   `assistant/server.mjs` (`handleAutolab`: GET proxied, POST only on a
   `/summarize/` path). Behind that proxy sit `POST /mission` — which writes
   MISSION.md and launches `drive.sh`, an agent that writes code and pushes
   repos, at $0.13–1.35 a job — and the review approve/reject transitions.
   The assistant listens on :8091 with no authentication, and its context
   carries text it did not author (cluster snapshots, and summaries written
   by a Claude run on another node). Opening writes here gives that injection
   surface an execution path onto cluster nodes.

   The invariant, not the form, is what survives: *no request without an
   identity changes the state of a node.* devpolicy/policy.md's Single
   Entrance already says auth may split entrances and that identity, not
   endpoint shape, decides what a request may do. **Introduce identity and
   this allowlist may go** — but not before it.
2. **The node allowlist** (`AUTOLAB_NODES` parsing in `server.mjs`). Without
   it the assistant is an unauthenticated open proxy into the LAN. The whole
   app is built on "the browser reaches only its own origin"; an arbitrary-URL
   relay erases that premise. Same class as agforge's kept `nctl-outbox`
   refusal: cross-boundary damage to systems agdevworld does not own.
   The *fact* that the node list is finite is told to the agent, not hidden.
3. **Run records and the backend allowlist** (`assistant.run.v1` on success
   and failure; `ollama` | `claude`, unknown = error, no silent fallback).
   devpolicy/policy.md (Agent ≠ Model) requires that every agentic run record
   which backend served it, and this turn is measured with those records —
   the same reason agforge kept transcript capture. The allowlist constrains
   the *operator*, not the agent: a silent fallback would make the record lie.
4. **Resource guards**: the fetch timeouts in the passthrough, the image and
   summary poll deadlines, and the new tool-round cap from work item 1.
   Same class as agforge's wall-clock kill — they guard against a hung
   counterpart holding resources, not against the agent being wrong. Numbers
   may be raised; the guards stay, and each is stated to the agent as a fact.
5. **The `/evidence/` refusal** (`403 evidence_not_proxied`) — **kept as a
   property of the passthrough only.** Raw iteration evidence belongs to the
   agautolab node that produced it, and the node-side summarize design rests
   on it. The commanding sentences in the role prompt and GUIDE.md are
   deleted: the agent may try, and the proxy answers with 403 and its reason.
   This is the weakest of the five — its harm is bandwidth and context, not
   destruction — and it is the first to drop if the exemption list is cut.

Two conditions attached to the same turn, which are not exemptions:

- **Secrets.** `ANTHROPIC_API_KEY` sits in the assistant process env; today
  nothing can reach it because the agent has no tools. Work item 1 changes
  that premise. Tools are browser-executed (see below) and no tool reads
  process env or the filesystem — confirm that once, deliberately, when the
  tool list is written.
- **`scripts/fetch-cluster-state.mjs` is Omni-side**, talking to cagent (an
  out-of-system agent), so it is a different layer from this survey. It is
  still loosened (work item 7), but the code-chosen output path and the
  atomic 0600 write stay: a URL extracted from prose deciding *where the
  filesystem is written* is a reach question, not a validation question.

## Work items

### 1. Give the assistant a tool channel (the structural change)

One channel, executed **in the browser**. `POST /api/chat` answers with either
prose or a set of tool calls; the browser runs them, appends the raw results,
and posts again. The server stays stateless and same-origin discipline is
untouched, so the guards in "What stays" 1, 2 and 5 still sit on the network
path exactly where they are today.

The backend seam widens from `(system, messages)` to
`(system, messages, tools) -> { reply, tool_calls, meta }`. Both backends
support tool calls natively (`tools` / `message.tool_calls` on ollama's
`/api/chat`; the Messages API's `tool_use` blocks). The `BACKENDS` map, the
default, and the unknown-value error are unchanged.

Tools, named as facts and no more:

| tool | what it is |
|---|---|
| `fetch(path)` | any same-origin path: `/cluster/state.json`, `/cluster/workspaces.json`, `/cluster/actual.json`, their `*.sample.json` fallbacks, `/api/autolab/nodes`, `/api/autolab/<node>/<path>`, `/api/forge/requests`, `/api/guide` |
| `switch_view(view)` | the screen; `nodes` / `workspaces` / `autolab` |
| `show_image(url)` | put a picture in the conversation |

A single `fetch` rather than one tool per endpoint is deliberate: an
enumerated tool per endpoint is a smaller allowlist wearing a different hat.
The paths above are told to the agent as a list of facts; the passthrough
decides what is actually reachable, and its refusals reach the agent verbatim.

Two rules for this item, and they are how it is judged:

- **Every tool result goes back to the agent raw** — the response body as
  text, the HTTP status, the error string. No parsing, no key extraction, no
  reshaping. If a tool fails or is unknown, *that fact* is the result the
  agent reads; code never silently drops an action (today
  `main.ts` drops unknown actions and `viewSwitcher` ignores unknown views).
- **Judgment must leave agdevworld's code, not merely move.** After this item
  no line of `main.ts`, `chatPanel.ts` or `server.mjs` decides whether the
  agent answered well, what it meant, or what it should have read next. If a
  reviewer can point at such a line, the item is not finished.

Resource guard: a cap on tool rounds per chat request (start at 8) and the
existing wall-clock. Stated to the agent as a budget fact, recorded in
`assistant.run.v1` as `tool_rounds`.

Lenient alternative, in agforge's pattern: the inline
`{"action": …}` object keeps working if the model emits one, because two
natural ways to act is less forced than one. What is deleted is the *mandate*
— see work item 2. This also covers the risk that the default local model
turns out not to tool-call well; if that happens it is F-worthy, not a defect
to patch by re-mandating the format.

### 2. Delete the prompt-side commands

`ROLE_PROMPT` in `server.mjs` shrinks to facts: what this is, what the tools
reach, where the card is. Specifically gone:

- "Answer concisely and in plain text", "say you do not know".
- The exact-JSON-object format contract for view switching, "only when
  asked", "never mention or explain the JSON".
- The image contract, and `desire` "must contain no double quotes, braces or
  backslashes".
- "You must never invent a price, a duration or a capability the card does
  not claim." Replaced by one line naming the card, as agforge did — F3 there
  showed a pointer carried the same information as the prohibition.
- The fixed sentence substituted when no cluster context is available.

The system prompt keeps being assembled from role + card (an unreadable card
still yields a placeholder — it is a fact about the environment, not a
judgment about the agent).

### 3. Tear down the digest walls

`summarizeClusterContext`, `summarizeNode`, `summarizeWorkspace` and
`summarizeJob` exist so the model never sees raw JSON — code choosing what the
agent may know, including the deliberate withholding of `facts_raw`. Delete
them as the agent's input path; the agent reads the snapshots itself with
`fetch`.

Consequences accepted, not mitigated: more tokens, and a small local model
that may cope worse with raw envelopes than with the curated lines. That
comparison is one of the results this turn exists to produce.

The panel rendering keeps whatever it needs to draw boxes (`views.ts`,
`PanelGridScene`) — the browser is a machine drawing a grid, not an agent.
The envelope validators in `clusterState.ts` / `autolabState.ts` drop to the
minimum that keeps rendering from crashing; they stop being gatekeepers that
throw on an unrecognised `schema` / `kind`.

### 4. "Ask agent" stops writing the user's question

`main.ts` composes both the question text and the context digest in code —
the user writes nothing and the agent receives a pre-chewed brief. Reduce to
the identity of what was clicked, phrased plainly (e.g. `the autolab job
<name> on node <node>`, `iteration <iter>`), handed to the agent as an
ordinary user message. What to look up next is the agent's call.

The one exception is the iteration summary text: it is already prose, written
by a Claude run on the node, and the popup shows it unabridged. When the user
asks about an iteration they are looking at, that text rides along verbatim.
Passing along what is on screen is not a digest.

### 5. Let the agent own the agforge exchange

Today the browser runs a fixed image pipeline: POST, `request_id` must be a
string, poll every 3 s, anything but `done` fails, take the first artifact
whose `kind === "image"`. agforge's turn1 made its result shape free-form and
unvalidated (its F5), and its plan named this turn as the moment that contract
dissolves.

The agent calls `/api/forge/requests` itself through `fetch`, reads whatever
comes back, decides whether to poll again, and calls `show_image(url)` when it
has one. The browser's remaining job is to display, and to say plainly when a
picture will not load. The poll deadline survives as a resource guard (item 4
of "What stays"); it is told to the agent, not enforced against it.

### 6. Shrink the guides

`assistant/GUIDE.md` and `README_DEV.md` to the target shape: paths,
commands, endpoints, one line each. Gone from GUIDE.md: the enumerated
capability prose and the whole "What it cannot do" contract. Kept: the price
and latency figures — they are facts, and the Entrance Guide obligation
(devpolicy/policy.md) depends on the agent being able to quote them.

README_DEV.md keeps a short "Safety devices" section naming the five
survivors above and why they differ in kind, as agforge's README now does.

### 7. Loosen `scripts/fetch-cluster-state.mjs`

Delete "reply with only the download URL" from the prompt and accept prose:
find a URL in whatever cagent says. Delete the envelope validation that
currently gates persistence. Keep the code-chosen output path and the atomic
0600 write (condition 2 above).

### 8. Build and container check

There is no test suite in this repo, so the deterministic shell is:
`npm run build` (tsc + vite) green, then
`docker compose up --build -d web assistant` and `curl -I http://localhost:8090/`
per README_DEV.md. Do not add tests that re-assert the removed contracts in
another form.

### 9. Live check — did the capability survive?

The point of the turn. Everything below worked before; run each on the ollama
default with the service restarted on the new code, and repeat the starred
ones on `claude` for contrast.

- "which nodes are drifting?" — a question answerable only from a snapshot
  the agent must now fetch itself \*
- "show me the autolab" — the view changes
- "draw me a red lighthouse at dusk" — a picture appears in the conversation \*
- click a node → **Ask agent** → an explanation that is actually about that
  node (the digest that used to guarantee this is gone)
- click an autolab job → **summary** → **Ask agent about this iteration**
- "what can you do?" and "how much does an iteration summary cost?" —
  answered from the card, figures intact
- one deliberately unreachable thing — ask it to start an autolab mission, or
  to read an iteration's raw evidence. Does it report honestly what came
  back, or invent an outcome?
- assistant stopped → the chat still says the assistant is offline

Then `report.md` with: which of these still work, what the agent did
differently without the rails, what regressed, tokens/latency/cost against the
current behaviour, and — most valuable — where a removed guard turns out to
have been holding something up. Regression is a finding, not a defect to
patch in the same turn.

## Facts worth not rediscovering

- GUIDE.md is re-read from disk per chat request, so card edits need no
  restart — but it is COPYed into the image, so a *container* does need a
  rebuild unless bind-mounted. `server.mjs` always needs a restart.
- The default backend is deliberately weak (`glm-4.7-flash` on the local
  ollama) so that wording gaps surface as behaviour. Whether it tool-calls
  reliably is unknown at the time of writing and is itself a result.
- Compose passes an empty string for unset variables, so the code reads env
  with `||`, never `??`. Preserve that when touching config.
- `public/cluster/*.json` are git-ignored live snapshots with a
  `*.sample.json` fallback; the Docker build copies whatever is in `public/`
  at build time.
- Dev: `npm run dev` (vite proxies `/api` to :8091). Prod-style: `docker
  compose up --build -d web` on :8090, assistant on :8091.
- agforge's F1: a guide that points at more doors needs the grants widened in
  the same commit, or the run dies with no output. F2: an agent given an
  unvalidated channel can leave a caller polling forever — expect the
  equivalent here and record it rather than guarding it.

## Out of scope

agautolab (its own turn), authentication/identity for the assistant, moving
the runtime into a sandbox, persistent chat memory across page loads, and any
change to agforge's or the autolab gateway's own surfaces.
