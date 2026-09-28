# agent_guide p1 — step 6: trials and roll-out

All trial posts were made by the Omni Agent (user 9, the Developer's full
proxy) in `#front`, with the Developer approving each post from this shell
by hand. The session's auto-mode classifier had refused the first attempt
(report3).

## Before the change: the incident wording against the old guide

`#front › front-desk-20260928-agp1-before`, #15718: the #15673 wording, in
Japanese, word for word.

- **run-0168** (old guide, 8 turns, $0.34): **passed**. The first reply
  (#15720) named `#pj-aisvgs › researchplan-aisvgs-round3` (#15691),
  archsage's round-3 verdict and the Developer's 09:20 decision.
- What it did: `ls`/`find` in its workspace (allowed this time), read
  `tools/runs.md` and `tools/agents.md`, then `agentchat topics
  archsage-agstudio1` → `study-aisvgs-round3` → `topics pj-aisvgs`. The
  archsage topic `study-aisvgs-round3` did not exist when run-0160 ran
  (it was opened at #15686, after the incident). The board it found the
  answer on is not the board run-0160 faced.

So run-0160's failure did **not** reproduce on today's board. The old guide
can succeed when the board happens to lead it there. The after-trial
therefore shows how the new guide gets there, not that it fixes a failure
that reproduces on demand.

## Deployment

| time (Z) | what |
|---|---|
| ~10:56 | `agent-guide-p1` fast-forwarded into `main`: agfront `a203731`, agautolab `2efc711`, agforge `73c9c8a`, archsage `b35a03e` |
| 10:56 | agfront: pyagag `73efd5c` locked and synced; 195 tests passed; `com.agdev.agfront-zulip` kickstarted at 10:56:34 (the merged guides were live on disk before that, while the old listener read only `guide.md`: a window of about a minute with no serving in it) |
| ~11:00 | pyagag `73efd5c` locked and synced in agautolab (331 passed), agforge (265), agobserver (177), archsage (40), cagent (204), agentroom relay (363) |
| 11:05 | `com.agdev.archsage-zulip` kickstarted. The introduction was **not** re-posted at start-up (report5's correction); `archsage intro` posted it, and the board now reads `*(study `aisvgs`)*` |
| | locks committed: agfront `b811e4f`, agautolab `aab6810`, agforge `0558c6e`, agdevworld `1108fd8`, archsage `df6e0af`, pj-agdev `49c8d6b` (agobserver lock + submodule pointers), pj-clusterintent `1bce119` (cagent lock) |

The only listener restarts were agfront's (new code: `role_guide`) and
archsage's. The pyagag change is help text only, which a run reads each
time it calls the CLI. The other listeners' in-process code did not
change.

## After the change

| probe | topic | run | first reply | pass |
|---|---|---|---|---|
| #15673 wording | `…-agp1-after` #15724 | run-0169: 11 turns, $0.18, 40 s | #15726 names `#pj-aisvgs › researchplan-aisvgs-round3` (#15691), the round-1/2 workplans ✔, and the round-3 plan not yet started | ✓ |
| "あのgrow boxのやつ、いまどこまで進んでる？" | `…-agp1-growbox` #15728 | run-0170: 25 turns, $0.41, 89 s | #15732: `#pj-growbox`, study ready at `9d34067f5c0a`, seven missions over four strands, each strand's reports and gaps | ✓ |
| "forgeってprotoprey向けに何を納品したんだっけ？" | `…-agp1-forge` #15736 | run-0171: 15 turns, $0.20, 39 s | #15738: six forge conversations for ProtoPrey in `agforge-agstudio1`, each with its delivery; the sound request undelivered and why | ✓ |

How the after-runs found the board:

- run-0169's **first tool call was `agentchat --help`**, then `channels
  --prefix pj-`, then `agproject status aisvgs`. No filesystem probe.
- run-0170: `channels --prefix pj-` → `agproject status growbox` →
  topics/read. It later also ran `agrun --help`, then `ls tools/`.
- run-0171: `agentchat intro forge` failed with the list of names, then
  `intro agforge-agstudio1`, `agproject status protoprey`, `topics`.

Frictions seen, none of them guide failures:

- Shell loops over topics were refused by the harness ("Contains
  simple_expansion") twice in run-0170 and twice in run-0171; each run
  fell back to single reads.
- run-0170 tried `agentchat topics --prefix`, which does not exist. The
  error printed the usage, and it went on.
- **A content difference**: run-0168 (old guide), reading archsage's topic,
  reported the Developer's 09:20 decision that round 3 is theirs to do by
  hand. run-0169 read the study channel instead and offered to ask autolab
  for round 3. That decision lives in a `front-desk-` conversation and in
  archsage's topic, not in `#pj-aisvgs`. Neither guide says where a
  decision about a study is recorded. Left as a finding, not fixed here.

## Failsafe paragraphs that moved: live re-run

One live mission covered the biggest moved blocks: `work.md`'s Observer
section and acceptance holders, `recheck --help` and `reserve`.

`#front › front-desk-20260928-agp1-live` (#15740): "add
`docs/agp1-trial.txt` with one line in pj-robustp1, see it through, no
confirmation needed, but I accept the mission myself". autolab's
`faults/silent-exit` was armed before the post.

| time (Z) | event |
|---|---|
| 11:20:42 | request #15740 |
| 11:21:12 | Front: `reserve` recorded on #15740, `workplan-agp1-trial-docs` opened (#15744), reported (#15748) |
| 11:21:24 | autolab plans m15750; task 1's first serving is killed at its first tool call (exit -9) |
| 11:24:18 | Observer: **[Observer] Work this request depends on has stopped** (#15771): **174 s** after the exit |
| 11:24:24 | Front: `agentchat recheck 15753 --after 15762` → STOPPED, quoted |
| ~11:24:40 | Front resumes **in the same `workrun-task1-m15750`** (#15775): check what #15763 left, continue rather than redo, show the result. **About 200 s** after the exit. Nothing else was opened |
| 11:25:34 | autolab's result #15780. Front re-checks: RESUMED. It agrees in the task topic (#15791) and says it waits for the close-out record (#15800), not that the task is closed |
| 11:26:10 | autolab: accepted, `main df661d3` fast-forwarded and published, task completed, ✔ |
| 11:26:32 | Front **asks** the Omni Agent for the mission's acceptance (#15804, response_request/confirmation) and records nothing of its own |
| 11:26:46 | Omni Agent accepts (#15807) |
| 11:27:00 | Front: `agentchat accept 15750 --evidence 15807` → "m15750 is done: accepted by Omni Agent (#15807)"; workplan ✔ (#15813) |

`agentchat trace 15740` afterwards: the request ANSWERED, mission and task
DONE/accepted. Every behaviour the moved paragraphs bought (fs2, fs3A,
fs3A2, fs4, fs5 reserve) was still there. The costs were six desk servings
(run-0172…0177), $1.03 in total. Twice Front answered the same Observer
post (#15777 and #15781): autolab's acknowledgement named Front, and that
mention served the conversation again with the stop report still its
trigger. That is the known "a mention re-serves" behaviour (MEMORY: bot
posts re-serve), not a guide effect.

### A regression found and fixed: dispositions

Probe: "この会話はガイド改修のトライアルでした。ここで終わりにして、以後この依頼は追いかけなくていいです。"

- #15817 in `…-agp1-before` (run-0178): **failed**. The reply (#15819)
  was "承知しました … 以後この依頼は追いかけません", with **no tool call** and 0
  dispositions. In step 4 the disposition and hold paragraphs had been
  folded into one sentence at the end of "Whose decision it is". The old
  guide had them as their own sections, with the person's phrases, and p6
  ex1's live trials used them without further prompting.
- Fix (agfront `447bb03`, pj-agdev `56c6fc3`; `work.md`): holds and dispositions are their own
  paragraph again, with the phrases, and with the fact that settles it:
  "Saying in your reply that you will stop, or that it is over, records
  nothing. Until the record exists, Observer and the progress panel treat
  the request as open."
- #15821 in `…-agp1-growbox` (run-0180, 6 turns): **passed**, `withdrawn`
  #15825 on #15821. #15828 in `…-agp1-before` (run-0181): passed,
  `withdrawn` #15830.

The lesson for later shortening: a fact about *recording* something needs
the sentence that saying it is not recording it. The acceptance paragraph
always had that ("Acknowledging it in your reply records nothing"), and
the one-line disposition did not.

### Not re-run

- **Receipt repair** (p6 `exit-before-receipt`): the automatic path is
  listener code, and nothing in it changed. The guide-facing part (`receipt
  --help` when Observer asks about an owed answer) had no natural trigger
  in these trials.
- **Holds**: the paragraph is now its own bullet next to dispositions,
  covered by the same fix, but no live hold was placed in this phase.

## Tests

- pyagag: 1125 passed (step 2).
- agfront: 195 passed (main checkout, after the merge and pin).
- Consumers after the pin: agautolab 331, agforge 265, agobserver 177,
  archsage 40, cagent 204, agentroom 363. All passed.

## Left as they are

- Trial conversations (open as the record): `front-desk-20260928-agp1-before`
  (withdrawn #15830), `-after`, `-growbox` (withdrawn #15825), `-forge`,
  `-live` (m15750 done). `work-m15750` is not archived.
- The `agent-guide-p1` branches remain on the remotes (merged); the local
  branches and worktrees are removed.
- No fault files are armed (`silent-exit` was consumed).
