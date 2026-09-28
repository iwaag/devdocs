# agent_guide p3 ex1 — a board that records what it claims, and rates instead of hints

## Goal and scope

p3 showed the delegation fix working (6/6 against 4/6) and the guards
holding, but on 3–6 runs per arm, on a fixture whose finished missions
contradict themselves, judged by rules that miss 4 of 108 correct answers.
Make the fixture consistent, make the rules and re-judging cheap, then
measure the delegation fix and the ✔-topic lesson (fd-wr) with enough runs
to call them rates.

Read `../report.md`, `../report1.md` (the three-arm runs and the four rule
misses) and `../report3.md` (fix, guards, the `queued` contradiction).

Private experimental, breaking-change phase: no backward compatibility.
The implementer chooses sample sizes above the minimums below, the
statistics, and the fixture's internals. Fixed rules carried from p1–p3:
no Evidence-Driven paragraph is deleted, and length is not a target. No
guide change in this ex unless a measured rate calls for one; if it does,
report it and stop there, rather than fixing inside this ex.

Out of scope, still held by the Developer: where a decision about a study
is recorded (p1 open finding 1), and the agautolab1 VM.

## step1 — finished missions record what they say

- The contradiction: m20390's done line says "accepted by Front" and its
  topic is ✔, but `agentchat trace 20050` reads it as `queued`, with no
  acceptance record and no `work-m20390` channel. One run (fix,
  `guard-finished` run 2) believed the trace and asked autolab.
- In `pyagag/src/agag/fixture/board.py`, the routine-run missions (around
  line 313–323: `germination-days` m20390 and its siblings) get a done line
  only. m20301 (line ~224–234) is fuller: a `work-m20301` channel, a
  `workrun-task1-` topic, Front's agreement, autolab's close, ✔. Neither
  writes the selfnotes a live finished mission carries.
- What a live finished mission holds (README_DEV, autolab section and
  § *Work relations and dispositions*): `[state] accepted` on each finished
  task, `[selfnote][acceptance] #<post> by <user> … after=#<shown>`,
  `[state] done`, and ✔. The cleanest template is a real one: `agentchat
  trace` on m15750 (p1's live mission) or m11579 against the live mirror
  shows the shape the readers expect. Copy the shape, not the content
  (realm-export fixtures are refused; build rows in code).
- Add a consistency test over the whole fixture: every mission whose text
  says done or accepted traces as done; every ✔ task topic's mission is
  not `queued`; every agent the board names has an introduction. The
  contradiction p3 found would have failed it.
- Stamp the board with a version (a constant in `agag.fixture.board`) and
  write it into `outcome.json`, so a result says which board it ran on.

## step2 — rules that hold correct spellings, and re-judging for free

- The four misses (p3 report § *The three-arm table*): a closing
  "進捗があれば教えてください" forbidden as "教えてください"; a correct
  growbox state without the literal `pj-growbox`; "12 時間" with a space.
  Fix the rules: accept the spellings, and forbid "教えてください" only
  where it asks the Developer what a study is.
- Add `python -m agag.fixture rejudge <out-dir>…`: re-apply the current
  rules to saved `outcome.json` files (reply, tool calls and sends are all
  there), without running a model.
- Re-judge p3's 108 runs and ex1's with it. Report the old and new counts
  side by side. Every change must be a spelling, not a changed standard;
  if a re-judged pass looks wrong when you read it, the rule is too loose.

## step3 — rates on the corrected board

Current guides (pyagag `1f49c28`, agfront `7daa945` or later) and the
pre-fix control on the new board:

| arm | how |
|---|---|
| fix | defaults |
| ctl | `--guides-rev 7daa945^` and `--shared-guides <dir>` holding pyagag's `src/agag/guides/` at `1f49c28^` (the option is from pyagag `2377b94`) |

| probe | runs per arm (minimum) |
|---|---|
| `delegate-answer` | 10 |
| `delegate-decision` | 10 |
| `guard-status` | 10 |
| `guard-finished` | 10 |
| `aisvgs-sufficient`, `growbox-thing`, `forge-protoprey`, `hold-release` | 3, fix arm only (a new baseline on the new board) |

About $0.30 per run, so roughly $25–30. Run the arms interleaved, not one
after the other, so a model-side drift hits both.

- **fd-wr rate from every run, not only the guard.** An attempted post
  into a ✔ topic (bare name or ✔ name) shows in the tool calls even when
  `send` refuses it. Count attempts over all runs of both arms.
- **Classify every delegation run** by its calls, as p3 did: reached,
  wrong door, none, proposed-and-waited.
- **Report rates with an interval** (a Wilson interval per cell is enough),
  and say plainly where the arms' intervals overlap.
- Read the calls behind a sample of passing verdicts too; ex1's responder
  defects and p3's rule misses were both invisible in the verdicts.
- The other agents' probes (`archsage-loose-sage`, `planner-other-project`,
  `entrance-plans`, `triage-unopened`) run once each on the new board, so
  their baselines move with it.

## step4 — report

- `report.md`: what the fixture now records and the consistency test; the
  rule changes and the re-judged counts for p3 and ex1; the rate table per
  probe and arm with intervals; the fd-wr attempt rate; the delegation
  classification; cost.
- If a rate says a guide still fails a lesson (fd-wr attempts in the fix
  arm, delegation below the control, a guard broken), state it as an open
  finding with the runs; the guide change belongs to the next phase.
- Update README_DEV § *Trying a guide change against the same board* for
  the board version, `rejudge`, and the consistency test.
