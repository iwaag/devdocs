# run rework — Step 5 report: tests

Done. 171 tests pass (146 at the start of the episode). Most were written
beside the code in Steps 1–3; this step closed the remaining gaps on the
plan's checklist and added the `run_target` unit tests it implied.

## Against the plan's checklist

| Asked for | Test |
|---|---|
| `upsert_work` returning the issue | `test_upsert_work_updates_an_existing_work_in_place` / `…creates_the_first_work` — it returns the **label**, which is what the caller names channels and topics with |
| reconcile: update keeps state | `test_reconcile_updates_a_serial_in_place_and_leaves_its_state` (Done stays Done), `…_without_touching_its_state` (In Progress stays In Progress) |
| reconcile: disappeared serial cancels | `test_reconcile_cancels_a_serial_that_disappeared` |
| reconcile: `@rev`-keyed legacy children still match | `test_a_legacy_generation_keyed_child_still_matches_by_serial` |
| mission serving creates the channel + one topic per task | `test_a_mission_serving_opens_one_run_topic_per_task` (end to end through the skeleton), `test_a_plan_reconciles_the_split_and_builds_the_run_surfaces`, `test_the_work_channel_follows_the_project_channels_folder` |
| topic-name parsing, wrong-channel reply | `test_a_run_topic_that_is_not_bound_to_a_task_is_explained`, parametrised over all three ways to miss: old any-channel button, right name/wrong channel, right channel/no serial |
| gate: N-1 not done → no agent run | `test_the_previous_task_gate_answers_before_any_cost`, plus the Plane-side `test_run_target_blocks_on_an_unfinished_previous_task` |
| report.md → comment + completed; none → task stays | `test_a_report_completes_the_task_records_it_and_resolves_the_topic`, `test_a_serving_runs_the_supercoder_…` (no report ⇒ no `report_work`, no push, topic stays open) |

## Added beyond the checklist

- `run_target` against the fake Plane: mission title/label travel for the
  devlog, the asset label is read off the task, a missing serial and a missing
  mission Work are both named errors, task 1 has no gate, and a **cancelled**
  task neither gates its successor nor answers as a target.
- `test_reconcile_says_nothing_changed_and_writes_nothing` — an unchanged task
  costs no PATCH and no Zulip post, which is what keeps a re-plan of task 3
  from disturbing tasks 1 and 2.
- `test_reconcile_leaves_a_child_nobody_serialised` — a hand-made sub-issue is
  not a planner's child and is not cancelled.
- `test_replanning_reuses_the_work_channel` — `create_channel` being
  subscribe-based is what makes a second planning round safe.
- `test_cancel_flag_cancels_everything_and_archives_the_work_channel` and
  `test_cancelling_a_mission_that_never_got_a_channel_is_quiet`.
- `test_title_slug_stays_one_readable_path_component`, including a title with
  no ASCII at all (falls back to `mission`).
- `test_the_mission_devlog_directory_is_minted_once_and_then_found`.
- `test_a_work_channel_without_a_binding_is_reported`.

## Test-side notes

- The fake Zulip client grew the channel API — `channels`, `channel_subscribers`,
  `create_channel`, `send_to_channel`, `archive_channel`, `stream_id`,
  `channel_topics` — and a `RunClient` subclass whose realm already holds the
  mission's `work-` channel with its binding description.
- `last_write`/`last_reply` were duplicates; one helper (`last_reply`) survives.
- No test reaches the network: Plane goes through the existing `FakePlane`
  request stub, Zulip through the fake client.

## Verification

`uv run pytest -q` — 171 passed in 0.7 s.
