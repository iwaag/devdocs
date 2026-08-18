# run rework — Step 2 report: mission serving prepares the run surfaces

Done. Planning a mission now also builds the place the work is done: one
`work-<label>` Zulip channel per mission Work, holding one
`run-task<N>-<label>` topic per Sub-Work, and re-planning **updates** those
surfaces in place on both sides instead of cancelling and re-creating them.

## Plane side — the serial reconcile

`register_task_files` (cancel-everything, then re-register under a fresh
`@<rev>` generation key) is replaced by `reconcile_task_files`, which matches
live Sub-Works to the new `task[N].md` set by **serial** — the `#<N>` tail of
the external id:

| Case | What happens |
|---|---|
| serial exists, content identical | nothing is written; reported `unchanged` |
| serial exists, content changed | title/description updated, **state untouched** |
| serial exists, changed, already Done | updated, reported `changed-after-done` |
| serial is new | Sub-Work created, key `<channel>/<topic>#<N>` |
| serial disappeared | that Sub-Work is cancelled |
| child carries no serial | left alone — a hand-made sub-issue is not a planner's |

Leaving the state alone is the load-bearing part: it is what lets a completed
task stay completed, which is what makes the Step-3 previous-work gate mean
anything across a re-plan.

Old keys carry `@<rev>` between the topic and the `#<N>` tail. `sub_work_serial`
reads the tail, so legacy children match their serial with **no migration**
(covered by a test). The generation directory `<G>/` stays as the workspace
double-act guard; it no longer appears in any Plane key.

`upsert_work` now returns `(report line, work label)`. It returns the *label*
rather than the issue the plan suggested, because composing a Plane label
(`PD-4`) needs the project row too, and that row only exists inside
`mission.py` — the caller wants the label, and only the label.

## Zulip side — one to one with the reconcile

`handle_superdirector_response` takes the client now and, on the `plan.md`
branch, calls `prepare_run_surfaces`:

- `ensure_work_channel` looks up the parent `pj-<slug>` channel, takes **its**
  subscribers as principals and **its** `folder_id` as the folder (including
  none — inventing a folder structure here was explicitly out of scope), and
  calls `create_channel`, which is subscribe-based and therefore idempotent
  across re-plans.
- The channel description carries the binding Step 3 reads back:
  `[AUTO] project: <slug>; mission: <channel>/<topic>`. The channel name gives
  back the Work label and nothing else, so the slug and mission key must
  travel here.
- `mirror_task_changes` mirrors each `TaskChange`: `created` → post the task
  content; `updated` → "Updated by planner." + the content; `cancelled` →
  "Cancelled by planner." then resolve the topic (the post's message id is
  what `resolve_topic` needs, so it goes through `send_to_channel`, which
  returns one, rather than `topic_write`, which returns `"success"`);
  `changed-after-done` → a note and nothing else; `unchanged` → silence, so a
  re-plan of task 3 does not disturb tasks 1 and 2.

The autolab bot is the one posting, so it is each new topic's last poster and
the sweep stays quiet. **The topic waits, by design, until a human posts.**

The `changed-after-done` note is posted under the topic's *resolved* name
(`✔ run-task1-pd-4`) when that is what exists — a resolved topic is a renamed
topic, so the bare name would open a second one beside it. Per the plan this
branch is minimal and deliberately not exercised live this episode.

## Mission cancel is the only cancel-everything path left

`cancel.flag` still cancels every live Sub-Work (`cancel_sub_works` is kept
for exactly this) and now also archives the whole `work-` channel. Nothing is
ever re-created after a mission cancel, so the archived channel's retained
name cannot collide with a later one.

## Changes

| File | Change |
|---|---|
| `src/agautolab/mission.py` | `reconcile_task_files` + `TaskChange` replace `register_task_files`; `sub_work_key` drops `@<rev>`; `NO_SERIAL`; `upsert_work` returns the label |
| `src/agautolab/zulip_listener.py` | `work_channel`, `run_topic`, `work_channel_description`, `find_channel`, `ensure_work_channel`, `live_topic_name`, `mirror_task_changes`, `prepare_run_surfaces`, `archive_work_channel`; `handle_superdirector_response(client, …)` without `number` |
| `tests/test_mission.py` | eight reconcile tests: update-keeps-state, cancel-on-disappear, legacy `@rev` match, unserialed child left alone, unchanged writes nothing |
| `tests/test_zulip_listener.py` | fake client grew the channel API; work-channel creation, folder inheritance, per-change mirroring, replan idempotence, cancel archives |

## Verification

`uv run pytest -q` — 157 passed (147 before, +10). No live Zulip or Plane call
was made; Step 6 is where this meets the real realm.
