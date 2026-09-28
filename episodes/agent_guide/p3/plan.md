# agent_guide p3 — measure what p1–p2 changed, and the delegation Front skips

## Goal and scope

p1 and p2 rewrote the guides, and p2 ex1 made trials repeatable. What has
not been measured is the behaviour against the guides *before* p1: every
fixture comparison so far used p1's guides as the baseline, and both
passed. And ex1 found a behaviour that may be a side effect of the
rewrite: asked to "ask autolab", Front answered from the board instead in
3 of 6 runs. This phase measures both against the pre-p1 guides, fixes the
delegation behaviour if the guides cause it, and versions the one trial
tool p7 left outside git.

Read `../p2/ex1/report.md` and `../p2/ex1/report5.md` (the runs), then
README_DEV § *Trying a guide change against the same board*.

Private experimental, breaking-change phase: no backward compatibility.
The implementer chooses sample sizes, probe wording and any guide change.
Fixed rules, carried from p1–p2: no Evidence-Driven paragraph is deleted
(it may move or be scoped), and length is not a target.

Out of scope, still held by the Developer: where a decision about a study
is recorded (p1 open finding 1), and the agautolab1 VM.

## step1 — the three-arm comparison

Run each probe under three compositions, from agfront's main checkout:

| arm | flags | desk prompt size (ex1) |
|---|---|---|
| before p1 | `--guides-rev 'ba28e90^' --no-shared` (= `1ef83f5`) | 27 255 |
| p1 | `--guides-rev 447bb03 --no-shared` | 19 534 |
| now | none | ~20 600 |

```
cd pj-agdev/agfront && .venv/bin/python -m agfront.trial <probe> --out <dir> [flags]
```

Probes: `aisvgs-sufficient`, `growbox-thing`, `forge-protoprey`,
`delegate-answer`, `delegate-decision`, `hold-release`. At least 6 runs
per probe and arm for the delegation probes, where ex1 saw 3 of 6; 3 are
enough for probes that have never failed. About $0.15–$0.45 per run, so
the whole step is roughly $15–20.

Hints:

- **The baseline is not the whole old system.** `--guides-rev` restores
  the guide tree only. `agentchat --help` and every subcommand help are the
  installed pyagag's (the 117-line index, not the 237-line manual), and
  `read` takes several topics. The reply and continuation sections are
  current too. So the before-p1 arm measures the old *guides* against the
  new tools. Say so in the report. If the difference matters, a pyagag
  worktree at `73efd5c^` installed into a scratch venv gives the old help
  texts too; the implementer decides whether it is worth it.
- run-0160's own failure was a filesystem `find` refused by the harness,
  then no `agentchat` call. The fixture runs under the same harness, so
  the before-p1 arm can reproduce it; the p1 report could not, because the
  live board had changed.
- Read the tool calls behind every verdict, not only the verdict (ex1:
  both responder defects looked like passes).

## step2 — why Front skips the delegation

- In the failing `delegate-answer` runs Front read `✔ workplan-growbox-
  germination-days`, answered "about 45 minutes, no stall on record", and
  did not post. The Developer's words were "autolab に聞いて教えて". The
  board holds only the start and done lines; the 41 minutes and the
  paywalled source are autolab's own experience.
- Suspected sentences (both Evidence-Driven, in pyagag
  `src/agag/guides/board.md`): as9, "a 'how is it going?' starts their job
  again … Post when you have something for them"; fd-wr, "a topic whose
  name begins with ✔ is finished … do not post a second start into it".
  ex1 read them as reasons not to post at all.
- step1's before-p1 arm says whether this is new. If the old guide
  delegates reliably and the new one does not, the rewrite caused it; if
  both skip, it is older and the guide's "answer from the board" framing
  (p1's replacement for the "just reply" branch, `requests.md`) is another
  suspect.
- A dry-run prompt (`--dry-run`) of each arm shows the exact text Front
  had; diffing the three is the cheapest first look.

## step3 — fix it, if the guides cause it

- Scope, do not delete. Candidates the implementer may use or replace:
  - as9 is about asking after *running* work to see progress; a question
    the Developer asks you to put to a named agent is theirs, and posting
    it is the request, not a poll.
  - fd-wr is about a second *start* in a resolved task topic; asking the
    agent in its entrance (its introduction says where) is not a start.
  - "Answer from the board" holds for facts the board records; what an
    agent experienced but did not post is theirs to tell.
- Guard the original lessons with probes, so the fix does not undo them:
  - as9: a status question about running work, with no request to ask
    anyone ("m20402 の制御ループ、今どう？") → no post, answered from
    the board. The first version of `delegate-answer` (ex1, probes.py note)
    is this shape.
  - fd-wr: a follow-up on finished work that invites a restart → no post
    into the ✔ topic.
- Re-run `delegate-answer` and `delegate-decision` (6+ each) and the two
  guards (3+ each) with the fix. Pass: delegation in most runs, guards
  clean. Report the counts, not a single verdict.

## step4 — version p7's reader measurement

- failsafe p7 measured its claims reader (13 cases × 4 runs, 0.3–3.9 s per
  reply) with `pj-agdev/.local/failsafe-p7/reader_probe.py` and three reply
  samples (`r15838.txt`, `r15844.txt`, `r15851.txt`), all ignored. If the
  host's model changes (`~/.config/agag/claims.toml`), that measurement
  cannot be repeated.
- Move the script into pyagag beside `agag.claims` (a `python -m
  agag.claims probe` or a module under `agag.fixture`), with the cases as
  data in the repo. The three samples are real replies; write synthetic
  equivalents rather than committing them.
- Run it once against the current reader and record the result.

## step5 — report

- `report.md`: the three-arm table per probe (runs, pass counts, turns,
  cost), what the before-p1 arm did on `aisvgs-sufficient` compared with
  run-0160, the delegation cause and fix with before/after counts, the
  guards, the reader probe's result, and the stated limit of the baseline
  (old guides over new tools).
- If a guide changed: deploy as README_DEV § *How a guide is put together*
  describes (pyagag change: pin, `uv sync`, restart listeners one at a
  time, check logs; service state through Nautobot or `nctl status`), run
  the consumer suites and read the summary line.
