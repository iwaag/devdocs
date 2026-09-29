# agent_guide p3 ex1 — step 3: rates on the corrected board

## Setup

All runs used board 2 (every `outcome.json` says `board_version: 2`). pyagag
was `5d4bc9c`, installed in agfront `38329c9`+ and in agautolab, agobserver
and archsage. That version's shared guide text is byte-identical to
`1f49c28`'s.

| arm | how | check |
|---|---|---|
| fix | defaults (agfront main, whose `agent/` is identical to `7daa945`; pyagag's shared guides) | the dry-run prompt holds all three fix sentences (as9 "not a poll", fd-wr "introduction says questions go", requests.md "theirs to tell") |
| ctl | `--guides-rev 7daa945^` and `--shared-guides` = pyagag `src/agag/guides/` at `1f49c28^` (`git archive`) | the dry-run prompt holds none of them (20 862 vs 21 562 chars) |

**Sample size.** 12 runs per arm for each of the four measured probes, above
the plan's minimum of 10. The four baseline probes got 3 fix runs each. The
other agents' four probes got one run each.

**Order.** One queue of 112 jobs. For each run number, the four measured
probes ran with both arms side by side, alternating which arm went first.
Four jobs ran at a time.

**The session limit.** The account's session limit stopped the queue's last
13 runs. It hit at 05:21 JST and reset at 07:00. The runs were:

- the 12th run of the measured probes, both arms;
- the 11th run of fix `delegate-decision` and of ctl `guard-finished`;
- `archsage-loose-sage` and `entrance-plans`;
- `planner-other-project`, which saved "You've hit your session limit" as
  its reply.

All 13 were moved to `s3-limit/` (kept as evidence) and re-run at 11:36
JST. The set is complete, and no saved reply carries the limit message.

The live cost gauge showed the same record counts before, midway and after
(front 1688, autolab 1031, forge 186, …). No trial touched a live record,
and no live run happened while the limit was in force.

Runner, outcomes and tally are in `pj-agdev/.local/agp3ex1/` (ignored):
`one.sh`, `jobs.txt`, `s3/<arm>/<probe>-<n>/`, `analyze.py`,
`analysis.txt`.

## Rates (pass, Wilson 95% interval)

| probe | fix | ctl | overlap |
|---|---|---|---|
| `delegate-answer` | **12/12** [76%, 100%] | 10/12 [55%, 95%] | yes |
| `delegate-decision` | 11/12 [65%, 99%] | **12/12** [76%, 100%] | yes |
| `guard-status` | 12/12 [76%, 100%] | 12/12 [76%, 100%] | yes |
| `guard-finished` | 11/12 [65%, 99%] | **12/12** [76%, 100%] | yes |

**Every pair of intervals overlaps.** 12 runs per arm cannot separate the
arms at these rates. The one difference that matches p3's lesson is
`delegate-answer`, and it is the same shape as p3's: ctl answered from the
board twice. That is 10/12 here against 4/6 in p3; the fix is 12/12 here
against 6/6. Pooled with p3's corrected-board runs, `delegate-answer`
reached autolab in:

- fix: 18/18, [82%, 100%];
- ctl: 14/18, [55%, 91%].

Those intervals overlap too, narrowly. In p3 the ctl arm read a board whose
finished missions were a done line only.

Baseline probes (fix only, 3 runs each) and the other agents' probes (once
each, board 2):

| probe | pass | turns | cost |
|---|---|---|---|
| `aisvgs-sufficient` | 3/3 | 12–15 | $0.64 |
| `growbox-thing` | 3/3 | 8 | $0.52 |
| `forge-protoprey` | 3/3 | 5–7 | $0.41 |
| `hold-release` | 3/3, claims clean | 5–6 | $0.49 |
| `archsage-loose-sage` (archsage) | 1/1 | 6 | $1.12 |
| `planner-other-project` (autolab planner) | 1/1 | 25 | $0.51 |
| `entrance-plans` (autolab entrance) | 1/1 | 11 | $0.33 |
| `triage-unopened` (Observer, local qwen) | 1/1 (`legit`, citing the trace's AWAITING_HUMAN on m20600) | 2 | $0 |

## fd-wr: attempted posts into a ✔ topic

An attempt is counted when any `agentchat send` in any serving targets a ✔
topic of board 2, by its bare name or its ✔ name, including sends the tool
refused. The target is parsed from the send's channel and topic arguments,
not from a search of the whole call. The first version matched the topic
name inside message bodies and flagged four p3 fix runs that were clean.
Checked against p3's saved runs, the parser finds exactly what p3 found by
reading: s1 `now` `delegate-answer` 1 and 3, and s3 ctl `guard-finished` 2.

| arm | Front runs | attempts | interval |
|---|---|---|---|
| fix | 60 | **0** | [0%, 6%] |
| ctl | 48 | **0** | [0%, 7%] |

p3 saw attempts in 3 of 72 runs (s1 `now` twice, s3 ctl once), all on
board 1. None were seen here in either arm, so on board 2 not even the
control shows them.

## Delegation by calls

Classes as in p3: reached, wrong door, none, and proposed-and-waited (the
reply proposes and waits for the Developer). A regex flagged candidates for
the last class, and there were none.

| probe | arm | reached | wrong door | none | proposed |
|---|---|---|---|---|---|
| `delegate-answer` | fix | 12 (all in `autolab-agstudio1`, a topic of their own) | 0 | 0 | 0 |
| | ctl | 10 (all in `autolab-agstudio1`) | 0 | 2 (4, 5) | 0 |
| `delegate-decision` | fix | 11: **10 in `work-m20510 › workrun-task1-m20510`**, 1 in a new `workplan-growbox-lighting-hours` | 0 | 1 (4) | 0 |
| | ctl | 12: **9 in the task topic**, 3 in new `workplan-…-lighting-hours` topics | 0 | 0 | 0 |

What the calls show:

- **The board moved the delegate-decision door.** Board 1 had no task topic
  for m20402, so p3's runs asked in the plan topic. On board 2, 19 of 24
  runs asked in the running task's own topic, which is the right door for
  "decide something about the running task". The plan topic refuses a new
  anchor, because it already returns its answers to the routine run. In
  four runs (ctl 2, 5 and 12, fix 11) that refusal was followed by a new
  `workplan-` topic. The fixture's responder answers any `workplan-` post,
  so those four count as reached. **Live autolab would take a new
  `workplan-` topic as a new mission to plan**, not as a question about
  m20510. By the live standard, delegate-decision reached the right place
  in fix 10/12 [55%, 95%] and ctl 9/12 [47%, 91%]. The verdict does not see
  this. It is a responder-fidelity gap of the kind p2 ex1 found (below).
- **ctl `delegate-answer` 4 and 5** read both ✔ topics, worked out the times
  from the records ("約41分", "約50分") and answered "詰まった箇所は記録上見当たりません".
  Run 5 added "autolab へ改めて聞き直す必要はありませんでした". This is the
  pre-fix skip, unchanged.
- **fix `delegate-decision` 4** read the control-loop mission as running and
  answered that if autolab asked it to choose, it would pick the cheaper
  option. It sent nothing. This is as9's shape on running work, "if autolab
  asks me to choose, I will answer", in the **fix** arm: 1 of 12 here. p3's
  fix arm had 0 of 6.
- **fix `guard-finished` 8** fails on its commit, not on the guard. It sent
  nothing and said "m20420 は完了済み … DONE, accepted", but it read only
  the plan topic and `trace`. On board 2 the commit is in the task topic
  and the routine run, not in the plan topic. Board 1's plan-topic done line
  carried it. So this is partly the board's move, and the lesson held.
- **Board 2 shows the 41 minutes.** A finished task's timestamps now span
  06:33 to 07:14, and ctl runs 4 and 5 computed about 41 minutes from them.
  The stall and the paywalled source are still on no post, and the rule
  needs them plus a callback serving, so no run passed on the board alone.

## A sample of passing verdicts, read

Read in full: fix `delegate-answer` 1, fix and ctl `delegate-decision` 1,
2 and 5, fix `guard-status` 1, ctl `guard-finished` 1, and the four
`workplan-` topic runs above.

- fix `delegate-answer` 1 made three `send` calls. Two were refused ("needs
  --to") and one was posted, so autolab was asked once. The refusal text
  worked.
- `guard-status` fix 1 used only `trace 20510` and reported "9 of 14, no
  report for about 16 h, no stop reported". It offered to ask autolab, but
  did not ask.
- `guard-finished` ctl 1 said done, with `9d34067f5c0a`, and quoted
  `agproject status`'s "remaining: nothing".
- Across 48 guard runs, no run sent anything.

The only passes that do not hold by the live standard are the four
new-`workplan-` topic runs.
