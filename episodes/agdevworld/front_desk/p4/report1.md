# Step 1 — generalizing the completion target

agdevworld `agentroom` only; nothing in agfront, agautolab, agforge or
pyagag changed, and nothing writes outside the fixtures. The Front Desk
routes still answer, now as one caller of a shared operation.

## What exists now

**The root is a conversation, `(channel, topic)`.** `closing.discover(topics,
root, …)` and `close.Closer.plan/close(key, …)` take that key; the Front
Desk's id is translated by `Closer.desk_key(ident)` and nothing else in the
completion path knows the desk exists. Two new relay routes carry the shared
operation — `GET /complete/plan?channel=…&topic=…` and `POST /complete
{channel, topic, fingerprint}` — plus `GET /complete/history` for what this
relay carried out (per request, or all). `/frontdesk/<id>/close-plan` and
`/close` are thin wrappers over them. The payload schema is
`ag.completion.v1`: `root`, a `scope` block (below), and everything p3 had.

**A conversation's kind is its name; its owner is its notes.** `classify()`
reads the writers' own prefixes: `desk`, `front`, `routine-run`,
`routine-standing`, `workplan`, `workrun`, `assetplan`, `assetrun`, `intro`,
or plain `topic`. The five *request* kinds may be roots. The two
*execution* kinds — and any unknown topic carrying a root note — answer with
their **parent** and one blocked action: selecting a task topic never closes
the plan and its siblings. The two *retiring* kinds (a routine's standing
request, an agent's introduction) are never roots or targets, because a ✔
there means something else.

**Ownership is one rule for every root** (`_ownership`, a fixed point):

- A topic whose root notes all name this request, or something it owns, is
  owned. One naming anything else is *anchored to another request* and
  excluded with that note's message id — p3's reused-plan rule, now for any
  request kind, not only another desk.
- **An execution topic's structural home decides.** autolab anchors a
  `workrun-` to its `workplan-`, forge an `assetrun-` to its `assetplan-`;
  a root note by anyone else in such a topic is a *visitor's* — Front,
  posting there on behalf of what it was serving. Visitors are kept on the
  node as `visitors`, shown and not obeyed. Without this the live realm
  deadlocks (below).
- A topic with no root note is owned only when a link from something owned
  reached it — unless it is itself a request (another Front conversation,
  another run, another agent's plan), which no served note can claim:
  *another request: a run of routine publish*.
- The root's own parents are *lineage*: a note naming them is not a note
  naming a stranger. That is what lets a `workplan-` be completed on its
  own while its desk conversation stays open.

**The walk reads in both directions now.** Two additions to
`related_topics`, both cached reads and both reported:

- *Upward, once.* Every reached topic's homes that nothing has read are
  read; a home joins only if a sweep then links it from something reached.
  This is how a plan Front never served (it served the task) is found: the
  task's note names the plan, the plan's note names the run.
- *A mission's own channel.* autolab's plan topic carries no note naming
  its task topics — each task names the plan, not the other way round — so
  `discover` reads every topic of `work-<label>` for each owned autolab
  mission (`external_source == agautolab`; forge's Works have no channel)
  and walks again. A topic there with no note of its own still joins nothing
  and keeps the channel from being archived.

**A routine run's scope is the run.** `scope.routine` carries the name,
stamp, standing topic and the previous run the fire line names;
`scope.context` lists the standing request and that previous run as
*mentioned, not owned*, and the description says *"the standing request,
the schedule and earlier runs are not touched"*. A previous run reached by a
served note is excluded as another run; the standing request as a retiring
kind. The pre-p7 shared `front-routine-<name>` topic classifies as an
ordinary `front` conversation and is treated as a whole.

## Verification

`agentroom` `uv run pytest -q` → **246 passed** (30 new in
`tests/test_scope.py`; the p3 tests ported to the key API with two
reason-strings updated). Pinned on fixtures: every kind's classification;
each supported root (Front conversation, routine run, `workplan-`,
`assetplan-`, an unknown topic without a note); task and run topics naming
their parent and closing nothing; the standing request and an introduction
refused; nested delegation owned five levels deep from a run; a plan
anchored to another Front conversation excluded with evidence; each request
kind excluded when reached by a served note; a previous run reached by a
link still excluded; a resolved intermediate walked through; a task with no
note neither swept in nor archived over; a forge plan owned by its note
whatever reached it; the re-run task shape from three roots; the upward
read; `parse_key`; per-request history; and the three routes over HTTP
(400, 409, and an execution topic with nothing to approve).

**Live, read-only** (Developer Zulip credential, admin Plane key, `topics={}`
so every read was paid; `discover` has no write path):

| root | kind | found |
|---|---|---|
| `front-routine-study-realworld-2026-09-08T16:47Z` | routine run ✔ | `workplan-investigate-realworld` ✔, `workrun-task1-s4-3` ✔, S4-3/S4-4 completed, **`#work-s4-3` archivable** — one ready action |
| `front-routine-publish-2026-09-08T16:57Z` | routine run ✔ | plan ✔, `workrun-task1-s4-5` ✔, `workrun-rerun-task1-s4-5` ✔ (visitor: run 17:09Z), S4-5/S4-6 completed, **`#work-s4-5` archivable** |
| `front-routine-publish-2026-09-08T17:09Z` | routine run ✔ | only itself; the re-run task *excluded: anchored to another request (workplan-publish-realworld)* with autolab's note as evidence |
| `pj-studyrealworld/workplan-publish-realworld` | workplan ✔ | parent run 16:57Z named and kept open; both tasks owned, `#work-s4-5` archivable |
| `work-s4-5/workrun-rerun-task1-s4-5` | workrun | not closable: *of workplan-publish-realworld … also served on behalf of run 17:09Z*; both parents listed, structural first |
| `pj-studyrealworld/workplan-create` | workplan, open | no Plane Work matches its `external_id`; one ready action (the topic) |
| `agforge-agstudio1/assetplan-red-apple` | assetplan ✔ | parent `front-comfy-command-relay`; no `assetrun-` exists for it on the realm (pre-p8), so no Work is found — reported as nothing, not guessed |
| `front/routine-ghtrends` | standing request | refused: *a ✔ on a standing request retires the routine* |
| `front-desk-20260908-164810` | desk ✔ | unchanged from p3: G-17/G-18 done, two task topics, `#work-g-17` already archived; every action `done` |

4–14 Zulip calls per root; no Plane errors; no gaps.

## What the live realm taught

**One task topic, two root notes.** `workrun-rerun-task1-s4-5` carries
autolab's note naming `workplan-publish-realworld` (planned for the 16:57Z
run) *and* Front's note naming the 17:09Z run that asked for the re-run.
p3's rule — any home naming another request disqualifies — would have
excluded the task from **every** root: from the plan because of Front's
note, from the 17:09Z run because of autolab's, from the 16:57Z run because
17:09Z is a stranger. Nothing could ever have closed it. The structural-home
rule is the answer, and the fixture `rerun_realm` reproduces the shape from
three roots.

**Front serves tasks, not plans.** A run's served notes named the task
topic and not the plan; without the upward read the plan was invisible and
the first probe reported the task as anchored to a stranger. Reading a
reached topic's home is the only way to learn what that home's own note
says.

## Left for the next steps

- The frontend still calls `/frontdesk/<id>/close-plan`; `ClosePlan`'s
  `conversation` field is no longer sent. Step 2 moves the panel onto
  `/complete` and the `scope` block (parents as navigation, visitors,
  context, the routine sentence).
- `agentroom/README.md` documents the new routes; `devenv.md` follows in
  step 3 with the deployment.
- Live candidates for step 3 with one real ready action each: the
  study-realworld run (`#work-s4-3`) and the publish run 16:57Z /
  its plan (`#work-s4-5`). `workplan-create` is an open plan with no Work —
  a topic-only completion.
