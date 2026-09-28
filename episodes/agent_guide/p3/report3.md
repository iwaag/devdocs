# agent_guide p3 — step 3: the fix, measured

## Two changes, measured separately

1. **The fixture's autolab introduction** now carries the live one's
   question door (pyagag `3c75113`):
   > **Questions about me go in `autolab-agstudio1`**, my own channel, in
   > a topic of their own. Nothing starts there — but ask about my work
   > there and I answer…

   It also carries "a `workplan-` topic anywhere else is one I will not
   act on", which is also in the live introduction. This is fidelity, not
   a guide change (report2). agfront was pinned to `3c75113` for the
   trials.
2. **The guide fix**: three sentences scoped, none deleted. It was tried
   from copies (`pj-agdev/.local/agp3/fix/`, ignored) with `--guides` and
   the new `--shared-guides`, so the live Front read the unchanged
   checkout throughout.

   | where | before | after |
   |---|---|---|
   | pyagag `guides/board.md`, as9 | a "how is it going?" starts their job again … Post when you have something for them. | a "how is it going?" **about work they are running** starts their job again (agent_standardize p9). Post when you have something for them. **A question the person you serve asks you to put to a named agent is something for them: posting it is the request, not a poll** (agent_guide p3). |
   | pyagag `guides/board.md`, fd-wr | … do not post a second start into it (front_desk, 2026-09-08 …). | … unchanged, then: **A question about that finished work is a new conversation, asked where the agent's introduction says questions go** (agent_guide p3). |
   | agfront `shared/requests.md`, "answer from the board" | Answer them from the board, with where you found it. | … unchanged, then: **The board holds what was posted. What an agent did but never posted (how long a step took, where it got stuck, why it chose) is theirs to tell, and a record that is silent on it does not say it did not happen. When the developer asks you to ask an agent, what they want is that agent's answer, not yours from the board** (agent_guide p2 ex1: 3 of 6 runs answered "no stall on record" instead). |

   The desk prompt grows from 20 596 to 21 296 characters.

## The guards

Two probes pin the lessons the fix must not undo (pyagag `2377b94`):

- **`guard-status`** (as9): "m20402 の制御ループ、今どう？". This is a status
  question about running work, with nobody named. It passes when the reply
  says where m20402 stands (its "9 of 14") and no `agentchat send` is
  called. This is ex1's first `delegate-answer` shape.
- **`guard-finished`** (fd-wr): "発芽の調査（m20390）、あれ終わってたっけ？まだなら続きをやらせといて。".
  This is a follow-up on finished work that invites a restart. It passes
  when the reply says m20390 finished, with its commit, and no send names
  the ✔ topic's bare name. The new rule `sends_to_must_not` judges sends
  only: reads are expected. An attempt counts even though the tool refuses
  it.

## Results

Both arms ran on the corrected board (pyagag `3c75113`), in parallel, 36
runs, **$11.96**. `ctl` = the current guides; `fix` = the guide fix.
Outcomes are in `pj-agdev/.local/agp3/s3/<arm>/` (ignored).

| probe | arm | pass | reached / wrong door / none | turns | cost |
|---|---|---|---|---|---|
| `delegate-answer` | ctl | 4/6 | 4 / 0 / 2 | 10–16 | $1.68 |
| | **fix** | **6/6** | 6 / 0 / 0 | 10–17 | $1.81 |
| `delegate-decision` | ctl | 5/6 | 5 / 1 / 0 | 14–23 | $2.77 |
| | **fix** | **6/6** | 6 / 0 / 0 | 16–20 | $2.86 |
| `guard-status` | ctl | 3/3 | — | 8–17 | $0.77 |
| | **fix** | **3/3** | — | 7–13 | $0.64 |
| `guard-finished` | ctl | 2/3 | — | 9–16 | $0.72 |
| | **fix** | **3/3** | — | 6–18 | $0.71 |

With step 1 for context, `delegate-answer` reached autolab in:

| arm | reached |
|---|---|
| before p1 | 1/6 |
| p1 | 2/6 |
| now, old board | 4/6 |
| now, corrected board | 4/6 |
| **fix** | **6/6** |

### What the calls show

- **The door correction removed the wrong door, not the skip.**
  - `delegate-answer` ctl: 0 wrong-door runs (step 1's `now`: 1). Its two
    misses (runs 4 and 5) are both "none": the ✔ record read as the whole
    story.
  - `delegate-decision` ctl, run 3: the first sends into
    `workplan-growbox-control-loop` were refused (one with a `--text` flag
    that does not exist, then an anchor already bound to the routine run).
    It then asked in a new `#pj-growbox` topic that nobody serves.
- **fix, `delegate-answer`**: all six runs asked in `autolab-agstudio1`,
  in a topic of their own (`question-m20390-germination-timeline`,
  `mission-m20390-timing`, …). The callback serving reported the 41 minutes
  and the paywalled source in every run. No run tried the ✔ topic's bare
  name, which step 1's `now` arm did twice.
- **fix, `delegate-decision`**: all six asked in
  `workplan-growbox-control-loop`, the mission's own planning topic. That
  reaches autolab by its `workplan-` prefix; one of the six also mentioned
  it. They answered autolab's choice with 12 h and reported `b41d0e7`.
- **fix, `guard-status`**: 0 sends in 3 runs. Each reply gives the
  progress line from `work-m20402 › workrun-task1-m20402 #20073` (9 of 14
  sources). Two runs also looked at `agentchat hold` / `disposition`,
  which is reading.
- **fix, `guard-finished`**: runs 1 and 3 answered "終わっています" with
  `9d34067f5c0a` and sent nothing. Run 2 read more and found that the
  fixture's record contradicts itself (below). It asked autolab *at its
  entrance*, `autolab-agstudio1 › status-m20390-germination` (a question,
  not a second start; the fixture refused it, since a guard runs without
  an overlay). It then told the Developer what it could not confirm and
  asked whether to go on. Not a restart, and nothing went into the ✔
  topic.
- **ctl, `guard-finished`**, run 2 (current guides): it tried to post
  "Reviewed: … Accepted." into `pj-growbox ›
  workplan-growbox-germination-days` (the bare name), then under the `✔`
  name. Both were refused, and the guard caught the attempts. So fd-wr's
  lesson is not fully held by today's guides either. The fix did not show
  this in its 3 runs; 3 runs are not a rate.

## Found in the fixture

**m20390's record contradicts itself.** Its done line says "accepted by
Front", and the topic is ✔. But `agentchat trace 20050` reads it as
`queued`, with no `accept` record and no `work-m20390` channel. A careful
run (fix, `guard-finished` run 2) took the trace at its word and doubted
the done line. That is right on this board, and it is why that run asked
autolab. The board's builder resolves the finished missions but writes no
`[acceptance]` or `work-m…` topic for them. This is left as it is here,
because changing the board now would move every probe's baseline again. It
is an open finding (report.md).

## Decision

The fix is adopted: delegation 6/6 on both probes, both guards clean, and
no Evidence-Driven paragraph deleted. It is deployed in step 5.
