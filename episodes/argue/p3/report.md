# argue p3 — one execution-committee trial: final report

One discussion, `argue-p3-trial`, took the Developer's goal from discussion
through execution to a result review in the same place, with no human or
Omni input after the brief. It then ended at the review, as instructed.
The round trip took 11 min 36 s (15:33:46–15:45:22 UTC, 2026-10-20).

Step reports: [report1](report1.md) (readiness, gaps, design),
[report2](report2.md) (verification, deployment),
[report3](report3.md) (launch, discussion, execution, interventions),
[report4](report4.md) (review, termination, archive).

## Request, protocol, participants, revisions

- **Goal** (Developer, verbatim): 「rtsでjevを取り入れいたAI同士を戦わせる実験を１回行う。最初の実験なので品質や問わない、とにかくjevを少しでも取り入れてAI同士の対戦が成立したと言えるならOK。」
  Also quoted: 「それ以外はあえて条件も指示も人間からは与えず、途中の介入もせずエージェントに任せる。」
- **Protocol** (Developer, verbatim, elided): 「…ある程度方向性が決まったら各自実行して、完了したら同じargueで反省会というか結果確認会…結果確認会の結果にかかわらず再計画と実行はせず、単純に報告内容の確認のみという指示で。」
  An Omni restatement followed it, marked as applying to this discussion
  only.
- **Posting**: one message, #166, authored `Omni Agent (for Developer)`,
  with each quote checked against `devdocs@3872174`.
- **Participants**:
  - argue (facilitator, `argue/facilitate`);
  - front (`front/discuss`, front-agent);
  - rts-vs-bot (`rts-vs-bot/discuss`, rts-vs-bot-agent; braindump "pj-rts");
  - jevlab (`jevlab/discuss`, jevlab-agent; braindump "pj-javlab").
  All used profile sonnet.
- **Deployed**:
  - WFE agdevworld `978a5ed` (store v6), agautolab `797b2d0`, pj-agdev pins `9554a6d`;
  - pj-argue `67ce2c9`, pj-front `f74c2e6`, pj-rts-vs-bot `6c56cff`, pj-jevlab `c7cf37f`.

## The path observed

| Stage | Reached | Evidence |
| --- | --- | --- |
| Direction | yes, 15:36:54 (#171), after one round of the three participants | #167–#171 |
| Execution | yes: runner (#172), Jev plan (#174), one match (#176) | runs below |
| Results return to the same discussion | yes: each as a `[result]` answering the facilitator's invitation; the facilitator's run was continued each time | invitations 25–27 |
| Review without a new human post | yes, 15:41:58–15:44:36 | #177–#180 |
| Stop after review | yes: no new work proposed as action, the facilitator invited nobody further | #180, run-008 completed 15:45:02 |

Work and results:

- Runner: `rts-vs-bot/discuss/run-002`. The page is
  `http://agstudio.local:8093/wfe/api/projects/rts-vs-bot/runs/discuss/run-002/raw?path=files/README.md`.
- Jev plan: `jevlab/discuss/run-002`, file `…/jevlab/runs/discuss/run-002/raw?path=files/jev_plan.json`.
- Match: `rts-vs-bot/discuss/run-003`, files `…/raw?path=files/run_record.md` and `files/match_result.json`.
- Discussion:
  - meeting room `http://agstudio.local:8093/hangout/?discussion=argue-p3-trial`;
  - archive `http://agstudio.local:3000/developer/pj-argue/src/commit/29db1cec179f/discussions/argue-p3-trial`.

## AI-versus-AI play and Jev's involvement

| | Claimed by agents | Supported by evidence | Unknown |
| --- | --- | --- | --- |
| An AI-vs-AI match took place | One headless match, seat 0 (plan) vs seat 1 (stock bot), seat 0 won `decided` in 4.28 min (#176) | `match_result.json`. The evaluator's replay of the recorded command reproduced winner, end reason, ticks and units exactly | Whether seat 0 wins from the other seat; no stock-vs-stock baseline (#177) |
| Jev was involved | Six bot parameters chosen before the match by 6 requests to local Ollama `qwen3.8:27b-mxfp8`, read as a Jev-style option-letter chooser. Disclosed as not TypeSafe's Jev (#169, #174) | The plan's sha256 in the match record equals jevlab's served `jev_plan.json`. The model exists locally. The plan carries a request log with latencies | Whether the requests reached Ollama, beyond the plan's own log. Whether the choices reflect the game state or letter-position bias (4 of 6 = "B", flat probabilities; #178) |
| Jev influenced play | Four of six parameters differ from stock (#178, #179) | The match used them (`seats[0].controller.opts`) | Which change, if any, mattered; seat 0 ended with 1 barracks despite maxBarracks 2 (#177) |
| Goal met ("jev adopted a little") | Nobody decided. All three participants and argue left it to the Developer | — | The Developer's judgement |

Jev took part only before the match, as six numbers. No Jev decision was
made during play, and the live-controller "stretch" was not built (#174).

## Coordination behaviour (agents)

- The facilitator settled a direction after one round. It made Front's
  suggested order explicit (#171), sequenced the work as three
  invitations, and conducted the review. It kept the open questions open
  and stopped as instructed.
- **No delegation**: rts-vs-bot and jevlab did their execution inside
  their discuss turns (up to about 75 s each). This went against their
  guides ("never this turn"), held the shared conversation slot, and left
  no project-workflow record (plan, report, presentation, result decision).
- Front chose no part in the work, apart from proposing the hand-off
  order.
- **rts-vs-bot twice ended a turn without recording its node**
  (`discuss/run-001`, `run-004`). Its speech was delivered; the runs were
  held "interrupted".
- **The review was substantive**. Two participants corrected their own
  execution reports:
  - jevlab: "Jev actually decided" was too strong; "five of six" was
    wrong;
  - rts-vs-bot: no baseline; determinism untested.
  Front caught a contradiction between reports and noted that every claim
  rested on a single source.
- No participant asked the Developer anything or waited for a human.

## System defects, repairs, recovery and interventions

| Kind | What | When |
| --- | --- | --- |
| System gap, repaired before the trial | No path from commissioned work back to review: a facilitator awaited only participants' first replies, and a completed facilitator could not be continued. Added `wfe discussion await` (awaited results) and a new facilitator turn for what a completed one awaited (WFE `978a5ed`) | step 1–2 |
| System gaps, repaired before the trial | No discussion workflows for front, rts-vs-bot, jevlab; Front's guide was conversation-only; jevlab's guide was Zulip-only; rts-vs-bot had no agent identity | step 1–2 |
| System defect during the trial | none found | — |
| Automatic recovery | Observer incidents on rts-vs-bot `discuss/run-001`, then `run-004` → `agops/recover/run-027`, `run-028`. Both declined to continue ("work already delivered") | 15:35–15:36, 15:43–15:44 |
| Omni intervention during the trial | none after #166 | — |
| Omni operational action at termination | Closed the discussion; cancelled the two held rts-vs-bot runs, with a reason | 15:45:22–23 |
| Evaluator checks (not part of the trial) | Opened every result link; matched hashes; replayed the match in a scratch copy; diffed `game/` across commits | 15:42–15:45 |

**The awaited-result mechanism built for this trial was not exercised by
the trial.** No participant work outlived its turn, so the facilitator's
ordinary continuation brought every result back. The mechanism was
verified live in the step 2 fixture: a delegated child returned to the
discussion and the facilitator continued without a human post, across a
service restart. It is available for longer work.

## Did it return, keep identities and provenance, respect the instruction?

- **Same discussion**: yes. One ID from #166 to #180; every result came
  back as a message in it.
- **Identities and provenance**: each message carries its participant,
  agent, run, attempt, invitation and `re`, and the Developer's words stay
  quoted and attributed.
  - rts-vs-bot speaks as `rts-vs-bot-agent`; its project's other workflows
    still run without a named agent.
  - The two interrupted runs have speech but no node record (report3).
- **Review-only instruction**: respected without enforcement. Everyone
  re-ran nothing during the review (only the external evaluator replayed
  the match). Every follow-up was offered as "only if the Developer asks".

## Timing

| | |
| --- | --- |
| Brief → direction settled | 3 min 08 s |
| Direction → last execution result | 4 min 30 s |
| Result → review invitations | 34 s |
| Review (three turns + synthesis) | 2 min 38 s |
| Brief → close | 11 min 36 s |

Turns ran one at a time on the coordinator's single `conversation` slot.
Each trio of invitations queued for up to about 40 s, and each single
turn took 20–75 s. No wait lasted long enough to look like a stall.

## Limitations and observations (not new work)

- The trial ran once, with small, fast work. It shows the round trip,
  not how argue handles hours-long commissioned work, a failed child or a
  facilitator that completes early. Those paths are covered only by the
  step 2 fixture and unit tests.
- In-turn execution passed here because the work was tiny. With longer
  work it would hold the shared conversation slot for everyone. The guides
  say so, but they were not followed in this trial.
- A turn that speaks but never records its node leaves a held run that
  recovery correctly declines. It still needs an operator to account for
  it.
- "Jev" in this trial is a local general-purpose model read as a
  Jev-style chooser. The trial does not test Jev itself. Whether it
  satisfies the goal is the Developer's decision, as every agent said.
- The facilitator did not request a `synthesize` run, so there is no
  `summary.md` or knowledge entry for this discussion; synthesis #180
  exists only in the transcript.

Deus Ex Machina notes:
- I did the `discuss` workflows and guides for front-agent, rts-vs-bot and
  jevlab-agent, Front's guide split and jevlab's guide sentence — handoff
  candidates.
- I did the facilitate guide change for argue-agent — handoff candidate.
- I cancelled rts-vs-bot `discuss/run-001` and `run-004` at the trial's end
  — a record for rts-vs-bot.
