# run rework — Step 3 report: `handle_run` is a task conversation

Done. A `run-` topic is no longer a channel-agnostic button that picks
whatever Work is next out of every `[AUTO]` project. It lives in one
mission's `work-` channel, it is bound to one Sub-Work by its own name, and
it is served through the same `serve_topic` skeleton as `mission-` — so every
later human post re-serves it and the task is finished by agreement between
the developer and the supercoder, not by one agent run deciding alone.

## The new contract, in order

1. **Binding.** `parse_run_topic` requires both halves: a `run-task<N>-…`
   name *and* a `work-` channel. A topic failing either gets one explanatory
   reply and no run — that reply is what replaces the old any-channel button.
   `dispatch` still routes every `run-` topic here; the handler is where the
   decision moved.
2. **Slug and mission key** come from the channel description
   (`work_channel_binding`), which Step 2 wrote. The channel name alone
   recovers only the Work label.
3. **Gate.** `run_target` reads the parent Work's live Sub-Works once and
   answers both the task at serial N and whether serial N-1 is `completed`.
   Not completed → `Please complete previous work (PD-5)` and stop,
   handler-side, before any cost. N=1 has no predecessor.
4. **Asset gate**, unchanged in shape but no longer queue-blocking: absent →
   order and stop; working → report and stop; done → re-sign immediately
   before the run and append the asset note to the prompt. Because topics are
   per-task now, a waiting asset blocks only its own task.
5. **Serving.** Generation workspace
   `.local/topics/<channel>/<topic>/<G>/supercoder/` with `chatlog.md`, then
   one `supercoder` run **in the project folder** (`.local/projects/<slug>/`,
   where `main/`, `direction/`, `devlog/` are real directories — matching the
   guide), workspace handed over by absolute path, `WORK_TIMEOUT_SECONDS`,
   record under `RECORDS_ROOT/supercoder`, `RunProgress` streaming into the
   topic.
6. **The task text travels in the prompt**, read back from Plane rather than
   from a file: Plane is the ledger from registration onwards, and the
   `task[N].md` the superdirector wrote lives in a different generation's
   directory.
7. **Closing the loop** — without it the Step-3 gate could never open. If the
   serving's workspace holds `report.md`, it is commented on the Sub-Work, the
   Sub-Work moves to `completed`, and the topic resolves. No `report.md` is
   not a failure: the conversation simply is not finished, and the topic stays
   open. `success.flag` is gone — one signal, not two.
8. **Devlog record**, on the same done branch: `work.md` (the task as handed
   to the supercoder) and a copy of `report.md` are written to
   `devlog/<mission dir>/task-<N>/` and the devlog clone is committed and
   pushed (`[AUTO] task <N> report for <label>`). Deterministic handler code,
   the `serve_bmining` pattern — the agent is never asked to run git.

`<mission dir>` is `<work label lowercased>-<slug of the Work title>`, e.g.
`pd-4-fix-title-screen/`, **frozen at the first write**: a later re-plan may
rewrite the Work title, and a record that moved would stop being a record.
The handler looks the directory up by its `<label>-` prefix and only mints a
name when no such directory exists. The current title lives inside `work.md`.

## What was removed, and why it had to be

`work_prompt`, `run_work` and `work_directory` are gone. `work_prompt` loaded
`guide("run_coding", "guide.md")` and that directory no longer exists on disk
— the old `handle_run` would have crashed at prompt build. The rework
subsumes it, as the plan said. `.local/work/` and its dirty-workspace refusal
go with it: a serving now writes into its own generation directory, which was
always the better guard.

Kept as the plan asks: `next_work`/`eligible_works` in `mission.py` (no
caller now), the `coding` and `director` roles. `director` is still live for
`bmining-`; `coding` is dormant.

## Changes

| File | Change |
|---|---|
| `src/agautolab/mission.py` | `RunTarget` + `run_target` — the task at a serial and its gate, from one issue read |
| `src/agautolab/zulip_listener.py` | `serve_run` through `serve_topic`; `parse_run_topic`, `work_channel_binding`, `supercoder_prompt`, `run_supercoder`, `devlog_directory`, `title_slug`, `mission_directory`, `record_task_in_devlog`; `work_prompt`/`run_work`/`work_directory` deleted |
| `tests/test_zulip_listener.py` | the whole run-topic block rewritten: binding rejections, the gate, workspace/prompt shape, generations, report→complete→record→resolve, devlog naming, asset route, progress tail |

## Verification

`uv run pytest -q` — 163 passed (157 before, +6 net after replacing the old
button's tests). Step 6 is where this meets the real realm.
