# p9 step 3 — agautolab: adopt

AI-generated (Omni Agent, 2026-08-21). Commits `4f30af4` and `2ee57c3` in
`agautolab`, pushed; listener kickstarted. 164 tests green (was 159).

autolab was the last agent on the participation ledger and the last pinned to
the pre-p8 pyagag (`065de12`). Three things moved together, because none of
them works alone.

## 1. The topic carries its binding

`src/agautolab/anchor.py` is new, and is agforge's `anchor.py` on autolab's
vocabulary. Planning opens a `workrun-` topic with two selfnotes **before**
the visible task description, so the description stays the topic's last real
post and opening a topic fires nothing:

```
[selfnote][rootchat] pj-<slug>/workplan-<stem>
[selfnote][work] <plane issue id>
```

`anchor_run_topic` writes them, and is idempotent by the work note — a
re-plan that re-creates a serial finds its topic already anchored.

A serving reads both off its own history (`run_binding`): the project from
the root note's channel, because `pj-<slug>` is what says which project the
work is for, and the task from the work note. What that replaces:

| was read from | now |
|---|---|
| `workrun-task<N>-<label>` — the serial | the `[work]` note |
| the `work-<label>` channel description — `project: …; mission: …` | the `[rootchat]` note |
| the channel prefix — "is this a work channel" | whether the notes are there at all |

`mission.run_target` takes `(project, issue_id)` instead of
`(project, channel, topic, serial)`: it finds the task by id, its mission by
`parent`, and the gate among its siblings. The serial is still what the gate
and the devlog path are keyed on — it is read off the task's external id
rather than off a topic name. A cancelled task is refused outright, which the
old lookup got for free from `sub_works` dropping cancelled children.

The channel description is **still written**, unchanged, because somebody
opening `work-pd-4` should be able to see what it is for. It is no longer
parsed — `test_the_channel_description_is_no_longer_read` pins that a channel
whose description says "made by hand" serves its anchored topics all the
same.

## 2. A callback answers at home

`handle_mention` reads `rootchat_home` — the root note `agentchat send` left
in the other agent's topic — instead of `home_for(AGENTCHAT_LEDGER, …)`. It
serves the `workrun-` topic with the remote in `threads/`, and the reply goes
**home**, into the task's own topic. `handle_workrun` lost its `reply_to`
parameter; there is no caller for it any more.

That is the p8 change arriving at autolab, and it is the one with teeth here:
until now every progress report of a delegating task was a post in forge's
conversation, and a post in somebody's topic serves them. Speaking to another
agent is now only ever a deliberate `agentchat send` inside the run.

`serve_run`'s gates, `RunProgress` and the agreement close-out are unchanged,
exactly as the plan expected — they just happen across more, shorter runs.

## 3. A served callback is marked

`note_served` (step 1) after the serving, into the `workrun-` topic. Because
the reply went home, autolab is never the last poster where it was named;
without the mark, criterion 3's restart would run the supercoder again
against a live repository.

## The removals

`agag.participation`, `AGENTCHAT_LEDGER`, the ledger-based `handle_mention`,
`work_channel_binding`, `parse_run_topic`, `WORK_CHANNEL_BINDING` and
`WORKRUN_TOPIC_NAME`. pyagag bumped `065de12` → `0cfe8a8`. Grepped across all
four repositories: the only surviving mentions of the ledger are the two
sentences in `role_run.py` saying it is gone.

## The guides: nothing changed

Both were read. The supercoder guide already said:

> Post the request or reply and finish. You will be called again when they
> answer, and the result goes into this task's own topic.

Until today the second half was aspiration — the answer went into forge's
topic. It is now description. The superdirector guide says nothing about
where a reply lands. The plan predicted "expect nothing to change"; nothing
did.

One guide commit is in this step's range and is **not p9's**: `4f30af4` is
the developer's own uncommitted edit, found in the working tree at 13:29 and
committed separately so the two are not mixed.

The introduction is unchanged too — it describes the entrance, the surfaces
and the agreement contract, none of which moved. Not re-posted.

## Tests

| test | what it pins |
|---|---|
| `tests/test_anchor.py` (4) | the work note's format, and that the earliest one wins |
| `test_a_run_topic_that_is_not_bound_to_a_task_is_explained` | four unanchored shapes, including another bot's notes |
| `test_a_root_note_naming_a_conversation_of_no_project_is_not_a_binding` | a root note outside `pj-` is dropped, not guessed at |
| `test_the_channel_description_is_no_longer_read` | replaces `test_a_work_channel_without_a_binding_is_reported` |
| `test_an_already_anchored_topic_is_not_anchored_twice` | re-plan idempotency |
| `test_a_plan_reconciles_…` / `test_a_mission_serving_opens_one_run_topic_per_task` | notes first, task description last |
| `test_a_mention_serves_the_task_the_request_was_made_for` | reads home off the chat; **replies at home** |
| `test_a_served_callback_is_marked_in_the_task_topic` | the mark, in the task topic, at the answering post's id |
| `test_a_mention_no_task_delegated_to_costs_no_run` / `test_a_root_note_that_is_not_a_task_is_not_guessed_at` | the two drop paths |
| `test_run_target_reads_the_task_by_its_issue_id_and_its_mission` and four siblings | the Plane lookup by id, with its gate |

## Kickstart

```
07:01:42Z agautolab zulip listener starting (pull sweep: … plus mentions)
07:01:43Z full sweep: 0 awaiting, 1 mentioning, 29 calls spent, 976 left in the window
07:01:43Z serving mention in 'pj-assetpipe1'/'create-asset_…'
07:01:43Z mention in … carries no root note of ours; ignoring
```

Zero runs. The one mention recovered is a p1-era `create-` topic in an
archived project; under the ledger it read "matches no participation", under
the chat it reads "carries no root note of ours" — same outcome, no file
involved.

## Criterion 4, checked now

- `agag.participation` imported nowhere.
- agautolab on pyagag `0cfe8a8`, the same commit as agforge and agfront.
- `AGENTCHAT_LEDGER` gone from the code and `.local/agentchat/` absent on
  disk in every agent.

Criteria 1–3 are step 4's to prove.
