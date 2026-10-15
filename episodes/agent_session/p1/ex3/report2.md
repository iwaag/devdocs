# Agent session p1 ex3 — step 2: one consistent trial protocol

Date: 2026-10-15. The driver, the setup scripts and the fixture checks are
in the ignored notes `pj-agdev/.local/agent-session-p1-ex3/step2/`. The
live cost estimator is in `tools/livecost.py` next to it. No model was
called in this step.

## Areas

- `setup-base.sh` makes the one trial commit, `7669fdb`: pj-rts-vs-bot
  `979d6a7` plus `ex3-fresh.yaml` and `ex3-run.yaml`. The two files are
  copies of `improve-game` and differ only in id, name and
  `execution.session`.
- `setup-area.sh` gives each of the six runs its own area, cloned from that
  commit. Each area has its own:
  - bare remotes for the project and study-rts;
  - workspace;
  - registry naming only that workspace;
  - exec store;
  - presenter port range: A-fresh 18800–18899, then +100 per area in run
    order;
  - a `bin/wfe` running the deployed release (`service/current`, `a642e27`).
- All six workspaces were checked at `7669fdb` / study-rts `d2734db`.
- Nothing in production is used. The production executor, its store and
  the held jevlab run are untouched.

## Workflow shape (both arms)

The workflow is `improve-game` unchanged:

> `_plan` → implement → verify → present → human-check (confirms result) → cleanup

The staged follow-up is the first result check's answer. The agent grows
the run with its own adjustment, as `agent/review.md` describes:

> repair → verify → present → check

The second check is the acceptance point. Only `execution.session`
differs between the arms. Model, effort, guides, contract, catalog, tools,
budgets, output requirements and evaluation criteria are identical.

## Driver (`drive.py`, reusing ex2's `step4.py`)

One run per invocation: `run.sh <area> <set> <arm> [--spent <usd>]`.
`run.sh` removes the Omni session's `CLAUDE*`/`ANTHROPIC*` variables and
starts the deployed executor code in-process against the area's `wfe`, as
in ex2. Changes from ex2:

- **Texts come from the frozen `contract.json`**: request, follow-up,
  plan, acceptance and other replies.
- **Boundary answers wait for the attempt's end.** The driver answers only
  when the executor holds no attempt. That means the previous attempt's
  end was recorded and its process group was killed (`finish`). The wait
  from that end to the answer is logged. In ex2 an answer arrived while an
  attempt was still running, and that arm continued in its own process.
  That cannot happen here.
- **Evaluation pause before acceptance.** A result check after the
  follow-up gets no automatic answer. The driver writes
  `pending-eval.json` and exits 5. The evaluator checks the presented
  branch against the frozen criteria and writes the reply to
  `decisions.json`:
  - acceptance;
  - or the single defect-repair answer.

  Restarting the driver posts it. A restarted driver does not answer a
  question twice.
- **Ceilings.** Every 20 s the driver adds up:
  - **Run cost**: completed attempts at their reported deltas (`measure.py`
    already turns a retained session's cumulative totals into
    per-attempt deltas), plus a killed attempt at its log estimate, plus
    the running attempt at a live estimate.
  - **Active time**: the attempts' wall time, plus the running attempt's
    elapsed time.
  - **The overall total**: `--spent` from earlier runs plus this run's cost.

  If any ceiling is crossed ($9 per run, 45 minutes per run, $45 in total),
  the driver stops the run (`wfe run stop`, the executor stops the attempt
  and confirms it is gone) and exits 4. There is no retry.
- **Exit codes**:
  - 0: completed with the last decision `accepted`;
  - 2: cancelled;
  - 3: held;
  - 4: a ceiling;
  - 5: an evaluation pause;
  - 6: completed without acceptance.

The ex1 launch gate and single-writer behaviour are the executor's. The
driver calls `Executor.tick()` as the production loop does, against its own
area's store, and changes nothing in the executor.

## Fixture checks (`check_driver.py`): 35 passed

| Area | Cases |
| --- | --- |
| Question matching | Each kind is recognised by recorded meaning: a plan document → plan; a presentation → result; a node that confirms the result, including one added by a revision → result; anything else → other. |
| Replies, per set | The first result check gets that set's follow-up, which is absent from its request. A later check pauses for evaluation. An evaluated acceptance is posted. A plan gets the plan reply. Other questions get the frozen other reply. No non-acceptance text contains the acceptance text. |
| Lifecycle gating | No answer while the executor holds an attempt. |
| Acceptance | Counted only when the run is completed and its last decision is `accepted`. An accepted-then-rejected run, or an accepted but unfinished one, does not count. |
| Budget | Synthetic exec store: one retained session with cumulative totals ($1.00 → $2.50), a killed attempt without a result event, a running attempt, and another run. The result is $1.00 + $1.50 + the killed attempt's estimate + the live estimate; the other run is excluded; active time is the attempts' wall time. Each ceiling is checked at both sides of its threshold. |

The estimator was also replayed on ex2's seven logs. It was 0–30 % above
the reported per-attempt costs, so it errs toward stopping early.

No paid smoke run was made. The first pair is the real-harness evidence, as
the plan says.
