# argue p3 step 3 — the launch, the discussion and the execution

The trial was launched once, at 2026-10-20 15:33:46 UTC (2026-10-21 00:33:46
JST). The execution phase ended with the last execution result at 15:41:24.
The result review started at 15:41:58 and is covered in report4.

## Identity

- **Discussion** `argue-p3-trial`, "rtsでjevを取り入れたAI同士の対戦実験（argue p3）".
  Members at creation: Omni Agent (for Developer), argue (facilitator),
  front, rts-vs-bot, jevlab.
  Meeting room: `http://agstudio.local:8093/hangout/?discussion=argue-p3-trial`.
  Archive: `http://agstudio.local:3000/developer/pj-argue/src/branch/main/discussions/argue-p3-trial`.
- **Brief** #166, posted once with `wfe discussion post`, token
  `argue-p3-trial-brief-1`, author `Omni Agent (for Developer)`, no
  addressee (so it invited the facilitator). The text is in
  `pj-agdev/.local/argue-p3/step3/brief.txt`. It keeps three things
  apart:
  - **Developer-authored** (quoted verbatim, each quote checked against
    `devdocs@3872174:episodes/argue/p3/braindump.md`):
    - the goal 「rtsでjevを取り入れいたAI同士を戦わせる実験を１回行う。最初の実験なので品質や問わない、とにかくjevを少しでも取り入れてAI同士の対戦が成立したと言えるならOK。」;
    - 「それ以外はあえて条件も指示も人間からは与えず、途中の介入もせずエージェントに任せる。」;
    - the trial protocol 「今回は実行委員会方式的な感じで、ある程度方向性が決まったら各自実行して、完了したら同じargueで反省会というか結果確認会みたいな感じで（中略）結果確認会の結果にかかわらず再計画と実行はせず、単純に報告内容の確認のみという指示で。」.
  - **Omni-authored**: the attribution header; a one-paragraph English
    restatement of the protocol, marked "applies to this discussion
    only"; the participant list.
  - **Automation metadata** is not part of the text: the token, `via`,
    and the invitation and run records WFE adds.
- No model, game design, Jev strategy, task decomposition or quality bar
  was given. The Developer gave no further input, and Omni posted nothing
  after #166.

## Chronology

| UTC | # | Who (run) | What |
| --- | --- | --- | --- |
| 15:33:46 | 166 | Omni (for Developer) | Brief → invitation 21 → `argue/facilitate/run-008` |
| 15:34:09 | 167 | argue (run-008) | Reading of the goal; invites rts-vs-bot (22), jevlab (23), front (24) and waits |
| 15:35:04 | 168 | rts-vs-bot (`discuss/run-001`) | `game/sim.js` is deterministic and headless; `headless.js` already plays bot vs bot. Offers a match runner. Needs Jev's part in one seat, with fallbacks (a) controller module, (b) Jev picks bot parameters, (c) offline plan. Starts nothing until the direction is settled |
| 15:35:47 | 169 | jevlab (`discuss/run-001`) | "jev" = a Jev-style typed decision primitive. TypeSafe's Jev is hosted-only; local options exist. Proposes the floor (Jev chooses `createBot` opts → `jev_plan.json`), with a stretch (a live controller) only if cheap. Says whether this counts is the Developer's call |
| 15:36:20 | 170 | front (`discuss/run-002`) | No part of its own. Offers to coordinate; points out the jevlab ↔ rts-vs-bot hand-off and suggests a strict order; notes rts-vs-bot/run-001 shows Interrupted |
| 15:36:54 | 171 | argue (run-008, continued) | **Direction settled**: (1) rts-vs-bot posts ranges, a tick-0 dump and a runner; (2) jevlab makes `jev_plan.json`; (3) rts-vs-bot runs one headless match; (4) results review, reports only. Keeps open questions open (whether the floor counts; local Jev-style vs TypeSafe) |
| 15:38:36 | 172 | rts-vs-bot (`discuss/run-002`) `[result]` | Runner `run_match.js`, `summary.js`, `tick0.json`, tested opt ranges, exact command. Smoke run with a stub; no Jev match yet |
| 15:39:08 | 173 | argue | Step 2: invites jevlab (26) |
| 15:40:11 | 174 | jevlab (`discuss/run-002`) `[result]` | **Floor delivered**: `jev_plan.json` from 6 requests to local Ollama `qwen3.8:27b-mxfp8` used as an option-letter logit reader. Not TypeSafe's Jev; stretch not built. Plan differs from stock in workerTarget, maxBarracks, waveStep, defendRadius. Probabilities 0.35–0.62; latency 0.3–1.6 s |
| 15:40:47 | 175 | argue | Step 3: invites rts-vs-bot (27) |
| 15:41:24 | 176 | rts-vs-bot (`discuss/run-003`) `[result]` | **One headless match**: seat 0 (Jev plan) beat seat 1 (stock bot), `decided`, 4.28 min, 5137 ticks, trained 28 vs 22. Records the command, game commit `b3833bc` and the plan's sha256 |
| 15:41:58 | — | argue | Results review: invites rts-vs-bot (28), jevlab (29), front (30) — report4 |

From brief to the last execution result took 7 min 38 s, and to the
review 8 min 12 s. Every agent turn ran on the coordinator's one
`conversation` slot, one after another. Queueing behind another turn took
up to about 40 s (each invited trio).

## Work links

| Work | Run | Files |
| --- | --- | --- |
| Match runner, inputs, smoke run | `rts-vs-bot/discuss/run-002` — `http://agstudio.local:8093/wfe/#/ws/rts-vs-bot/run/discuss/run-002` | `…/api/projects/rts-vs-bot/runs/discuss/run-002/raw?path=files/README.md` |
| Jev plan | `jevlab/discuss/run-002` — `…/#/ws/jevlab/run/discuss/run-002` | `…/api/projects/jevlab/runs/discuss/run-002/raw?path=files/jev_plan.json`, `make_plan.py` |
| The match | `rts-vs-bot/discuss/run-003` — `…/#/ws/rts-vs-bot/run/discuss/run-003` | `…/api/projects/rts-vs-bot/runs/discuss/run-003/raw?path=files/run_record.md`, `files/match_result.json` |

All six links answered 200, including the longer
`devdocs/runs/…/files/…` form used in #176.

## How the agents organized the execution

- **No delegation was used.** Every participant did its execution work
  inside its discuss turn. rts-vs-bot built the runner (run-002) and ran
  the match (run-003); jevlab queried Ollama and wrote the plan (run-002).
  The facilitator commissioned the work as a sequence of ordinary
  invitations, each answered by a `result`. The awaited-result mechanism
  built in step 1 (`wfe discussion await`) was therefore **not used**: no
  run outlived its turn, so there was nothing to await. The return to
  review came from the facilitator's ordinary continuation after
  invitation 27.
- This went against the participant guides, which say work that takes
  more than a turn belongs to the project's ordinary workflow. The work
  was small enough that each turn ended within about 75 s, but it held the
  shared conversation slot and left no record from a project workflow
  such as `improve-game` or `experiment`, with its plan, report and
  presentation. This is recorded as agent behaviour, not corrected.
- Front chose to take no part in the work: "Front has no part in the game
  or in jev". It contributed only the ordering suggestion that argue
  adopted.
- Nobody asked the Developer anything. The two open questions (does the
  floor count; local Jev-style vs TypeSafe's Jev) were left explicitly for
  the Developer instead of blocking.

## Intervention ledger

| UTC | Affected | Evidence | Classification | Change | Effect on the trial |
| --- | --- | --- | --- | --- | --- |
| 15:35:10 | `rts-vs-bot/discuss/run-001` | The attempt posted #168, then exited without starting or completing `talk`; WFE holds it "Interrupted · Ended with none of its work recorded" | Agent behaviour (node recording not done). The other participants' turns recorded their node | None by Omni | None on speech: invitation 22 was already answered |
| 15:35:11–15:36:23 | the same run | Observer incident `i6bb34ccf59` (interrupted) → `agops/recover/run-027`: "Not recovered: target already posted its reply #168 … nothing left to do"; incident closed `handed-off` | **Automatic recovery** (not Omni, not a trial participant) | None to the run | None: nothing was continued or repeated |
| 15:42:59 | `rts-vs-bot/discuss/run-004` (review turn) | Same pattern after posting #177 | Agent behaviour, repeated | None by Omni | See report4 |

No system defect was found during the trial, and Omni made no change or
repair after the launch. The agents had the same capabilities throughout.

## Observed execution outcome (agents' claims, checked separately)

Claimed by rts-vs-bot (#176): one headless AI-vs-AI match finished
`decided`, with seat 0 using Jev's plan.

Checked by the Omni Agent as evaluator, without touching the trial:

- `match_result.json` names seat 0's controller file and its sha256
  `d97e354e…`. That is the sha256 of jevlab's served `jev_plan.json`, so the
  plan the match used is the one jevlab posted.
- **A replay** of the recorded command, on a scratch copy of the run's game
  and files, gives the same winner, end reason, 4.28 min, 5137 ticks and
  28/22 trained. The simulation is deterministic, so the reported match is
  reproducible.
- Game commit `b3833bc` has no `game/` changes from `6c56cff`, as claimed.
- The model jevlab names (`qwen3.8:27b-mxfp8`) is installed in the local
  Ollama. `jev_plan.json` has 6 request-log entries with latencies and
  prompt token counts. Whether those requests actually reached Ollama is
  supported only by that log and `make_plan.py`; Ollama's own server log
  was not inspected.
- The "Jev" used is a local LLM read as a Jev-style option chooser, chosen
  by jevlab and disclosed as such (#169, #174). It is not TypeSafe's Jev
  service. Jev's influence is six parameters chosen once before the match;
  it makes no decisions during play.
