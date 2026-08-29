# scheduled_routine p7 — Step 3 report

## Did parallel Task launches actually happen?

Yes. From task2's own transcript (quoted in `report2.md`'s timeline) and its
`report.md`: all 5 `Agent` (Task) tool calls were issued in a single
assistant message with `run_in_background: true`, one per paper, at
~23:09:39 local. All 5 were confirmed still in flight together during the
~23:10:20–23:11:29 window (repeated directory polling showed no
`summary.md` yet for any of the 5), and their five completion timestamps
land close together (23:11:29, 23:11:57, 23:12:21, 23:12:48, 23:13:07) —
far tighter than the 82–122s individual durations, which is only possible
if the runs overlapped. **5 concurrent subagents**, the maximum possible
for this mission.

## Wall clock vs. serial estimate

- Individual subagent durations: 82s, 108s, 102s, 120s, 122s.
- Sum if strictly sequential: ≈ 8m52s (532s).
- Actual wall clock, launch to last completion: ≈ 3m30s (210s).
- Speedup: **≈2.5×** on 5-way work — well under the naive 5× ceiling
  (expected, since duration_ms scales with each subagent's own web-fetch
  latency and the parent still serializes its own file-write/verification
  turns after each finishes), but decisively faster than one-after-another,
  consistent with true concurrency rather than an illusion of it.
- Task1 (selection, no subagents): ~5 minutes end to end (13:53:54–13:58:50)
  for research + write across 5 papers in one sequential session — a rough
  same-agent baseline showing task2's subagent fan-out did not simply move
  the same sequential cost into background threads that still serialize on
  a shared bottleneck.
- Cost/turns were not captured per-run by the listener's own record for
  this session (the harness result JSON's `cost_usd`/`num_turns` were not
  separately surfaced into the Zulip report by either task); the probe in
  Step 1 (2 subagents, $0.27, 3 turns, 10.9s) is the only harness-level cost
  figure directly observed in this episode. This is a gap — a future
  episode should have the mission explicitly quote its own result meta.

## Output integrity

- 5 new `main/papers/<id>/summary.md`: `2608.15089`, `2608.23041`,
  `2608.25593`, `2608.15763`, `2608.26530` — all present, 40–56 lines each,
  each ending in an explicit `runnable:` line.
- 5 new `INDEX.md` rows, appended by the **parent** session only (verified:
  no subagent touched `INDEX.md` per both tasks' reports and the file's own
  git blame — the single `Edit` in task2's transcript is the parent's).
- No duplicate of the 4 pre-existing papers (2608.20430, 2608.21156,
  2608.23283, 2608.23552) — all 5 new ids are distinct from those and from
  each other.
- No `INDEX.md` corruption: the file is a clean 9-row table (4 old + 5 new),
  read back successfully after commit.
- `main` committed (`aff781e` selection, `0c6fdac` summaries+index) **and**
  pushed — confirmed via `git fetch`/`git log origin/main` in both tasks'
  transcripts, and independently re-verified here:

  ```
  0c6fdac Add summaries for 5 papers investigated concurrently (2026-08-28)
  aff781e Add trend-signal selection of 5 new arXiv candidates (2026-08-28)
  ```

## Quality skim (Developer/Omni read, 2 of 5)

Read `2608.15089` (StateM) and `2608.15763` (TaoLive) in full.

- Both follow the established shape (problem, method, results, why it
  trends, local run, `runnable:`) at the same depth as the pre-existing
  singly-produced summaries (2608.23552, 2608.23283).
- Both cite concrete, checkable numbers (StateM: 95.3% raw accuracy/445
  trials, $15 vs $574.68, 445 HF upvotes, 628 GitHub stars re-verified live
  at read time; TaoLive: per-metric benchmark table, GMV uplift figures,
  explicit statement that the live current HF upvote count could not be
  re-verified this session and should be read as a snapshot, not fresh).
- Both verdicts are justified with specific evidence (StateM `yes`: open
  Apache-2.0 CLI, real README, zero non-stdlib deps; TaoLive `no`: no
  repository, no weights, explicitly proprietary/internal).
- **No quality degradation observed from parallel production.** If
  anything, StateM's summary is unusually careful about distinguishing a
  live-reverified GitHub star count from the selection doc's stale figure —
  a level of self-checking not asked for explicitly.

One friction surfaced only in the transcript, not in the final files: two of
the five subagents (TaoLive, PILOT) hit a tool-level guard rejecting `Write`
calls to paths matching `summary`/`report` in the filename, and had to
return their content as text for the parent to write instead. This did not
affect the delivered files' quality or existence, but it is a second,
distinct friction from the git-push one — noted for the phase report.
