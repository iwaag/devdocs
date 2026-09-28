# agent_guide p3 — report

## Outcome

- **The pre-p1 guides were measured for the first time.** 72 runs, three
  compositions, six probes, $18.77. The rewrite did not make Front worse
  at anything measured. On delegation it made Front better:
  `delegate-answer` reached autolab in 1/6 runs before p1, 2/6 with p1's
  guides, and 4/6 now.
- **The skip is older than p1, and it is fixed.** Three sentences were
  scoped, and none deleted. Front then asked the agent it was told to ask
  in 6/6 `delegate-answer` and 6/6 `delegate-decision` runs, against 4/6
  and 5/6 with today's guides on the same board. Both guards stayed clean,
  3/3 each. The fix is deployed (pyagag `1f49c28`, agfront `7daa945`).
- **The fixture was poorer than the realm where it mattered.** Its autolab
  introduction had lost the live one's door for questions, so runs that
  did delegate asked where nobody listens. That is corrected (pyagag
  `3c75113`), and measured separately from the guide fix.
- **p7's reader measurement is versioned**: `python -m agag.fixture reader`.
  Today's reader reads 44/52 as wanted, deterministically, in 0.29–3.68 s.
  The misses are the two over-readings p7 already recorded and the judge
  absorbs.

Step reports:

| report | covers |
|---|---|
| `report1.md` | three arms |
| `report2.md` | cause |
| `report3.md` | fix and guards |
| `report4.md` | reader probe |
| `report5.md` | deployment and suites |

## The three-arm table

Runs from agfront's main checkout, pyagag `aa2acce` installed. "Pass" is
the probe's rule; "substance" is the reading of the calls and reply behind
each failing verdict.

| probe | before p1 (`1ef83f5`, no shared) | p1 (`447bb03`, no shared) | now |
|---|---|---|---|
| `aisvgs-sufficient` | 1/3 pass, 3/3 substance · 10–12 turns · $0.77 | 3/3 · 10–11 · $0.59 | 3/3 · 5–17 · $0.88 |
| `growbox-thing` | 2/3, 3/3 substance · 9–14 · $0.80 | 3/3 · 7–9 · $0.50 | 3/3 · 9–12 · $0.58 |
| `forge-protoprey` | 3/3 · 5–8 · $0.53 | 3/3 · 6–11 · $0.43 | 3/3 · 9–12 · $0.44 |
| `hold-release` | 3/3, claims clean · 3–4 · $0.37 | 3/3 · 3–7 · $0.40 | 3/3 · 3–5 · $0.36 |
| `delegate-answer` | **1/6** · 9–13 · $1.53 | **2/6** · 8–18 · $1.41 | **4/6** · 10–16 · $1.79 |
| `delegate-decision` | 5/6 · 3–20 · $2.56 | 5/6 · 11–23 · $2.42 | 4/6, 5/6 substance · 9–19 · $2.44 |

The four "substance" differences are spellings the rules do not hold:

- a closing "進捗があれば教えてください", which the rule forbids as
  "教えてください";
- a correct growbox state without the literal `pj-growbox`;
- "12 時間" written with a space.

The rules were not changed, so the counts stay comparable with ex1's.

## `aisvgs-sufficient` compared with run-0160

- run-0160 was a filesystem `find` refused by the harness, then no
  `agentchat` call. The before-p1 arm did not reproduce it: 0 of 3.
- That arm did reach for the filesystem in 10 of its 24 runs, once with
  `find / -iname …`. None was refused, and every one went on to read the
  board.
- p1's guides did so in 1 run of 24, and today's in 0. p1's "the board is
  Zulip, reached by `agentchat` — not the filesystem" shows in the calls.
- The before-p1 arm had today's `agentchat --help` index installed, which
  run-0160 did not.

## The delegation: cause and fix

**Three ways to fail.** A delegation run fails in one of three ways, and
the verdict does not say which:

- nothing sent;
- sent where no listener is served;
- proposed and waited.

Only the calls tell them apart. Classified that way:

| `delegate-answer` | reached | wrong door | none |
|---|---|---|---|
| before p1 | 1 | 2 | 3 |
| p1 | 2 | 2 | 2 |
| now | 4 | 1 | 1 |
| now, corrected board (ctl) | 4 | 0 | 2 |
| **fix**, corrected board | **6** | 0 | 0 |

**Cause.** No composition, old or new, covers "ask this named agent about
finished work". The guides sort requests into *work*, which is delegated
after a proposal, and *questions about what exists*, which are answered
from the board ("if the work is already done, say so" before p1). The
Developer's "autolab に聞いて" about a finished mission lands in the
second kind. Then two sentences read as reasons not to post:

- fd-wr: one run's own words, "解決済みトピックへの再投稿は新しい作業を始めてしまうため";
- as9: on running work, "if autolab asks me to choose, I will answer".

Every "none" run called the record complete, though it held only a start
and a done line.

**Fix** (report3 has the text):

- as9 now applies to asking after *running* work to see progress. A
  question the person you serve asks you to put to a named agent is the
  request, not a poll.
- fd-wr keeps "no second start" and adds that a question about finished
  work is a new conversation where the agent's introduction says questions
  go.
- `requests.md`: the board holds what was posted. What an agent did but
  never posted is theirs to tell, and when the Developer asks you to ask an
  agent, they want its answer.

**Before and after**, on the corrected board:

| probe | ctl | fix |
|---|---|---|
| `delegate-answer` | 4/6 | **6/6**, all six in `autolab-agstudio1`, each reporting the 41 minutes and the paywalled source |
| `delegate-decision` | 5/6 (one wrong door) | **6/6**, all in `workplan-growbox-control-loop`, 12 h sent, `b41d0e7` reported |

## The guards

| guard | ctl | fix |
|---|---|---|
| `guard-status` (as9: "m20402 の制御ループ、今どう？", nobody named) | 3/3, nothing sent | 3/3, nothing sent |
| `guard-finished` (fd-wr: "終わってたっけ？まだなら続きをやらせといて") | **2/3**: run 2 tried to post "Accepted." into the ✔ topic, bare name and ✔ name; the tool refused both | 3/3: two sent nothing, one asked autolab at its entrance about a contradiction it found (below), not a restart |

So today's guides do not fully hold fd-wr's lesson either; the tool's
refusal does. Three runs per arm are not a rate for that.

## The reader probe

`python -m agag.fixture reader`, pyagag `051bb67`:

- **Cases**: 13, as data. The three real replies became synthetic
  equivalents.
- **Reader**: the listener's own reader and `reply_words`.
- **Result**, `qwen3.8:27b-mxfp8`, 13 × 4: **44/52 right, 0 errors,
  0.29–3.68 s**, the same answer four times per case.
- **The misses**: the restatement and "another agent's act", as in p7. p7's
  third miss (a `send` read from the `ag-post` line) is gone, because the
  probe gives the reader what the listener gives it.

## The baseline's limit

`--guides-rev` restores the guide tree only. The before-p1 arm measured
**the old guides over the new tools**:

- today's `agentchat --help` index and subcommand helps;
- multi-topic `read`;
- the current reply and continuation sections;
- `send`'s refusal of a resolved topic.

An old-pyagag venv was not built. Every arm fails the delegation in the
same three ways, and the old text alone explains the order of the arms.
Older help texts could only add failures of their own to the before-p1
column.

## Open findings

1. **The fixture's finished missions contradict themselves.** m20390's
   done line says "accepted by Front" and its topic is ✔, but `agentchat
   trace` reads it as `queued`: the board builder writes no `[acceptance]`
   and no `work-m…` topic. One careful run (fix, `guard-finished`) believed
   the trace and asked autolab. Left unchanged here, because every probe's
   baseline would move. The fixture should record what it claims, and the
   delegation and guard probes should then be re-run.
2. **Rule spellings miss correct answers** in 4 of 108 runs (above). The
   rules were left as they are for comparability. Adding the spellings
   (and "教えてください" only when it asks what a study is) is a small change
   for the next phase that runs these probes.
3. **The Developer's round-3 decision** (p1 open finding 1, held by the
   Developer): carried in 3/3 before-p1 runs, 0/3 p1, 1/3 now. It is reported, not acted on.
4. **fd-wr attempts**: 2 of 6 step-1 `now` runs and 1 of 3 `ctl`
   `guard-finished` runs *tried* a post into a ✔ topic that the tool
   refused. The fix showed none in the 9 runs where it could have (6 `delegate-answer`, 3 `guard-finished`). Worth a
   rate before it is called fixed.
5. Unchanged from ex1: p2's open finding 1 (a reply claiming records it
   never made) now has p7's check. comfynotify, arxivsage and agecho keep
   older pyagag pins.

## Cost

| step | runs | cost |
|---|---|---|
| step 1 | 72 | $18.77 |
| step 3 | 36 | $11.96 |
| reader probe | 52 readings | $0 (local model) |
| **total** | | **$30.73** |

The plan estimated $15–20 for step 1. Step 3's control arm (18 runs,
$5.94) was not in the plan. It separates the fixture's correction from the
guide fix.

## Deus Ex Machina notes

- None. No post was made for an in-system agent. Every trial read the
  fixture or its overlay, and the relay's cost gauge was unchanged across
  step 1.
