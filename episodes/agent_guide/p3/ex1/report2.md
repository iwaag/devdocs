# agent_guide p3 ex1 — step 2: rules that hold correct spellings, and re-judging for free

## The rule changes

Three rules changed, each by spelling only:

| probe | before | now | why it is a spelling |
|---|---|---|---|
| `aisvgs-sufficient`, `growbox-thing` | `must_not: 教えてください` | `must_not_patterns: ASKS_WHAT_THE_STUDY_IS` | The failure these probes guard against is asking the Developer which or what study they mean. The patterns catch that question: "どの調査のことか教えてください", "どのプロジェクトのことでしょうか", "何を指していますか", "which study do you mean?". A closing "進捗があれば教えてください" is not that question. `どのプロジェクト` stays a plain `must_not`. |
| `growbox-thing` | the study named as `pj-growbox` | `pj-growbox` / `routine-study-growbox` / `sage:growbox` | These are the same three places `aisvgs-sufficient` already accepts for its study. "grow box" alone is still not enough, because it only echoes the question. |
| `delegate-decision` | `12 h` / `12時間` / `12 hours` / … | adds `12 時間` (in `must` and `sends_must`) | The same hour count, written with a space. |

A test pins each change in both directions:

- the courtesy passes, and the four questions fail;
- "どちらで進めるのがよろしいでしょうか" (asking for a choice) passes;
- `routine-study-growbox` names the study, and "grow box" alone does not;
- `12 時間/日` counts as sent.

Every verdict now carries `rule`, a 12-digit digest of the probe's rule.
From here on, a result says which rule judged it.

## `rejudge`

`python -m agag.fixture rejudge <out-dir>…` re-applies today's rules to
saved `outcome.json` files: the last reply, every tool call, the sends and
the number of servings. No model runs, and dry runs are skipped. It prints
one line per run and the old and new count per probe. `--json` writes every
result.

Two cases needed a decision:

- **A result judged by another question.** The first `delegate-answer` (p2
  ex1, `s3-delegate-answer`) asked about the running control loop, and its
  verdict names "4 hours / 20 s". A result is re-judged only if every fact
  its saved verdict names is still a fact of the rule: the same spellings,
  or more of them. Otherwise it is listed as `other rule` and left out of
  the counts.
- **A result from board 1.** Board 2 renamed the missions (step 1). The
  `guard-status` rule names the running mission, and board 1 called it
  m20402. `probes.for_board(probe, version)` spells the rule in the names of
  the board the run was on (`board.EARLIER_NAMES`). An outcome without
  `board_version` is board 1. Without this, all six p3 `guard-status` runs
  were first listed as `other rule`, although the standard is the same.

## Re-judged counts

**p3**: 108 runs (`pj-agdev/.local/agp3/s1`, `s3`).

| probe | runs | old rules | new rules |
|---|---|---|---|
| `aisvgs-sufficient` | 9 | 7 | **9** |
| `growbox-thing` | 9 | 8 | **9** |
| `delegate-decision` | 30 | 25 | **26** |
| `delegate-answer` | 30 | 17 | 17 |
| `forge-protoprey` | 9 | 9 | 9 |
| `hold-release` | 9 | 9 | 9 |
| `guard-status` | 6 | 6 | 6 |
| `guard-finished` | 6 | 5 | 5 |
| **all** | **108** | **86** | **90** |

Exactly four verdicts changed. They are the four p3 named as passes in
substance:

- `s1/before/aisvgs-sufficient-1`: "…round 3 をご自身で進めるとのことでしたが、進捗があれば教えてください";
- `s1/before/aisvgs-sufficient-3`: "round 3 を進められましたら、その内容を教えてください";
- `s1/before/growbox-thing-1`: names the study as `routine-study-growbox` and `work-m20402`. It gives both accepted strands with their commits and the control loop at 9 of 14;
- `s1/now/delegate-decision-5`: sent "12 時間/日にしてください", and the last reply reports 12 時間/日 (06:00–18:00) at `b41d0e7`.

With the new rules, p3's step-1 table reads:

| probe | before p1 | p1 | now |
|---|---|---|---|
| `aisvgs-sufficient` | 1/3 → **3/3** | 3/3 | 3/3 |
| `growbox-thing` | 2/3 → **3/3** | 3/3 | 3/3 |
| `delegate-decision` | 5/6 | 5/6 | 4/6 → **5/6** |

The substance column p3 wrote by hand is now the verdict.

**p2 ex1**: 21 runs (`pj-agdev/.local/agp2ex1`, dry runs skipped).

| | runs | old rules | new rules |
|---|---|---|---|
| judged | 21 | 18 | 18 |
| other rule | 1 (`s3-delegate-answer`, the first version) | — | — |

No verdict changed. The three failing verdicts are `delegate-answer` runs
that did not reach autolab (`s3-delegate-answer-3`, `s5/delegate-answer-2`
and `-4`), which is the failure, not a spelling.

## Reading the passes

- The four new passes were read in full (above). Each is the answer the
  probe asks for.
- The new patterns were run over every saved reply: 171 of them, from p2,
  p2 ex1 and p3. They match none. So no pass became a fail, and no run in
  the record asked the Developer what a study is.
- The "choice" sentence in the test is why the first draft of the pattern
  was narrowed. That draft, bare "どちら…でしょうか", would have failed a reply
  asking the Developer to choose. That would have been a changed standard,
  not a spelling.

Results: `pj-agdev/.local/agp3ex1/rejudge-p3.json`, `rejudge-p2ex1.json`
(ignored).
