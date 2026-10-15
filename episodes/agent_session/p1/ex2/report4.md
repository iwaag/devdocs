# Agent session p1 ex2 — step 4: one controlled RTS pair

Date: 2026-10-15, 17:14–17:39 JST. The raw evidence is in the ignored notes
`pj-agdev/.local/agent-session-p1-ex2/step4/`:

- the setup script, the driver and its log;
- `budget.md`, written before the start;
- every attempt's stream log and the four session transcripts;
- the exec store copy;
- `measure.json`, `audit.json`, `transitions.txt`, `summary.txt`.

## Setup (recorded before starting)

- **Code**: the deployed revision.
  - WFE release agdevworld `a642e27`, deployed for this step. The area's
    `wfe` runs `service/current`.
  - The executor is agautolab at the deployed `wfexec.py`, run in-process
    by the driver against the area's `wfe`.
  - Nothing in production was touched.
- **Harness**: claude_code 2.1.283 (the same as p1), `claude-sonnet-5`, the
  host's default profile, a Max subscription. Costs are client-side
  list-price estimates, not bills or quota.
- **Baseline**: a private clone of pj-rts-vs-bot at `979d6a7`. That is the
  production workspace, equal to its origin. One commit was added with
  `ex2-fresh` and `ex2-run`: copies of `improve-game` that differ only in
  id/name and `execution.session` (the Zulip intake line was dropped).
  The common contract is the wfrun guide `cf7ff7b2…`. The project guide
  `agent/review.md` and the node catalog (3 templates) are the production
  ones.
- **Task**, absent from the baseline (the result screen shows only the
  winner and the time):
  > 勝敗が決まったときの結果画面に、その試合の統計を表示してほしい。最低限、生産したユニット数・失ったユニット数・倒した敵ユニット数・採掘した鉱石の合計を出してください。細かい見た目は任せます。
- **Scripted inputs**, labelled `Trial Person (scripted trial input)`.
  They were recognised by the recorded question, not by node name:
  - a plan agreement (the question records a plan document) →
    「この計画で進めてください。」
  - the first result check (a presentation or deliverables, or a node that
    confirms the result) → correction:
    「統計は自分とボットを横に並べて、項目ごとに比べられるようにしてください。」
  - later result checks → 「確認しました。これで合格です。」
  - anything else → 「お任せします。進めてください。」, never counted as
    acceptance. It was not needed.

  All answers were immediate.
- **Cap**: $15 reported. Used: **$4.86**. One run per arm; no repeats,
  order reversal or retries. Requests went in fresh-first, as in p1's
  pair 1, so the two runs' attempts interleaved on the one slot.
- **Unavoidable difference from p1**: the driver was started with the Omni
  session's `CLAUDE*`/`ANTHROPIC*` environment removed. These were a
  messaging socket, child-session flags and an effort setting that the
  production executor under launchd does not have. Both arms got the same
  environment. A new session's first request is 42.8 k tokens here against
  53.7 k in p1, because the shared harness prefix is smaller. Absolute
  figures are therefore comparable within this pair, not with p1's.

## Results

| Run | Policy | Attempts | Steps | Cache write | Cache read | Output | Active time | Reported cost |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| ex2-fresh/run-001 | fresh | 3 | 85 | 135 k | 5.04 M | 30 k | 517 s | **$1.84** |
| ex2-run/run-001 | run (optimized) | 4 | 99 | 130 k | 10.24 M | 43 k | 939 s | **$3.02** |

Per attempt:

| Attempt | Fresh: first → last context, steps, cost | Run: first → last context, steps, cost |
| --- | --- | --- |
| 1 plan (both new) | 42.8 k → 64.2 k, 18, $0.44 | 42.7 k → 56.1 k, 11, $0.29 |
| 2 implement, verify, present, ask | new: 43.1 k → 84.7 k, 47, $1.02; **also took up the correction and re-asked (no code change)** | resumed: 57.4 k (1.3 k written) → 103.5 k, 40, $1.03 |
| 3 correction: repair, verify, present again, ask | (none: done in attempt 2) | resumed: 104.8 k (1.3 k written) → 144.3 k, 37, $1.31 |
| last: acceptance, cleanup, final report | new: 43.0 k → 57.9 k, 20, $0.39 | resumed: 145.8 k (1.5 k written) → 151.2 k, 11, $0.39 |

There was no compaction. All three resumes read their prefix from cache:
the waits were seconds to minutes, and only 1.3–1.5 k tokens were written
at each resume.

On shared-prefix warming: each arm's first new session read 20.4 k tokens
from cache and wrote 22.4 k. The fresh arm's later new sessions read 25.9 k
and wrote 17 k, because the contract part of the prefix was warm from its
own earlier sessions.

## Mechanism: what the optimization changed

| Measurement | p1 retained (pre-optimization reference, stored) | ex2 run-optimized | ex2 fresh |
| --- | --- | --- | --- |
| Continuation prompt (chars) | 3,546 / 2,938 / 3,235 | **1,682 / 1,828 / 1,884** | — (whole bootstrap 28.9–29.5 k each) |
| Tokens appended at resume (first request minus previous attempt's last) | +1,936 / +1,266 / +1,699 | +1,284 / +1,305 / +1,532 | — |
| Full-history reads in resumed attempts | 3 (1.2–3.8 k chars each) | **0** | every attempt: 2.8 k, 4.1 k, 9.6 k chars |
| Compact `wfe run state` reads in resumed attempts | — | 1 (245 chars) | 3 (1.3 k chars total) |
| Plan / request rereads in resumed attempts | 0 | 0 plan; the final report read the request once (287 chars) | plan.md once, braindump 4× |

The continuation carried the answer text, so no resumed attempt needed the
record to find it. The prompts in full are in `audit.json`. One example:

```text
Recorded since your previous attempt began (after seq 8), not by it (2):
  22 presentation.observe by presenter (presentation p1): available
  28 question.answer by Trial Person (scripted trial input) (question q2): 統計は自分とボットを横に並べて、項目ごとに比べられるようにしてください。
…
Answers recorded and not yet taken up (1), exact text; …
- q2 answer 0 from Trial Person (scripted trial input) via cli at …; your question from human-check: "…"
    統計は自分とボットを横に並べて、項目ごとに比べられるようにしてください。
```

**The token saving per resume is small, as the step 1 audit predicted.** The
prompt shrank by about half (≈1.3–1.7 k chars, a few hundred tokens). The
rest of the 1.3–1.5 k appended tokens is the identity header and what the
harness itself adds on resume. In this trial that included the skill
listing and two harness messages:

- "Your response above was cut off mid-stream. Resume directly…": the
  harness recovering an interrupted stream inside attempt 2;
- a `<task-notification>` about a background shell from attempt 3 "that
  didn't finish before the previous session ended", delivered at the next
  resume.

Neither comes from WFE. The second is retained-session specific: a stale
notice of something the previous attempt's process left behind.

**What dominates is still the carried history.** The retained session's
steps started from 57 k, 105 k and 146 k tokens. Fresh sessions started
from 43 k. That is 10.2 M against 5.0 M cached tokens read.

## Work variation

The two agents did different amounts of work, so the cost difference is
mostly not the policy:

- **Fresh** designed the side-by-side table (あなた / 相手) from the start.
  When the correction arrived, its implementation attempt was still open:
  it had checked `wfe run state` before ending and found the immediate
  answer. It took the answer up, said that p1 already shows the two sides
  side by side, and re-asked on the same presentation. It changed no code,
  and so saved an attempt.
- **Run** first showed the player's four numbers on one line. The
  correction was real work, done properly: an adjustment (revision r2)
  added repair → verify2 → present2 → human-check2. That attempt cost
  $1.31 (37 steps), and the fresh arm had no counterpart.

On the attempts both arms had:

- the implementation attempt: retained $1.03 (40 steps from 57 k) against
  fresh $1.02 (47 steps from 43 k);
- the final attempt: $0.39 each (11 steps from 146 k against 20 steps from
  43 k, the fresh one reading the 9.6 k-char history to reconstruct).

Fewer steps offset the larger context, so this is parity, not a saving.
Leaving out the run arm's extra correction attempt gives $1.71 against
$1.84. That is not a matched figure: a fresh correction attempt would also
have cost something (p1's cost $0.85–1.03).

## Quality

| Check | ex2-fresh | ex2-run |
| --- | --- | --- |
| Original requirement retained (4 stats) | yes, for both sides | yes, for both sides (kept through the correction) |
| Correction | already satisfied; said so and re-asked | implemented (table, 項目 / あなた / ボット) |
| Plan adherence | plan → implement → verify → present → check → cleanup → final report | the same, plus the recorded r2 repair loop |
| Tests | `node game/test/headless.js` 39/39 (8 added) | 34/34 (3 added) |
| Explicit acceptance | `accepted` from q3 (the scripted acceptance) | `accepted` from q3, asked from the added node `human-check2`. The driver recognised it as a result check; p1's driver defect did not recur. |
| Stale-state failures | none | none (no `attempt-stale`; the only refusals were ordinary ones the agent corrected) |
| Sessions | 3 sessions, 1 attempt each | 1 session, 4 attempts, no replacement |

Neither result was integrated into the shared project.

## What this pair can and cannot say

- **Measured**:
  - the continuation no longer appends stable material and no longer
    causes a history read;
  - resumed attempts acted on the delivered answer text at once, and
    handled a graph revision and two presentations correctly;
  - the real harness behaved as the stubs did.
- **Not measured**: an average saving. One pair, one task, and the agents'
  designs diverged. Whole-run cost favoured `fresh` ($1.84 against $3.02),
  but most of the gap is the run arm's extra correction attempt. On
  comparable attempts the two policies cost the same.
- **Not tested**: long human waits. p1 already showed that a retained
  session's cache is lost after an hour and the whole history is
  rewritten. Nothing here changes that.
