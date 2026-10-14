# Agent session p1 — step 5: fresh vs run on RTS

Date: 2026-10-14. Raw evidence (driver, its logs, every attempt's stream
log, `measure.txt`/`measure.json`, the budget recorded beforehand) is in the
ignored notes `pj-agdev/.local/agent-session-p1/step5/`.

## Setup

- **Baseline.** A disposable clone of `rts-vs-bot` at its workspace commit
  `979d6a7`, plus one commit adding `improve-fresh` and `improve-run`. Both are
  byte copies of `improve-game` (default documentation policy, guide
  `agent/review.md`, plan → implement → verify → present → human-check →
  cleanup). They differ only in `execution.session`. Every run started from
  the same commit. Nothing was integrated into the shared project.
- **Harness.** claude_code 2.1.283, `claude-sonnet-5`, the host's default
  profile, the working-tree WFE and executor (`e7fbe73`…`dab1e54`).
- **Request.** 「ユニットを複数選択したとき、選択中のユニット数と兵種ごとの内訳を画面に表示してほしい。細かい見た目は任せます。」
- **Scripted trial inputs** (recorded as `Trial Person (scripted trial input)`):
  - plan → 「この計画で進めてください。」
  - first result question → correction 「見出しを「選択中」にして、兵種ごとの行は数の多い順に並べてください。」
  - re-check → acceptance 「確認しました。これで合格です。」 (pair 1 only, see the driver defect below).
- **Budget** recorded beforehand: at most 4 runs, $25 reported. Used:
  **$14.06**: fresh $5.99 plus run $8.07, the four runs below. The trial
  driver's own running total also ended at $14.06. (Corrected in ex1: this
  line said $11.06, an addition error. The per-run and per-attempt figures
  were right and are unchanged.)
- **Pairs.**
  - Pair 1: fresh first, then run; answers immediately. The two runs'
    attempts interleaved on the one slot.
  - Pair 2: run first, then fresh. The result answers were delayed 65 min,
    past the 1-hour cache TTL this subscription gets.

Costs are Claude Code's reported client-side list-price estimates (Max
subscription, `costBasis: list`), not bills and not quota. Resumed attempts
are cumulative deltas, and no reported cost was missing.

## Results

| Run | Policy | Attempts | Steps | Cache write | Cache read | Output | Active time | Reported cost |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| improve-fresh/run-001 | fresh | 4 | 105 | 186 k | 7.36 M | 50 k | 787 s | **$2.75** |
| improve-run/run-001 | run | 4 | 143 | 153 k | 18.19 M | 66 k | 1131 s | **$4.91** |
| improve-run/run-002 | run (delayed) | 4 | 98 | 201 k | 9.61 M | 42 k | 686 s | **$3.16** |
| improve-fresh/run-002 | fresh (delayed) | 4 | 121 | 214 k | 9.10 M | 54 k | 885 s | **$3.24** |
| **fresh, both pairs** | | 8 | 226 | | 16.5 M | | | **$5.99** |
| **run, both pairs** | | 8 | 241 | | 27.8 M | | | **$8.07** |

Per attempt (attempts 2–4 are the continuations):

| Pair | Fresh: first-request context, cost per attempt | Run: first-request context, cost per attempt |
| --- | --- | --- |
| 1 | 54 k each (16–22 k written); $0.40 / 0.85 / 1.03 / 0.47 | 54 k → **76 k → 148 k → 176 k** (1–2 k written); $0.45 / 2.32 / 1.52 / 0.62 |
| 2 | 54 k each; $0.53 / 0.72 / 1.47 / 0.52 | 54 k → 75 k → **95 k all written, 0 read** (after the 65-min wait) → 128 k; $0.46 / 0.61 / 1.54 / 0.54 |

No compaction occurred (the largest context was 176 k).

## What the numbers say

1. **The bootstrap a retained session saves is small.** A new session's
   first request is about 54 k tokens. About 32–37 k of it is already a cache
   hit, because the system prompt and tools are shared across sessions. Only
   16–22 k is written: the contract, guides and brief. Resuming saves that
   write and some re-orientation turns, roughly $0.05–0.10 per continuation.
2. **A retained session carries its whole history into every later
   request.** After a long implementation attempt the run session's requests
   started at 148–176 k tokens, against 54 k for a fresh one. Every step pays
   to read that. In pair 1 the run policy read 18.2 M cached tokens against
   7.4 M and cost 1.8× as much. Part of that difference is the work done, not
   the policy: its implementation attempt took 73 steps (729 s) against 37
   (218 s), because it chose to go further (below).
3. **A delayed answer erases the advantage.** After the 65-minute wait the
   resumed session's prefix had expired. Its first request wrote all 95 k
   tokens again, which is more than a fresh session writes (22 k), because
   the conversation is longer than the bootstrap. A fresh session after the
   same wait still read the shared system prefix from cache. Pair 2 ended
   even ($3.16 vs $3.24), and neither policy repaired anything.
4. **Retention does not guarantee cache.** Retention keeps the
   *conversation*, not the provider's cache. Waits for a person are routinely
   longer than an hour, and the subscription's 1-hour TTL drops to 5 minutes
   once usage credits are drawn on.

## Quality

All four runs passed `node game/test/headless.js` (31/31), presented the
result, and followed the plan and the correction.

- **improve-fresh/run-001** applied the correction literally: the heading
  became 「選択中」 and **the unit count disappeared**. That is a regression
  against the original request, which the scripted reviewer accepted
  anyway. It is the only quality defect in the four runs.
- **improve-run/run-001** kept the count (「選択中 N 体」), sorted the
  breakdown, and also wrapped the breakdown so that it no longer overlaps the
  action buttons. That is sensible, but it is unrequested extra work, and it
  is what made this run expensive.
- **Pair 2**: both kept the count (「選択中」 + 「N 体」 on a new line, or on one
  line with the breakdown), sorted the breakdown, and changed 4 lines. The
  quality is equivalent.
- No run acted on stale assumptions: each run's attempts took up the latest
  answer, and no report was refused for a stale attempt.

One retained conversation kept the original requirement where one fresh
session lost it. With one instance each this is not evidence that either
policy is better at quality.

## Defects in the trial itself

- **The driver** recognised the re-check question only on the node
  `human-check`. In pair 2 both agents asked the re-check from an added node
  (`human-check-2` / `human-check2`). The driver answered 「お任せします。進めてください。」
  instead of the acceptance. Both agents treated it as approval to finish.
  Correctly, neither recorded an acceptance decision from an ambiguous answer.
  Both runs completed without a decision. The input was the same for both
  arms, so the comparison holds.
- **Integration of pair 1** shows `needs-agreement` because the reports were
  written after the acceptance. This is p9's existing integration rule and is
  unrelated to sessions.
- The previous Claude Code session ended during pair 2's wait. The driver
  stopped with it and was restarted on the same runs. No attempt was open, and
  the delay rule was unaffected.

## Recommendation: keep `fresh` as the default

- In two matched pairs, `run` cost $8.07 against $5.99. That is one loss
  (1.8×, partly from extra work) and one tie (after a long wait). There is no
  case where it saved money.
- The saving `run` can offer is bounded by the 16–22 k bootstrap write per
  continuation. The cost it adds grows with every step of history. It also
  loses the cache entirely across real human waits.
- Small sample: two pairs, one task, one model. A workflow whose
  continuations are many, short and immediate, such as rapid back-and-forth in
  a talk node, could favour `run`. That is why it stays selectable in Detail.
  A negative cost result is a valid result.

So: an undeclared `execution.session` stays `fresh`. `run` remains a
supported, tested choice for workflows where carrying the conversation
matters more than cost. Compaction and splitting sessions per node are
untested candidates for making `run` cheaper on long runs. Neither was tried
in this phase (no compaction occurred in any run), so neither is an
established saving.
