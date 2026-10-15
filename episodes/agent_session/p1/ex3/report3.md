# Agent session p1 ex3 — step 3: three pairs executed

Date: 2026-10-15, 18:23–20:43 JST. The raw evidence for each run is in
the ignored notes `pj-agdev/.local/agent-session-p1-ex3/step3/<set>-<arm>/`:

- the driver log;
- the request;
- the evaluator's decisions;
- the exec store copy;
- every attempt's stream log;
- the session transcripts;
- `measure.json`;
- the run record;
- a bundle of the run branch.

`aggregate.py` and `aggregate.json` beside them hold the per-attempt
figures. Exactly six runs were made: no repeats, retries, order reversal,
substitutions or paid probes.

## Order and outcome

| # | Run | Window (JST) | Attempts | Outcome | Reported cost |
| --- | --- | --- | --- | --- | --- |
| 1 | A fresh | 18:23–18:57 | 5 (plan, implement, follow-up, repair, close-out) | completed, accepted | $6.57 |
| 2 | A run | 18:57–19:19 | 5 (same phases, one session) | completed, accepted | $6.00 |
| 3 | B run | 19:20–19:35 | 4 (no repair) | completed, accepted | $3.02 |
| 4 | B fresh | 19:35–19:53 | 4 (no repair) | completed, accepted | $3.71 |
| 5 | C fresh | 19:53–20:20 | 5 | completed, accepted | $5.57 |
| 6 | C run | 20:21–20:43 | 5 (one session) | completed, accepted | $6.08 |
| | **Total** | | 28 | 6/6 accepted | **$30.95** of $45 |

- No run reached a ceiling. The largest run was A fresh: $6.57 against $9,
  and 1,890 s of attempt wall time against 2,700 s.
- Every attempt exited 0.
- There was no compaction, interruption, stale-state refusal or
  infrastructure incident. Production services were not touched.

## Boundaries as delivered

- Every answer was posted only after the executor had recorded the
  previous attempt's end and killed its process group. The driver logged
  each answer as "1 s after the attempt ended". No attempt took up an
  answer inside the process that asked for it; that was ex2's asymmetry.
- The plan agreement and the staged follow-up were automatic. In all six
  runs:
  - the first question recorded a plan document and got the plan reply;
  - the second presented `p1` and got the set's follow-up.
- The later result checks waited for evaluation (step 4). These
  evaluation pauses were natural waits:

| Run | Pause before repair answer | Pause before acceptance |
| --- | --- | --- |
| A fresh | 102 s | 26 s |
| A run | 33 s | 21 s |
| B run | — | 78 s |
| B fresh | — | 33 s |
| C fresh | 115 s | 25 s |
| C run | 84 s | 34 s |

- Every gap between attempts, pauses included, was under two minutes. The
  retained sessions' resumes all read their prefix from cache. Each wrote
  only 0.8–1.3 k tokens at its first request, so no cache expired.
- No unexpected question arrived, so the frozen "other" reply was never
  used.

## Shared-cache conditions

- Every run's first attempt was a new session. It read the same 20.4 k
  shared harness prefix from cache and wrote ≈23.5 k.
- In the fresh arm, every later new session read 26.0 k from cache and
  wrote 18.2–18.4 k. Its contract part was warm from the run's own earlier
  sessions.
- Order was fresh-first in A and C and run-first in B. Each run had its own
  area, so no workspace, record or code was shared between arms.

## The staged follow-up and repairs

| Run | Follow-up attempt | Repair answer (one allowed per arm) |
| --- | --- | --- |
| A fresh | Z key and a HUD button clear the rally | The help line explaining how to set and clear the rally was cut off at the canvas edge. Fixed by wrapping the help. |
| A run | Z key and a HUD button clear the rally | The same defect, with the same wording. Fixed by wrapping the help in a smaller font. |
| B run | Numbered waypoints 1…n | none |
| B fresh | Numbered waypoints plus a "残り N" counter over the unit | none |
| C fresh | Saved match time shown, Load disabled without a valid slot | A save taken while a worker helped an unfinished building could not be loaded. That build order has no x/y, and the strict validator rejected it. The slot then showed "保存データなし" and the earlier save was lost. Fixed. |
| C run | The same follow-up, with buttons moved to the top right | Partly corrupted saves loaded: a queue item of an unknown kind crashed the loop after load, and a missing ore value gave NaN. Fixed by per-field validation. |

Both arms already knew their follow-up was new work. Neither had
implemented it before it was asked for, so there was no asymmetry of that
kind.

## Trial branches

The six run branches are stored as
`trial/agent-session-p1-ex3/{a,b,c}-{fresh,run}`. Each holds its run
records and its game changes on top of the trial commit `7669fdb`
(`979d6a7` + two workflow files). They are in the ignored bare repository
`pj-agdev/.local/agent-session-p1-ex3/trial-branches.git` and in each run's
bundle.

The first push to the RTS project's Gitea repository was refused by the
session's permission classifier as a change to a shared resource. After the
person approved it manually, the branches were pushed there (see
[report5](report5.md)). Nothing was integrated into `main`.
