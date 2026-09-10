# refactor p1 step 3 — observation and acceptance, without Plane

## What the step asked

Adapt the operation room and the completion traversal to discover autolab's
new work relationships and outcomes without Plane; display the plan, tasks,
results and replaced work well enough for a human, reusing the existing UI;
keep execution success, human acceptance and conversation closure
distinguishable; and keep the closure scope to the selected request, leaving
replacements and shared channels outside it. Remove the autolab-only Plane
setup, tools and documentation this makes obsolete, keeping shared Plane
support where forge and cagent still use it.

## What changed

### The room reads the record where it now is

`agentroom/src/agentroom/autolab.py` (new) is the reader. It reproduces the
format `agautolab.anchor` writes — exactly as `closing.parse_work_note`
reproduces the `[work]` note and `routines` reproduces the run-topic names —
because this room reads other agents' realms and holds no dependency on any
of their packages. A `[selfnote][mission]` note makes a `workplan-` topic a
mission; a `[task]` note makes a `workrun-` topic one of its tasks;
`[state]`, `[doc]` and `[replaces]` say where the work has got to, which post
is the current document, and what it replaced.

One rule was added rather than copied: **the identity note's author is the
record's author**, and only that sender's other notes are read. The writer
asks this of its own history with `self_id`; from outside the author has to
be discovered, and without the rule a visitor's `[state]` line in a task
topic would move somebody else's work.

### A task belongs to a mission by id

This is the load-bearing consequence of step 2. A retired conversation gives
up its display name to its replacement, so a `workrun-` topic's root note may
name a topic that is now *different work*. The name says it belongs here; the
**anchor id** says it does not, and the id decides:

- a task whose mission the walk reached and owns is owned;
- a task whose mission the walk reached and excluded is excluded;
- a task whose mission was not reached, but whose named home *is* now a
  different mission, is excluded as "the name was reused and this is the
  older work" — with the note ids as evidence.

A mission whose state is `replaced` is excluded outright: its replacement is
the request to complete, and closing one must never close the other. That is
the closure scope the plan asks for, tested at the sharpest place it can be
wrong.

### Acceptance is a word written where the work happened

`close.py` judges an autolab mission by `agautolab.mission_done`'s own
counting rule (reproduced in `autolab.reason_not_finished`) and, when the
human clicks, writes:

- `[selfnote][state] accepted` in each finished task's topic,
- `[selfnote][state] done` in the mission's topic,

and only then resolves the topics and archives the channel. The three things
the plan asks to keep apart are now three different marks in the record:

| what happened | who says it | where |
|---|---|---|
| the run did the work | the run and the developer, in the task's conversation | `[state] completed` |
| a person accepted it | this button | `[state] accepted`, `[state] done` |
| the conversation is over | this button, afterwards | Zulip's `✔ ` |

`accepted` existed as a state with nothing writing it after step 1. This is
what writes it.

The notes go in under each topic's **live** name, because a task topic is
usually already resolved by the run that finished it and a post under the
bare name of a `✔ ` topic opens a twin beside it. A cancelled task is left
alone: accepting it would say a person approved work nobody did.

### `WorkTarget` carries either storage

It was "one Plane issue". It is now "one work record this request is
responsible for", with a `source` to branch on, Plane's coordinates
(`project_id`/`issue_id`) empty for a chat-kept record and the conversation's
(`channel`/`topic`/`anchor_id`) empty for a Plane-kept one. An autolab
mission's tasks are its `children`, which is what the existing UI already
renders; a task is not a target of its own, because it is a conversation and
the conversation already has its row.

### The Plane path, narrowed to forge

`_plane_and_channels` became `_works`, which asks two sources separately.
What went with autolab: the `external_id` lookup, the
`pj-<slug>`-to-`[AUTO]`-project rule, `_project_slug`, and the bare
`<issue id>` `[work]` note — the last of which is now *reported* as "a record
in a format nothing writes any more" rather than guessed at. What stays is
what forge actually writes: a note naming a project and an issue outright.

### One honesty fix found on the way

`partial` was judged from the post-write plan's blocked count. Archiving a
channel takes its topics out of that plan, so a completion that kept the
Front conversation open because a mission was blocked could answer
`partial: false`. It is now also true when the request itself was kept open.

### The UI, reused

No new panel. Three changes, all of them removing a lie:

- the "No Plane credential" warning fires only for rows that are actually
  Plane-backed — a warning about a credential autolab never uses would read
  as a gap where there is none;
- an autolab mission's tasks are listed under it with their own state words
  and their own topics, so a reader sees `completed` before they accept;
- the copy no longer promises to "mark its Plane Work Done".

## Evidence

```
$ cd pj-agdev/agdevworld/agentroom && uv run pytest -q
235 passed in 8.52s

$ cd pj-agdev/agdevworld && npx tsc --noEmit && npm run build
✓ built in 296ms
```

226 → 235 tests. The fixtures were rewritten to the realm as autolab now
writes it — mission `m10`, channel `work-m10`, topic `workrun-task1-m10`,
with the notes rather than a Plane issue — which is itself most of the proof
that the Plane path is gone from this side.

The step's own verifications:

- **the request and its replacement, in the existing UI**
  (`test_a_replacement_request_says_what_it_replaced_and_owns_only_its_own`):
  the scope names both missions and says the retired one is not touched.
- **the closure scope**
  (`test_the_retired_missions_task_is_not_the_replacements_to_close`,
  `test_a_replaced_mission_is_another_request_and_is_not_touched`): the older
  work is excluded with its evidence, and nothing of it is written or
  resolved.
- **accepting the finished request**
  (`test_acceptance_is_written_into_the_conversations_it_is_about`): the two
  state notes, in order, under the live topic names, before the resolves.
- **new autolab work is not broken for want of a Plane issue**
  (`test_an_autolab_request_reads_whole_with_no_plane_at_all`,
  `test_without_a_plane_credential_autolab_work_is_still_whole`): with no
  Plane credential at all the mission reads whole and its action is `ready`;
  only forge's row is missing, and the gap says why.
- **the Plane half still works** (`test_a_forge_mission_is_completed_in_plane`).

## Limitations carried into step 4

- **Nothing has been deployed or run live.** No relay has been restarted on
  this code and no real request has been accepted through it. The frontend
  is built but not published.
- **Old autolab work is unreadable, deliberately.** A `work-<plane label>`
  channel and its topics carry no `[mission]`/`[task]` notes, so the room
  reads them as conversations with no work record. The plan permits this;
  nothing was migrated.
- **A related topic is still resolved when its mission is blocked.** The
  conversation closes and the work is not accepted — which is the
  distinction working — but a reader may find that surprising, and archiving
  the channel afterwards takes the unfinished task's topic with it. Left as
  it is; it is pre-existing behaviour and not this phase's proof.
- **`planeState.ts` and the `tasks` view are untouched.** That is the older
  Plane-issue dispatch path through the autolab *gateway*, not the Zulip
  listener, and it is one of the remaining Plane consumers this phase
  deliberately leaves alone.
- **The Plane boundary, restated.** `agag.plane` stays for forge's plan and
  toolset storage and cagent's change registration;
  `nctl/src/nctl_core/agent_registration.py` and Nautobot's Plane identities
  are a later phase. This phase is not Plane removal.
