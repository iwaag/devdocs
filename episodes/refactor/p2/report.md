# refactor p2 — forge's work moves out of Plane and into Zulip

## What this phase set out to prove

That autolab can request an asset from forge, receive it, incorporate it into
a project and complete the request **with no Plane on that path** — and that
what is left is simple enough to say what should follow.

It is proven. One development request went through the whole shape live on
2026-09-10 — planned, delegated, generated, delivered, committed, pushed and
accepted — with neither forge nor the operation room able to reach Plane,
because neither imports the client and forge holds no credential.

## What changed

### Step 1 — unfinished work stays reachable

`refactor` p1 left a limitation standing: a mission blocked on an unfinished
task was correctly refused acceptance, and its plan topic was resolved, its
unfinished task's topic was resolved, and `work-m<id>` was archived on top of
both. Completion had exactly one dependency rule — anything blocked keeps the
*selected request's* conversation open — and every other target was
independent.

`close.dependents()` names what a work record decides: its own conversation,
its children's, and the `work-` channel named after it. `_hold_dependents()`
keeps those targets open when the record is blocked, and `_apply()` applies
the same rule to a record whose acceptance **write fails** — the same fact
learned a moment later, actionable in one pass because the plan's fixed order
runs the records first. Independent finished branches still close.

### Step 2 — the conversation *is* the record

`agforge/plane.py` and `works.py` are deleted. `record.py` holds the same
model, read and written out of Zulip:

| was | is |
|---|---|
| a Plane issue keyed `<channel>/<topic>` | the `assetplan-` topic itself |
| `description_html` | a visible Markdown post, named by `[selfnote][doc]` |
| the `[TOOLS] …` description footer | `[selfnote][tools] toolset-image, …` |
| `[selfnote][work] <project>/<issue>` | `[selfnote][assetrun] <request id>` |
| an issue comment carrying `[S3KEY]` | `[selfnote][result] <object key>`, in both conversations |
| a Plane state group | `[selfnote][state]`, newest wins |
| — | `[selfnote][replaces] <message id>` |

**Identity is a message id.** The `[asset]` note's own id *is* the request;
the `[assetrun]` note's own id is the run. Three things follow, and they are
why the plan asked for it:

- the run topic is `assetrun-<stem>-a<request id>`, so a requester's stem can
  be reused without two requests merging;
- the delivery follows the **anchor** home, not the `[rootchat]` note's name,
  so a result lands in a conversation that has since been resolved, renamed
  or retired — under the name it wears now, which is what keeps a post from
  opening a twin beside a ✔ topic;
- a deleted origin is **absent**: the run says "the request this topic runs
  (a5814) is gone" instead of delivering to whatever took the name.

`agforge.retire` is the one retirement route: the request is marked `retired`
*before* anything moves, both conversations become `✔ retired-…-a<id>`, and
the stem is released. Old-first is not a preference — Zulip has one topic per
name in a channel, so a replacement created first would merge into what it
replaced.

### Step 3 — one job, one generation, one delivery

The asynchronous shape (submit → notifier callback → collection) mostly
survived step 2 unchanged, because the run's anchor id already names the
workspace: a replacement has its own run topic and therefore its own
workspace, so it can neither collect the old attempt's job nor overwrite its
outputs, and a collecting run keeps the `plan.md` and `tools/` its job was
submitted with.

What was missing: **a callback can arrive twice**. The notifier's callback is
an ordinary post, and an ordinary post starts a run — so a retried callback,
or a restart re-reading the same mention, would find no `watching.json`, read
as a fresh trigger, submit a *new* ComfyUI job and deliver a second time for
one request. `collected.txt` beside the workspace is what makes the second
callback a no-op. On disk (a callback outlives the process), matched on the
id and never on the notifier's wording, and written before the delivery.

### Step 4 — the operation room reads it where it is

`agentroom/forge.py`, the companion of `autolab.py`, reproducing the format
rather than importing agforge. **The lifecycle is forge's own**, and that is
the part that mattered: applying autolab's parent/child counting rule would
have called a request with no run "blocked, it has no task" and left a
request that failed once and succeeded twice blocked forever by its first
child. A request is finished when **something has been delivered**. A run
still `pending` on ComfyUI blocks — which is what makes step 1's hold keep an
unfinished generation reachable.

Plane left the relay with it: `PlaneBoard`, `PlaneReader`, `_plane_works`,
`PlaneOps`, `plane_factory`, `AGENTROOM_PLANE_ENV` and its plist entry,
`status.plane`/`gaps.plane` in the payload and in the frontend. The
`[selfnote][work]` note went too — nothing writes it, and no reader was kept
for the old format.

### Step 5 — deployed, demonstrated, one defect

Deployed to `agstudio`: the forge listener, the agentroom relay (booted out
and bootstrapped, because `kickstart -k` does not re-read a changed
`EnvironmentVariables`), the web container, and forge's re-posted
introduction.

## Evidence

### The live demonstration (2026-09-10, `#pj-refactorp2`)

| step | what happened |
|---|---|
| plan | `workplan-splash` → mission **m5790**, two tasks |
| delegate | autolab opened `assetplan-refactorp2-logo`; forge answered `recorded a5814 … (toolsets: toolset-image)` / `posting in assetrun-refactorp2-logo-a5814 starts it` |
| generate | forge ran it and delivered into the plan topic with a presigned URL and `[S3KEY] files/2026-09-10/51afa927…zip` |
| callback | that delivery named autolab, which resumed task 1, downloaded the zip, verified `512 x 512` PNG, committed `main/assets/logo.png` (`3afe543`), pushed and filed the report |
| task 2 | `main/index.html`, approved in the topic, committed (`645ea17`), pushed and filed |
| accept | the operation room: **two work records, no Plane row, no gaps**; `m5790 is done; accepted m5790#1, m5790#2` and `a5814 is accepted; its asset is files/2026-09-10/51afa927…zip`; `work-m5790` archived, both forge topics ✔, the request ✔, `partial: False` |

Read back through the anchors alone, after everything closed:

```
a5814: state='accepted' at agforge-agstudio1/'assetplan-refactorp2-logo'
       tools=['toolset-image'] replaces=None
       results=['files/2026-09-10/51afa9273ce043769fbbce1519f8c011.zip']
       run topic='assetrun-refactorp2-logo-a5814'
run r5820: state='delivered' request=5814 results=[…the same key…]
```

The artifact was inspected: a flat geometric mark, one orange accent on
black, 512x512, in the repository and rendered by the page.

**Cost**: 10 agent runs, ~335 s, **$1.29**.

### Plane unavailability

Live isolation *and* an import-graph property, on the deployed venvs, while
those processes were serving the run:

- `agforge/.local/plane-credentials.env` — removed; there is no credential.
- importing every forge and relay entry point loads no `agag.plane`.
- `AGENTROOM_PLANE_ENV` is absent from the running job's environment.

`tests/test_plane_independence.py` (agforge) and `tests/test_no_plane.py`
(agentroom) assert the import graph in CI, each including a subprocess that
imports from scratch.

### Controlled tests, distinguished from the live run

The live run used **image** generation, which is synchronous. Everything
about pending jobs, restarts, repeated callbacks and retirement is fixtures:

- `agforge/tests/test_record.py` — identity through a rename, a deleted
  anchor, plan and toolsets round-tripping, an over-long plan refused rather
  than silently truncated, retirement releasing the stem, a replacement
  taking the freed name, and a late result belonging to the retired request.
- `agforge/tests/test_assetrun_topic.py` — pending → collection, a restart
  while waiting, a repeated callback submitting and delivering nothing, a
  person asking again still running, a collecting run keeping its own plan,
  and a replacement unable to collect the old attempt's job.
- `agforge/tests/realm.py` — a Zulip that renames, resolves and deletes, so
  nothing here passes on a record that only works while nothing moves.
- `agentroom/tests/test_forge.py`, `test_closing.py`, `test_close.py` — the
  reader, the lifecycle, the hold, and acceptance written by the **human's**
  identity rather than the agent's.

### Suites and build

```
pyagag      543 passed
agforge     221 passed
agautolab   237 passed
agentroom   267 passed
agdevworld  tsc clean; vite build ok; web container rebuilt on :8090
```

### The defect the live run found

The record read back as `delivered` after a person had accepted it. The room
accepts with the **Developer's** credential, and forge's own reader took
state notes only from the record's own author — so forge would have gone on
calling an accepted request merely delivered and offered to work on it again.

`agforge.anchor.EXTERNAL_STATES` is p1 step 4's answer applied on this side.
What is worth recording is *why it was missed*: the agentroom half was
written with `EXTERNAL_STATES` from the start, because p1 had recorded the
lesson — and the lesson was filed as a fact about that reader rather than as
a fact about the rule. **A convention with two readers needs the rule in both
of them**, and only one was carrying it.

## Remaining limitations

- **Old forge work is unreadable, deliberately.** Existing Plane issues carry
  no `[asset]`/`[assetrun]` notes, nothing was migrated, and no reader was
  kept for the old `[work]` note. Any request in flight at deployment time is
  stranded.
- **The asynchronous path and retirement were not exercised live.** Both are
  fixture-tested and reported as fixtures.
- **`collected.txt` is per workspace**, so deleting the workspace forgets
  which jobs were collected. The plan does not require rebuilding render
  state after local files are deleted.
- **Retirement is a command, not a conversation.** A person (or an agent with
  a shell) runs `agforge.retire`; forge does not retire a request because
  somebody asked it to in the topic.
- **A channel accumulates retired topics**, distinct by anchor id, unpruned —
  as p1 recorded for autolab.
- **The acceptance was made through the completion route**, which is what the
  browser's button calls, rather than by driving the Phaser canvas. The board
  itself was inspected in the browser.
- **Front supervision** was not part of the demonstration, as the plan
  allowed.
- **agautolab1** runs older code and no listener.

## Remaining Plane consumers

Plane is **not** gone, and nothing here should be read as implying it is.
What still uses it:

| consumer | what for |
|---|---|
| `agag.plane` | the shared client, still shipped by pyagag as an optional dependency |
| **cagent** (`pj-clusterintent`) | change registration |
| **nctl / Nautobot** | agent Plane identities — `nctl agents observe` still prints `agforge … plane=yes`, which is a *declaration about the system*, not something forge uses |
| **agdevworld's `tasks` view** | `planeState.ts` and the autolab **gateway** route, the older Plane-issue dispatch — untouched, as the plan reserves it |
| the Plane service itself | still deployed, serving the above |

What is gone from Plane's consumers: autolab's work record (p1), forge's work
record (p2), and the operation room's completion (p2). The two agents that
*do work* keep no ledger there any more.

## What this phase says about the architecture

Three things, all of which p1 predicted and this phase can now confirm from a
second instance rather than one.

**The removal is bigger than the replacement, again.** `plane.py` and
`works.py` — project routing, a fallback project, a label, an `[AUTO]`
marker, a `[TOOLS]` footer, issue comments and state transitions — became
notes in the conversation and one module that reads them. The relay lost a
credential, a protocol, a reader and a whole discovery branch. Nothing was
reimplemented in a new place; it stopped needing to exist.

**A message id is a better identity, and this time it bought something p1
could not use.** For autolab the id made *replacement* expressible. For forge
it makes **the run topic's name safe**: `assetrun-<stem>-a<id>` means a
requester can ask for "the logo" twice and get two requests that cannot merge
or collect each other's jobs — and it makes the delivery follow a renamed,
resolved or retired conversation home, which a `[rootchat]` name never could.

**The counting rule was not the shared part.** The tempting move at step 4
was to reuse `reason_not_finished` for both agents: same room, same shape of
record, same button. It would have been wrong in two directions at once.
What is genuinely shared is the *note format* and the rule that a person's
word is read from anybody; what is not shared is what "finished" means. This
phase's cost is that both those shared rules now exist in four places (each
agent and the room, twice) — and its one defect was exactly a place where one
of the four did not have it.

**Recommended next**: `cagent`'s change registration, which is the largest
remaining consumer and the last one that is a *record of work* rather than a
declaration. The `tasks` view and the gateway route can then be retired or
rebuilt on the conversations, and only after that is it worth deciding what
Nautobot's Plane identities are for.
