# agent_guide p2 ex1 — step 5: run it

Every p2 probe was served through the versioned drivers from the **main
checkouts**, with the new guides, plus the two delegation probes and one
baseline. Runner: `pj-agdev/.local/agp2ex1/run_s5.sh`; outcomes under
`pj-agdev/.local/agp2ex1/s5/<probe>/outcome.json` (ignored). Final version
set: pyagag `3a16479` everywhere, comfynotify included.

## Results

| probe | agent / role | result | turns | cost | p2 (new guide) |
|---|---|---|---|---|---|
| `aisvgs-sufficient` (#15673) | Front desk | pass; **observed the decision** | 15 | $0.28 | pass, 11, $0.19; decision not observed |
| `growbox-thing` | Front desk | pass | 8 | $0.17 | 9, $0.18 |
| `forge-protoprey` | Front desk | pass | 8 | $0.15 | 10, $0.16 |
| `receipt-owed` | Front desk | pass | 8 | $0.21 | 6, $0.18 |
| `hold-release` | Front desk | pass | 8 | $0.19 | 3/3 |
| `delegate-answer` ×6 | Front desk | **3 pass, 2 fail, 1 void** (below) | 10–15 | $1.72 | new |
| `delegate-decision` ×2 | Front desk | 2 pass (3 servings each) | 18, 16 | $0.90 | new |
| `aisvgs-sufficient`, `--guides-rev 447bb03 --no-shared` | Front desk (p1's composition) | pass; decision not observed | 7 | $0.18 | old: 8, $0.21 |
| `archsage-loose-sage` | archsage (Fable 5.1) | pass | 7 | $1.14 | 6, $1.15 |
| `planner-other-project` | autolab planner | pass | 21 | $0.46 | 20, $0.28 |
| `entrance-plans` | autolab entrance | pass | 12 | $0.17 | 13, $0.35 |
| `triage-unopened` | Observer triage (local model) | pass (`legit`) | 2 | — | pass |

Step 5 spent **$5.57** (21 runs; the triage ran on the local model).
Step 3's four real runs spent $1.30, so the ex totals **about $6.9**. The
plan expected about $6 for the full set.

### The baseline works

`--guides-rev 447bb03 --no-shared` built p1's composition from git: 19 534
characters, as p2 measured it, with the tree extracted to
`s5/aisvgs-baseline-447bb03/guides@447bb03/`. It passed p1's rule in 7 turns
and, as in p2, did not carry the Developer's decision to do round 3 by
hand.

### The #15673 probe observed the decision, once

With the new guides this run read `archsage-agstudio1 › study-aisvgs-round3`.
It went there after `agentchat intro archsage` and `topics
archsage-agstudio1`. Its reply says: 「開発者ご自身が『round 3 は自分で手を動かしてやる』と明言してroutineは回さない方針になりました（#20041, archsage応答#20042）」.
That is p1's open finding 1 (a decision recorded only in archsage's channel).
In p2 neither guide found it. One run is not a rate; the observed fact is
reported as observed and nothing is concluded from it. The baseline run
beside it did not find it.

### Delegation

`delegate-decision` passed both times, with the whole loop:

1. Front opened a `workplan-growbox-control-loop-lighting-hours` topic, or
   posted in `work-m20402 › workrun-task1-m20402`.
2. The next serving answered autolab's `response_request … ask=decision`
   there with the 12-hour choice (`send --intent report --re <id>` once).
3. The last serving reported what autolab set.

`delegate-answer`, six runs:

| run | result | where Front asked |
|---|---|---|
| 1 | pass | autolab's channel (its entrance), topic `growbox germination-days (m20390)` |
| 2 | fail | asked nobody: read `✔ workplan-growbox-germination-days`, answered "about 45 minutes, no stall on record", did not post into the ✔ topic |
| 3 | **void** | `pj-growbox › growbox-germination-days-recap` with `--to autolab-agstudio1`, and no mention |
| 4 | fail | as run 2 |
| 5 | pass | autolab's channel, `question-m20390-timeline` |
| 6 | pass | autolab's channel, `question-m20390-germination-days` |

Run 3 exposed a responder defect. The responder counted the `to=11` of the
post's `ag-post` line as addressing autolab and answered it. But
`agentchat send --to` adds only that line, and a listener is served by a
mention or by a topic it owns (`agag.listen`: its channel, its prefixes).
The real autolab would never have seen that question. Fixed in pyagag
`3a16479`: only a mention, the agent's channel or its topic prefix takes a
script line, with a test (`test_a_to_line_alone_reaches_nobody`).
Runs 4–6 were run after the fix.

So, among the five valid runs, **Front delegated in 3**. The other 2 ended
the same way as step 3's third run: the question was about finished work,
Front read the ✔ mission topic, did not post into it (fd-wr: ✔ is
finished), answered from the record ("no stall on record") and asked
nobody else. That record does not say what the Developer asked, and the
reply does not claim that it does. Across steps 3 and 5 the rate is 3 of 6
with the current probe and responder. No guide was changed (the ex does not
change guides). This is a finding for the phase that owns the text (see
report.md).

## A trial touched nothing real

- **Checkouts**: after the start stamp, 0 new files under `.local/agent/`
  and `.local/topics/` of agfront, agautolab, agobserver and archsage, and
  no `*fixture*` topic directory in any of them. Every serving's workspace
  and record is under `s5/<probe>/records/<agent>/.local/`.
- **The relay's cost gauge** (`GET /cost` before and after): records per
  root unchanged (front 1682, autolab 1031, agforge 186, arxivsage 13,
  agecho 2), and today's totals 0 runs / $0.00 both times, while about $6
  of trial runs happened in between. No live serving ran in that window, so
  the comparison is clean.
- **Gitea**: `agproject status aisvgs` inside the #15673 run answered
  `main: https://gitea.fixture.invalid/autodev/aisvgs.git at f57eed1a27de`,
  read from the run's Claude Code session log.

## Suites (summary lines, main checkouts, pyagag `3a16479`)

| suite | passed |
|---|---|
| pyagag | 1170 |
| agfront | 198 |
| agautolab | 334 |
| agobserver | 178 |
| archsage | 41 |
| comfynotify | 28 |
| agentroom relay | 363 |
| agforge | 265 |
| cagent | 204 |

No listener was restarted for the pin. The in-process changes are inert
unless `AGAG_RECORDS_ROOT` is set or a store is a trial overlay, and the
`agentchat` and `agproject` CLIs load per call. The notifier was restarted
twice (step 4). `nctl status`: ok.
