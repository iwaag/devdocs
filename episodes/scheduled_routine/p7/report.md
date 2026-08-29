# scheduled_routine p7 — phase report

## The braindump's question, answered

**Yes — Claude Code's harness-native parallel execution (the `Task` tool)
works inside an in-system autolab session, with explicit instruction, and
nothing had to change to allow it.**

- The exact `roles.supercoder` `allowed_tools` grant (no `"Task"` entry)
  ran a 2-subagent probe cleanly (`permission_denials: []`,
  `subagent_stats.completed: 2`) — see `report1.md`.
- The real mission's task2, run through the deployed `com.agdev.agautolab-zulip`
  listener with the same grant (and autolab's existing
  `--dangerously-skip-permissions` bypass, which predates this episode), ran
  all 5 subagents concurrently — see `report3.md` for the timing evidence
  (5-in-flight-at-once, ~2.5× real speedup over serial).
- No `agents.toml` edit, no deploy, no pin bump was needed anywhere in this
  phase.

## Cost/time: 5-in-one-session vs. five routine fires

- **This episode (parallel, 1 mission, 2 tasks)**: mission posted
  13:53:19 → mission Done 14:19:46, wall clock ≈ 26 minutes total, including
  two rounds of human-in-the-loop approval (picks, then final confirmation)
  and one push-auth friction resolved mid-run. Within that, task2's own
  investigate-5-papers work was ≈10 minutes (14:09:28 start → 14:19:46 Done)
  for research + write + merge + commit + push of all 5 papers, of which the
  concurrent subagent phase itself was ≈3m30s.
- **p4's precedent (serial, one paper per routine fire)**: one `papers`
  routine fire produces one paper's summary per `WORK_TIMEOUT_SECONDS`-bounded
  task (1200s ceiling; observed real duration well under that per p4/p5).
  Five serial fires would mean five separate Front→autolab round trips (each
  with its own Zulip overhead, plan/approve cycle if not pre-approved
  standing text, and dispatcher tick), plus the *summed* research time this
  episode measured directly: 82+108+102+120+122s ≈ 8m52s of actual model
  work, without task2's parallel win.
- Precise `cost_usd`/`num_turns` per mission-run were **not captured** in
  either task's Zulip report (a gap noted in `report3.md`); the only
  harness-level cost figure directly observed this episode is the Step 1
  probe's $0.272 for a 2-subagent, 3-turn, 10.9s run. A future test that
  wants a clean cost comparison should have the mission explicitly quote its
  own result JSON's `cost_usd`/`num_turns`/`duration_ms`.
- **Directionally**: one parallel session clearly beats five serial routine
  fires on wall clock and on Zulip/dispatcher overhead (one plan-approve
  cycle instead of five), at the cost of a harder-to-verify "did it really
  run in parallel" claim that this episode had to establish with file
  timestamps rather than a built-in metric.

## Should `routine-papers` adopt a parallel multi-paper form?

**Recommend yes, with two caveats**, based purely on this run's evidence —
not decided here, left for whoever edits the standing text:

1. The concurrency-evidence burden (reading `summary.md` mtimes and
   individual subagent durations from the transcript) is real work a
   routine mission would have to do every time to prove it wasn't secretly
   serial. If the routine is trusted going forward, this could be dropped
   from every report to just the first few fires.
2. The recurring git-push-auth friction (`agstudio` bare hostname needs
   credentials `agstudio.local` does not) hit **both** tasks in this single
   mission independently — a `papers`/`localtest`-style routine that fires
   regularly would hit it on every fire until the `main` clone's remote URL
   is changed once and left that way, or push credentials are provisioned
   for the bare hostname. This is a one-line fix but is explicitly left for
   a later episode, since p7's scope was proving parallelism, not fixing
   git config.

## Deus Ex Machina interventions

Two, both permission-classifier stops on Omni's own `agentchat` calls (not
on autolab's runs), both required by `localrule.md` ("権限分類器に作業を
止められた時は手動で許可を出すから、すぐに作業を停止して報告しろ"):

1. **Did X for agent Y — handoff candidate**: relayed the `agstudio.local`
   git-remote workaround into `work-s3-5 › workrun-task2-s3-5` by hand
   (`agentchat send`), after the classifier blocked the first attempt and
   the user granted permission. autolab's own task2 run then applied the fix
   itself once told — Omni supplied information, not the fix; the run did
   its own `git remote set-url` and `git push`.
2. **Did X for agent Y — handoff candidate**: read
   `pj-studyarxiv › workplan-parallel5`'s final state by hand
   (`agentchat read`) after a second classifier block, again with user
   permission, purely to confirm mission completion for this report — no
   effect on autolab's own state.

Neither substituted for autolab's own execution; both were narrow,
read/relay actions blocked by an overcautious classifier rather than by any
real risk, consistent with `localrule.md`'s framing.

## Numbers

- Papers selected: 5 (2608.15089, 2608.23041, 2608.25593, 2608.15763,
  2608.26530), all new, no duplicates of the 4 pre-existing rows.
- Runnable verdicts: 2 `yes` (StateM, JIT-Agent), 1 `unclear` (AutoSaddler),
  2 `no` (TaoLive, PILOT).
- Repositories: one (`autodev/papers`, the existing `studyarxiv` `main/`
  clone), 2 commits pushed (`aff781e`, `0c6fdac`).
- Front runs: 0 — this phase bypassed the schedule/Front leg entirely, by
  design (out of scope), posting the workplan by hand as the Developer.
- autolab tasks: 2 (`S3-6` selection, `S3-7` parallel investigation), both
  Done; mission `S3-5` closed with `mission_done`.
- Human/Omni interventions: 2 approval gates (picks, final confirmation) +
  1 relayed workaround, all as the Developer in Zulip; 2 permission-classifier
  stops, both resolved by explicit user grant.
- Concurrency observed: 5 of 5 subagents ran simultaneously in the
  ~23:10:20–23:11:29 window; wall clock ≈2.5× faster than the summed serial
  durations.

## Next feature, if any

Not proposed here as a commitment, only as evidence-backed candidates for a
future episode to pick up or not:

- Fix the `agstudio` vs `agstudio.local` git-push friction once, at the
  clone-creation level (`init-repo` or the `papers` clone's remote), so
  routine fires stop re-discovering it.
- Have missions that use subagents quote their own harness result JSON
  (`cost_usd`, `num_turns`, `duration_ms`) in the Zulip report, so future
  cost/concurrency comparisons don't require transcript archaeology.
- If `routine-papers` moves to a 5-parallel form, verify unprompted (not
  explicitly instructed) parallel behavior once, since this phase only
  tested the instructed case per the braindump's own scoping.
