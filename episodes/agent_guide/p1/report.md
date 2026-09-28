# agent_guide p1 — report

## Outcome

The conversational guides now tell an agent **what it can see and what is
assumed of it**, and hand it its tools by pointing at their `--help`.
Procedures under conditions are gone. Each fact lives in one place: a
tool's `--help`, one shared guide file, or the role's own guide. The guides
shrank as a result.

- All three board probes passed on the first reply after deployment: the
  incident wording, "the grow box thing", and "what did forge deliver for
  protoprey?".
- The largest moved failsafe paragraphs passed a live stopped-work mission.
- One regression, dispositions, was found in step 6 and fixed. It is
  described below.

Step reports: `report1.md` (inventory), `report2.md` (help texts),
`report3.md` (shared text), `report4.md` (guides), `report5.md` (sage
label), `report6.md` (trials and roll-out).

## What moved where

The per-paragraph table is in report1 (source and evidence) and report4
(destination). In short:

| from | to |
|---|---|
| tool usage in guides (agentchat subcommands, agrun, agproject documents, agrefs, budget reading) | the tool's own `--help`: `agentchat --help` became an index (236 → 117 lines); `send`, `read`, `recheck` (+UNOWNED), `receipt`, `hold`, `disposition`, `channels`, `resolve`, `accept`, `use`, `anchor`, `argue`, `intro`, `agproject open/status/plan`, `agrefs`, `agrun` and each subcommand, `agbudget` gained what the guides said |
| text shared by desk, front, routine_run and argue | `agfront/agent/guides/shared/{board,requests,work}.md`, appended by `zulip_listener.role_guide` per `SHARED_GUIDES` |
| desk and front bodies (diverged in both directions) | one body (`requests.md` + `work.md`); desk and front are 8- and 6-line heads |
| agrefs manual copied into 11 guides across 4 agents | a 5-line pointer to `agrefs --help`, and each guide's own follow-up |
| archsage's "no findings yet" | not printed; the introduction carries only what a definition decides |

## Sizes

Guide text only; the conversation, which varies, is not counted.

| role | before | after | with reply + continuation sections |
|---|---|---|---|
| desk | 21 798 chars / 367 lines | 14 077 chars / 247 lines (13 607 before the step-6 disposition fix) | 25 996 → 18 275 |
| front | 22 731 / 362 | 13 906 / 245 | 26 929 → 18 104 |
| routine_run | 16 647 / 284 | 15 949 / 282 | 20 845 → 20 147 |
| argue | 9 653 / 191 | 9 688 / 195 | 13 851 → 13 886 |

routine_run and argue did not shrink. They now receive the board, and
routine_run also receives the delegation facts, which they lacked; what
moved out of them into help texts is about the same size. What the desk
and front prompts lost in length, they gained in facts they did not have
before.

The desk prompt of run-0160 was 27 730 characters. A like-for-like
serving is about 7 700 characters shorter now.

## Trials

| trial | run | result |
|---|---|---|
| before: #15673 wording, old guide | run-0168 (8 turns, $0.34) | passed; the board now has archsage's `study-aisvgs-round3`, which it found. run-0160's failure did not reproduce |
| after: #15673 wording | run-0169 (11 turns, $0.18) | passed; first call `agentchat --help`, then `channels --prefix pj-`, `agproject status` |
| after: "the grow box thing" | run-0170 (25 turns, $0.41) | passed |
| after: forge's past work for protoprey | run-0171 (15 turns, $0.20) | passed |
| failsafe re-run: stopped task with a reserved acceptance (live mission m15750) | run-0172…0177 ($1.03) | passed: Observer at +174 s, recheck quoted, resume in the same topic at ~+200 s, close-out awaited, acceptance asked of the proxy and recorded on its words |
| disposition probe | run-0178 | **failed** (no record); fixed; run-0180 and run-0181 passed (#15825, #15830) |

Total Front spend on the trials: about $2.5 (runs 0168–0181).

## Paragraphs dropped as Anxiety-Driven

Restore from agfront `ba28e90^` if a failure returns.

| text | where | why dropped |
|---|---|---|
| "If the last message is small talk or something plain text answers, just reply." | desk, front | no trial behind it; the branch run-0160 took |
| "If nothing on the board can do it, say so kindly." | desk, front | no trial; covered by the proposal paragraph |
| "If the work is already done, say so." / "If you think task is already done, just reply so." | desk, front | no trial; covered by "answer from the board" |

Moved into help rather than dropped: "Everything you send to another agent
is ordinary, professional language" (no trial) is now in `send --help`.

## What the phase learned

- **A fact about recording needs the sentence that saying it is not
  recording it.** Folded into one line, the disposition paragraph lost its
  pull: Front said "I won't chase it" and wrote nothing. The acceptance
  paragraph kept "Acknowledging it in your reply records nothing" and kept
  working. Shortening must keep such sentences.
- **The before-trial of an incident can stop reproducing**: the board it
  ran against changes. A later phase that wants a controlled comparison
  needs a fixture board (a synthetic realm), not the live one.
- **Guides are live on disk; code and help texts are not.** Changes that
  span both are prepared on a branch and merged in one go, followed at
  once by the pin, `uv sync` and the listener restart (README_DEV *How a
  guide is put together*).
- archsage posts its introduction only through `archsage intro` and
  `_after_change`, not at start-up (report5 correction; preresearch §2.3
  said otherwise).

## Open findings

- A decision about a study made in a `front-desk-` conversation (the
  Developer doing round 3 personally) is not visible from the study's
  channel. run-0169 read `#pj-aisvgs` and offered what the Developer had
  declined. Where such a decision should be recorded is a question for a
  later phase.
- The Claude Code harness refuses shell loops (`simple_expansion`) in
  Front's runs. That cost turns in two probes. A multi-topic read in
  `agentchat` (or the fact written into `board.md`) would save them.
- A mention in an agent's acknowledgement re-serves Front with the
  previous trigger (the double reply to #15771). This is known behaviour,
  cost one serving.

## Documents updated

- `devdocs/README_DEV.md`: *How a guide is put together (`agent_guide` p1)*.
- `devpolicy/styles.md`: *Agent guides*, one paragraph.

## Deus Ex Machina notes

- Omni recorded nothing for an in-system agent: every record in the trials
  was written by Front or autolab. Omni posted as the Developer's proxy
  (requests, the acceptance, the trial-over statements) and ran
  `archsage intro` to re-post archsage's introduction. That was the
  operator's step for a code change, not archsage's work — no handoff.
