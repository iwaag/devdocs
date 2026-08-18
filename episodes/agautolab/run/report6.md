# run rework — Step 6 report: rollout and live smoke

Done and **live**. Both placements run the rework, and one whole mission was
planned, gated, worked and closed end to end on the real Zulip + Plane.

## Rollout

| Where | What |
|---|---|
| `iwaag/agautolab` `main` | `92cc45d` (Steps 1–5) pushed to GitHub |
| `pj-agdev` superproject | submodule bumped to `92cc45d`, pushed |
| agstudio listener | `launchctl kickstart -k gui/$(id -u)/com.agdev.agautolab-zulip` — restarted, sweeping again with prefixes `('mission-', 'run-', 'create-', 'bmining-')` |
| agautolab1 | `setup_autolab_node.yml --limit agautolab1` — `ok=20 changed=3 failed=0`; node checkout verified at `92cc45d` |

`autolab_node_repo_url` in the rendered inventory is
`https://github.com/iwaag/agautolab.git` for both hosts — checked, not quoted
from docs. agautolab1's Zulip-listener tasks are `skipping:` by design
(`autolab_node_zulip_listener: false`), so that node serves the gateway only
and the smoke test ran against the agstudio listener.

## Smoke test — a whole mission, live

Fresh project `runsmoke1` (`init_project.py`), channel `pj-runsmoke1` created
with Developer, the Autolab bot and Omni Agent. Then, in order:

1. **Mission posted** into `pj-runsmoke1` / `mission-hello-file` asking for two
   trivial tasks. The superdirector planned it and the handler answered:

   ```
   created R-1 "Add hello files to the project"
   created sub-work R-2 "Create README.md in main/"
   created sub-work R-3 "Create HELLO.md in main/"
   work channel work-r-1 is ready
   opened work-r-1/run-task1-r-1
   opened work-r-1/run-task2-r-1
   mission R-1 is now In Progress
   ```

   The channel came out as expected: `work-r-1`, description
   `[AUTO] project: runsmoke1; mission: pj-runsmoke1/mission-hello-file`,
   **3 subscribers inherited** from `pj-runsmoke1`, folder `None` — the realm
   still has zero channel folders, and the code correctly created no folder of
   its own. Both topics carry their task content, posted by the bot.

2. **Gate proven closed.** A post into `run-task2-r-1` while R-2 was unstarted
   got exactly `Please complete previous work (R-2)` — no agent run, no cost.

3. **Task 1 served.** One `supercoder` run (`claude_code` / `sonnet`, record
   `.local/agent/supercoder/run-0001.json`, `outcome: done`) in the project
   folder, `report.md` written into
   `.local/topics/work-r-1/run-task1-r-1/1/supercoder/`. The handler then:
   commented and completed **R-2**, resolved the topic (`✔ run-task1-r-1`),
   wrote `devlog/r-1-add-hello-files-to-the-project/task-1/{work.md,report.md}`
   and pushed it (`[AUTO] task 1 report for R-1`).

4. **Gate proven open.** The same post into `run-task2-r-1` now ran task 2
   (`run-0002.json`), which closed the same way: R-3 completed,
   `✔ run-task2-r-1`, `task-2/` recorded and pushed
   (`[AUTO] task 2 report for R-1`).

Final state: `main/` holds `README.md` and `HELLO.md`; Plane shows R-2 and R-3
`completed`, R-1 still `started`; the devlog clone is two commits ahead-and-pushed;
both run topics resolved, the `work-r-1` channel open. That last part is
expected — how a finished `work-` channel gets closed is next phase's decision,
as the plan says.

No handler failure was logged at any point.

## One gap found, deliberately not closed

**Nothing commits `main/`.** After both tasks, `main/` was
`?? README.md  ?? HELLO.md` — the work product sits uncommitted in the clone.
This is **not new**: `commit_all_and_push` has only ever been called on the
`direction` clone (bmining) and now the `devlog` clone; the old coding flow
never committed `main` either, and neither the old `run_coding` guide nor the
new `run_supercoder` guide asks the agent to. The supercoder does carry
`Bash(git:*)`, so it *could* be asked to — but whether committing the source is
the agent's job or the handler's is a design decision outside this plan, so it
is reported rather than decided. Worth putting in the next braindump.

## Deus Ex Machina note

The Omni Agent posted the smoke mission and both task triggers, and created
the `pj-runsmoke1` channel by hand (`init_project` does not create Zulip
channels) — did those for the Developer and for agautolab; handoff candidate.

## Verification summary

- `uv run pytest -q` — 171 passed.
- agstudio: listener restarted, log clean.
- agautolab1: `failed=0`, HEAD `92cc45d`.
- Live: mission → 2 topics → gate blocks → task 1 done → gate opens → task 2
  done, with Plane, Zulip and the devlog all agreeing.
