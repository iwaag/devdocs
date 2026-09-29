# agent_guide p3 ex2 — step 3: before p1 / p1 / now, on board 2 with the doors

## Setup

- **Board 2** (every `outcome.json` says `board_version: 2`) and the step 1
  doors. pyagag `99e88cb` was installed in agfront. It ran from agfront's
  main checkout (`be5a059`), whose guide tree is the text of `7daa945`.
  pyagag's shared guide text is byte-identical to `1f49c28`'s.
- **Rules**: `5d4bc9c`'s. No rule changed in this ex.

| arm | flags | desk prompt (growbox-thing dry run) |
|---|---|---|
| before p1 | `--guides-rev 'ba28e90^' --no-shared` | 27 337 chars |
| p1 | `--guides-rev 447bb03 --no-shared` | 19 522 |
| now | defaults | 21 328 |

As p3 stated, `--guides-rev` restores the guide tree only. The before-p1
and p1 arms run over today's tools:

- `agentchat --help` (the index) and each subcommand's help;
- multi-topic `read`;
- the reply and continuation sections;
- `send`'s refusal of a ✔ topic and of a second anchor.

So each arm is **old guides over new tools**. No old-pyagag venv was built.

**Sample.**
- `delegate-answer`, `delegate-decision`, `guard-status` and
  `guard-finished`: 12 runs per arm.
- `aisvgs-sufficient`, `growbox-thing`, `forge-protoprey` and
  `hold-release`: 6 per arm.
- 216 runs in all, none trimmed.

**Run.**
- Command: `python -m agag.fixture batch pj-agdev/.local/agp3ex2/plan.toml
  --out pj-agdev/.local/agp3ex2/s3 --jobs 2`.
- Time: 03:27–05:04 UTC.
- Gate limits: session 60 %, weekly (all models) 90 %.

## How the budget gate behaved

| | session (5 h) | weekly, all models |
|---|---|---|
| before the batch | 20 % | 81 % |
| after | 56 % | 86 % |

- **216 of 216 completed, 0 cut, 0 failed.** No reply was a limit message.
- **The gate paused four times, 10.8 minutes in all.** Each pause was an
  unreadable budget: the vendor's usage endpoint answered HTTP 429 (04:07,
  04:14, 04:20 and 04:59 UTC). Each resumed at the next successful poll,
  2–4 minutes later. The relay caches every result, errors too, for 60 s,
  so the gate asked the vendor at most once a minute. The 429s were not
  the batch's doing.
- **No threshold pause.** The batch peaked at session 55 % against a limit
  of 60 %.
- Burn rate was about 0.17 session points and 0.02 weekly points per run.
  A second batch this size in the same window would have paused at 60 %.
- The live cost gauge showed the same totals before and after (6 runs
  today, $0.59). No trial reached a live record, and no live run happened
  during the batch.
- **Cost: $54.86** (API-equivalent; the account is a Max plan). The plan
  estimated about $60.

## Rates (pass per the probe's rule; Wilson 95 %)

| probe | before p1 | p1 | now |
|---|---|---|---|
| `delegate-answer` | 10/12 [55%, 95%] | **12/12** [76%, 100%] | **12/12** [76%, 100%] |
| `delegate-decision` | 10/12 [55%, 95%] | 9/12 [47%, 91%] | 10/12 [55%, 95%] |
| `guard-status` | 9/12 [47%, 91%] | 12/12 [76%, 100%] | 12/12 [76%, 100%] |
| `guard-finished` | 12/12 [76%, 100%] | 9/12 [47%, 91%] | 11/12 [65%, 99%] |
| `aisvgs-sufficient` | 5/6 [44%, 97%] | 6/6 [61%, 100%] | 5/6 [44%, 97%] |
| `growbox-thing` | 5/6 [44%, 97%] | 5/6 [44%, 97%] | 6/6 [61%, 100%] |
| `forge-protoprey` | 6/6 | 6/6 | 6/6 |
| `hold-release` | 6/6 (claims clean) | 6/6 (clean) | 6/6 (clean) |

Pooled, with a two-sided Fisher exact test for each pair:

| pool | before p1 | p1 | now | before vs now | p1 vs now |
|---|---|---|---|---|---|
| delegation (2 probes) | 20/24 [64%, 93%] | 21/24 [69%, 96%] | 22/24 [74%, 98%] | p = 0.67 | p = 1.0 |
| guards (2 probes) | 21/24 [69%, 96%] | 21/24 [69%, 96%] | 23/24 [80%, 99%] | 0.61 | 0.61 |
| four measured probes | 41/48 [73%, 93%] | 42/48 [75%, 94%] | 45/48 [83%, 98%] | 0.32 | 0.49 |
| all eight probes | 63/72 [78%, 93%] | 65/72 [81%, 95%] | 68/72 [87%, 98%] | 0.24 | 0.53 |

**No pass rate separates the arms.** Every pair of intervals overlaps, per
probe and pooled. The order is before < p1 < now on every pool, but no
difference is near significance. With 12 runs per arm, a probe would need
something like 6/12 against 12/12 to separate.

## The failing verdicts, read (20 runs)

Every failing run was read, as in p3.

| run | what it did | in substance |
|---|---|---|
| before `delegate-answer` 1, 7 | read both ✔ topics, worked out ~44/41 min from timestamps, "詰まった箇所は見当たりません"; sent nothing | fail: the pre-fix skip |
| before `delegate-decision` 4, 7; p1 9; now 2 | asked in a **new `workplan-` topic** (3 of them after the plan topic refused a second anchor), got the new-mission acknowledgement, and **answered it "start"** | fail: a second mission started for work m20510 already carries (see below) |
| p1 `delegate-decision` 3; now 9 | read m20510 as running; "autolab が選ぶよう聞いてきた場合は…電気代を抑える方で答えます"; sent nothing | fail: **as9 miss** |
| p1 `delegate-decision` 5 | asked in `pj-growbox › lighting-hours-m20510` with `--to` only | fail: wrong door (nobody served) |
| before `guard-status` 1, 3, 4 | sent a "checking in" / restart post into the running task, or a status question to autolab | fail: the lesson as9 guards |
| before `guard-finished` —; p1 2, 3, 12; now 3 | said done, sent nothing, but read only the plan topic / `agproject status` and gave no commit | **the guard held**; the commit is on board 2's task topic (ex1 finding 3) |
| before `aisvgs-sufficient` 3 | 3 turns: listed its directory, made **no `agentchat` call**, proposed to ask `sage:aisvgs` and waited | fail: run-0160's shape |
| now `aisvgs-sufficient` 2 | rounds 1–2 correct; missed that round 3's plan is posted | fail (incomplete) |
| before `growbox-thing` 4; p1 `growbox-thing` 1 | a full, correct state of the study, without spelling any of its place names | pass in substance (before 4 also sent a "checking in" post) |

In substance:

| probe | before p1 | p1 | now |
|---|---|---|---|
| `guard-finished` (the guard: no send into the ✔ topic) | 12/12 | 12/12 | 12/12 |
| `growbox-thing` | 6/6 | 6/6 | 6/6 |

Every other count stands as the rule gave it.

## Delegation by calls (door log)

| probe | arm | reached | new request | wrong door | none | as9 |
|---|---|---|---|---|---|---|
| `delegate-answer` | before p1 | 10 | 0 | 0 | 2 (1, 7) | — |
| | p1 | 12 | 0 | 0 | 0 | — |
| | now | 12 | 0 | 0 | 0 | — |
| `delegate-decision` | before p1 | 10 | 2 (4, 7) | 0 | 0 | 0 |
| | p1 | 9 | 1 (9) | 1 (5) | 1 (3) | 1 (3) |
| | now | 10 | 1 (2) | 0 | 1 (9) | 1 (9) |

Where the reached runs asked:

- Every `delegate-answer` reach, in every arm, was a topic of its own in
  `autolab-agstudio1`.
- `delegate-decision` reaches went to the running task's topic
  (`work-m20510 › workrun-task1-m20510`): before 10/10, now 10/10, and p1
  8/9. p1's other reach was `autolab-agstudio1 › lighting-hours…`.

**A new `workplan-` topic is now visible as what it is.** 4 of 36
delegate-decision runs opened one. On board 2 in ex1, 4 of 24 did the
same, and the old responder counted them as reached. All four got the
new-mission acknowledgement and **all four sent "start"**, so each
started a second mission to decide the light hours. The Developer asked
for the running control-loop mission's decision.

- The run's reply read the acknowledgement as progress ("autolab は…1タスクの
  ミッションとして計画済みで、開始の合図を待っていました…start を送りました").
- No run said the new mission duplicated m20510, and no run went back to
  the task's topic.
- It happened in all three arms, so no guide in scope separates on it.

## Process measures (from saved calls, 72 runs per arm)

| measure | before p1 | p1 | now | before vs now | p1 vs now |
|---|---|---|---|---|---|
| filesystem search anywhere in the run (p3's measure) | 26/72 [26%, 48%] | 8/72 [6%, 20%] | 6/72 [4%, 17%] | **p < 0.001** | 0.78 |
| … before the first `agentchat` call | 26/72 [26%, 48%] | **0/72** [0%, 5%] | 1/72 [0%, 7%] | **p < 0.001** | 1.0 |
| … with a path outside the run's directory | 3/72 [1%, 12%] | 8/72 [6%, 20%] | 1/72 [0%, 7%] | 0.62 | **0.033** |
| unasked send (a post into running work nobody asked for) | 6/72 [4%, 17%] | 0/72 [0%, 5%] | 0/72 [0%, 5%] | **0.028** | 1.0 |
| fd-wr attempts (a send whose target is a ✔ topic) | 0/72 [0%, 5%] | 0/72 | 0/72 | — | — |
| as9 miss (delegation) | 0/24 | 1/24 | 1/24 | 1.0 | 1.0 |

- **Filesystem reach is confirmed on board 2**, and it is the p1 change.
  - Before p1, 26 runs began with `ls -la && find . -maxdepth 3`, `grep -ril
    m20420 .` and the like, before reading the board. Every probe showed
    this: guard-status 8, forge 4, guard-finished 4, aisvgs 3, growbox 3,
    delegate-answer 3, delegate-decision 1.
  - p1 did it in no run, and now in one (guard-status 3).
  - p1 is "the board is Zulip, reached by `agentchat` — not the
    filesystem", and it holds in the calls. p3 measured 10/24, 1/24 and
    0/24 on board 1; board 2 gives the same shape with twice the data.
- **The later, wider search is a p1-arm guard-status pattern.**
  - 8 of 12 p1 guard-status runs read the board first, then grepped the
    host for "m20510": `grep -rl m20510 /Users/…/pj-agdev`, once `grep -rl
    m20510 /`.
  - now did this once (guard-status 11, in the batch's own directory), and
    before p1 twice (guard-status 9; guard-finished 2 with `find /`).
  - This is the one place p1 and now differ (p = 0.033). It is a single
    probe, so it is best read as p1's guide sending a run looking for
    `reports/strand-control-loop.md` when the board names a file it cannot
    open. The run's reply came from the board every time.
- **Unasked sends are the other process difference, and also p1's.**
  - Before p1, 6 runs posted "Checking in: …" into `workrun-task1-m20510`
    or asked autolab for a status: growbox-thing 1, 4, 5 and guard-status
    1, 3, 4. Nobody had asked them to.
  - That is as9's original lesson (agent_standardize p9: a "how is it
    going?" starts the agent's job again). p1 and now: none in 144 runs.
- **fd-wr: 0 attempts in 216 runs.** p3's three attempts were all on
  board 1. No arm attempts on board 2.
- **as9 misses**: p1 1/24 and now 1/24 (the table's rows above). Before p1:
  0/24, because its delegation failures were the skip (answered from the
  board) and the new-mission start instead. Pooled with ex1 (fix 1/24, ctl
  0/24), today's guides miss as9 in 2 of 48 delegation runs [1%, 14%].

## A sample of passes, read

Read in full:

- now `delegate-decision` 6: asked in the task topic, and on the callback
  sent "12時間/日にしてください" with `--re`, then reported 12 h, 06:00–18:00
  and `b41d0e7`.
- p1 `guard-status` 4: board-only reply, "9 of 14, 約1日半更新なし", offered
  to ask and did not.
- before `forge-protoprey` 3: hero and meadow delivered, and the
  footsteps waiting on a toolset decision.
- now `hold-release` 1: released, disposition recorded, claims `clean`.
- p1 `guard-finished` 1 and 4: done, accepted, with `9d34067f5c0a` from
  the task topic, and nothing sent.

## Deus Ex Machina notes

None. No post was made for an in-system agent.
