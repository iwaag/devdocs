# refactor p1 — the autolab work record moves out of Plane and into Zulip

## What this phase set out to prove

That one autolab development request can be planned, executed, revised,
resumed and accepted using Zulip as its work record, with no Plane read or
write on that path, and that the result is simple enough to say whether more
of Plane should go.

It is proven. A request was taken through the whole shape live on 2026-09-10
— planned, started, executed, **replaced** with unfinished work still in it,
resumed, reported and accepted — with autolab unable to reach Plane at all,
because it no longer imports the client.

## What changed

### Step 1 — the conversation *is* the record

`agautolab/mission.py`, the Plane mirror of one chat topic, is deleted.
`worklog.py` holds the same model read and written out of Zulip:

| was | is |
|---|---|
| a Work keyed `<channel>/<topic>` | the `workplan-` topic itself |
| a Sub-Work `…#<N>` | the `workrun-` topic itself |
| `description_html` on an issue | a visible Markdown post in that topic |
| a Plane state group | a `[selfnote][state]` note |
| an issue comment | a visible `## Result` post where the task ran |

Prose is what a human reads, so the plan and each task's description are
ordinary posts. Everything a program must answer without guessing is a
selfnote — machine-to-machine, hidden from every chatlog, never counted as
somebody speaking, so writing one never buys a run:

```
[selfnote][mission] <project slug>       in a workplan- topic
[selfnote][task] <mission id>#<serial>   in a workrun- topic
[selfnote][doc] <message id>             which post is the current document
[selfnote][state] <word>                 where the work has got to
[selfnote][replaces] <anchor id>         the work this one replaced   (p2)
```

**Identity is a message id.** The `[mission]` note's own id *is* the mission;
the `[task]` note's own id is the task. `ZulipClient.message` answers with
the conversation an anchor is in *now* — through a resolve, a hand rename, a
move to another channel — and `None` for a deleted message. Names follow from
the id (`work-m5512`, `workrun-task3-m5512`), so no second mission can want a
channel name and a deleted origin is **absent** rather than whatever took its
name.

`project_init.py` no longer creates or finds a Plane project;
`project_archive.py` lost `archive_plane_project`; `mission_done` counts
conversations. `agag/__init__.py` stopped re-exporting `PlaneConfig`, which
had made `import agag.selfnote` load the Plane HTTP client — so no agent
could honestly claim not to reach Plane.

### Step 2 — replacement as a conversation operation

Re-planning revises a mission in place. `replace.flag` beside `plan.md` does
the other thing, for when the request itself was wrong: **retire this
conversation and open its replacement.** Four operations, in an order that is
the workflow —

1. unfinished tasks cancelled and resolved;
2. the mission marked `replaced`, its `work-m<id>` channel archived;
3. its conversation renamed to `✔ retired-<name>-m<id>` — out of the
   `workplan-` vocabulary **and** resolved, so neither a topic filter nor a
   sweep can reach it;
4. the replacement opened under the name step 3 released, with its own
   mission anchor, `[selfnote][replaces]`, and a visible post saying what it
   carries forward and what it drops.

Step 4 cannot precede step 3: Zulip has one topic per name in a channel, so a
replacement created first would **merge into** the conversation it was meant
to replace. Finished tasks are carried forward **by reference** and never
re-created as completed rows.

`ZulipClient.rename_topic` (pyagag) is the one shared addition — the general
form of what `resolve_topic` always was.

### Step 3 — observation and acceptance

`agentroom/autolab.py` reads autolab's record where it now lives, reproducing
the format the way this room already reproduces every other agent's. A task
belongs to a mission by the **anchor id its note names**, never by the name
its root note carries — which is what keeps a reused display name from
handing one request another's work, and what makes a `replaced` mission a
different request that is never closed alongside its replacement.

Accepting a request now writes the human's word where the work happened:
`[selfnote][state] accepted` on each finished task, `done` on the mission,
before anything is resolved. The three things stay distinguishable:

| what happened | who says it | how it is written |
|---|---|---|
| the run did the work | the run and the developer, in the task's topic | `[state] completed` |
| a person accepted it | the operation room's button | `[state] accepted`, `[state] done` |
| the conversation is over | the same button, afterwards | Zulip's `✔ ` |

The Plane path stays for forge, narrowed to what forge writes: a `[work]`
note naming a project and an issue. A bare `<issue id>` note — autolab's old
shape — is now *reported* as a format nothing writes any more.

### Step 4 — deployed, demonstrated, and one defect found

Deployed to `agstudio`: the autolab listener, the agentroom relay, the
agdevworld web container, and autolab's re-posted introduction.

## Evidence

### The live demonstration (2026-09-10, `#pj-refactorp1`)

A fresh project channel, a fresh Gitea project, and a two-task request for a
`wordcount.py` tool.

| step | what happened |
|---|---|
| plan | `workplan-wordcount` → mission **m5702**, `work-m5702` with two task topics; the plan is a post in the topic |
| start | `[state] started`, and each task waits for a post in its own topic |
| execute | task 1 wrote `main/wordcount.py`, the developer agreed, the run posted its `## Result`, marked the task `completed`, resolved the topic, pushed `main` to Gitea and filed the report in `devlog` |
| **replace** | the developer scrapped the rest; **m5702 retired** to `✔ retired-workplan-wordcount-m5702`, `work-m5702` archived, task 2 cancelled, and `workplan-wordcount` became mission **m5735**, carrying task 1 forward by reference |
| gate | posting in the replacement's task 2 first answered `Please complete previous work (task 1 of m5735)` — handler-side, **no agent run bought** |
| resume | task 1 of m5735 added `--json`; task 2 added its test; both committed, pushed and filed |
| accept | `POST /complete` on the operation room: mission done, `work-m5735` archived, the conversation ✔ |

Read back live through the anchors alone, after everything closed:

```
m5702: state='replaced' at pj-refactorp1/'retired-workplan-wordcount-m5702'
       replaces=None title='Add a wordcount command-line tool'
m5735: state='done'     at pj-refactorp1/'workplan-wordcount'
       replaces=5702 title='Add a --json flag to wordcount.py'
```

The retired mission kept its identity and its plan under the name it was
moved to; the replacement wears the freed name and says what it replaced.
Neither could be told apart by name alone, which is the whole point.

Git holds the durable half: `main` at `26e2a3b` (three commits, pushed), and
`devlog/m5702-…/task-1`, `devlog/m5735-…/task-1`, `devlog/m5735-…/task-2`.

Cost: 10 agent runs, ~242 s of run time, **$1.23** total.

### Plane unavailability

Verified as an **import-graph property**, which is stronger than a rejecting
client — the code cannot call Plane because the module is never loaded:

```
$ uv run python -c "import agautolab.zulip_listener, agautolab.mission_done,
  agautolab.project_init, agautolab.cli, sys;
  print([m for m in sys.modules if m.startswith('agag.plane')])"
[]
```

on the *deployed* venv, and `tests/test_plane_independence.py` asserts the
same in CI, including a subprocess that imports the whole package.

### Controlled tests, distinguished from the live run

`agautolab/tests/test_replacement.py` covers the four cases the plan named,
against a realm fixture that renames topics and archives channels the way
Zulip does: a **reused topic name**, a **late callback** to retired work, a
**restart** through the real `agag.zulip.sweep_topics`, and a **deleted
origin**. `agentroom/tests/test_scope.py` adds the same four seen from the
operation room.

### Suites and build

```
pyagag      543 passed
agautolab   237 passed
agentroom   236 passed
agdevworld  tsc clean; vite build ok; web container rebuilt on :8090
```

### The defect the live run found

The operation room accepts with the **human's** credential — that is what
acceptance means, and it is the credential that already posts in these
channels. Both readers took a conversation's state notes only from the
record's own author, so the `done` note the room wrote into the mission's own
topic was invisible to the record it was written into: the next preview read
a finished mission as having no task, and a completed operation reported
itself `partial`.

`accepted` and `done` are read from any sender now; every other state stays
the agent's own report of its own work. Both fixtures had been writing the
acceptance *as autolab*, which is why no test caught it — the room's fixture
writes as the Developer now, and six tests fail without the fix.

This is the only defect the live run found, and it was found in the last
minute of the phase. It is worth recording that the whole record-level
rewrite — identity, retirement, replacement, the gate — ran correctly the
first time, and what broke was the seam between two credentials.

## Remaining limitations

- **Old autolab work is unreadable, deliberately.** `work-<plane label>`
  channels (`work-s4-13`, `work-g-21`, …) carry no `[mission]`/`[task]`
  notes; nothing was migrated, as the plan permits. In-flight work in
  `pj-studyrealworld` predating the deployment is stranded and was left
  alone.
- **A related topic is still resolved when its mission is blocked.** The
  conversation closes and the work is not accepted — the distinction working
  — but archiving the channel afterwards takes the unfinished task's topic
  with it. Pre-existing behaviour, not this phase's proof.
- **A channel accumulates retired topics.** A second replacement of the same
  request leaves `✔ retired-<name>-m<old>` beside the first. Distinct,
  because the label is an anchor id; nothing prunes them.
- **The demonstration was a direct autolab request.** Front supervision and
  forge delegation were not added to the live scenario, as the plan allowed.
- **agautolab1** (the other node) runs older code. It has no listener, so
  nothing there reads the new record, but it was not updated.
- **`planeState.ts` and the `tasks` view are untouched** — the older
  Plane-issue dispatch through the autolab *gateway*, not the listener.

## Should more of Plane go?

**Yes, and the evidence is the size of what was removed rather than the size
of what replaced it.**

Three things carry the argument.

**One system instead of two that had to agree.** Understanding a mission used
to mean reading a chat topic and a Plane issue and trusting they still said
the same thing. `refactor` p1 step 1 deleted a whole mirror — read-back,
reconciliation, predecessor checks, result writes, project creation, project
archival — and replaced it with four selfnote lines and the rule that the
newest state wins. Nothing was reimplemented in a new place; it stopped
needing to exist.

**A message id is a better identity than an issue id, for this work.** Plane
gave autolab an identifier that lived outside the conversation, which meant a
revised mission kept one and a replaced mission needed one nobody could mint.
The anchor id is minted by the act of writing the note, is unique by
construction, survives every rename, and is *absent* when the work is
deleted. Replacement — the operation this system could not previously express
at all — became possible one step early, because nothing else could have
carried the record.

**The reader got simpler too, not just the writer.** The operation room's
discovery lost the `external_id` lookup, the channel-name-to-project rule,
the Plane project scan and the state-group mapping for autolab's half, and
gained one module that reads notes out of topics it was already reading.
Where a preview once had to say "no Plane credential, so the Work states
below are what the relay could not read", an autolab request now previews and
closes completely with no Plane at all.

Against that: the counting rule now exists twice (`agautolab.mission_done`
and `agentroom.autolab`), as the note format does, because the room depends
on no agent's package. That is the standing cost of this architecture and it
was already being paid for `[rootchat]`, `[served]` and `[work]`. It is a
small, stable surface — five tags and one gate — and duplicating it is
cheaper than the two-system agreement it replaced.

**Recommended next**: forge's plan and toolset storage, which is the largest
remaining consumer and the one whose shape most resembles what was just
replaced. `cagent`'s change registration and `nctl`'s agent registration are
smaller and can follow. Nautobot's Plane identities should be decided last,
because they are declarations about the system rather than records of work.

**This phase is not Plane removal**, and nothing here should be read as
implying Plane is gone: `agag.plane` remains, forge and cagent still write to
it, and the `tasks` view still dispatches from it.
