# Step 3 report — the study exists, the hand-over held, research is running

Plan: [plan.md](plan.md) step 3. 2026-09-20 14:33–15:10 UTC.

## What was created, and by whom

- **`#pj-protoprey-research`** (study) with its channel folder, opened by Front from
  the argue with `agproject open` during step 2; `researchplan-protoprey-research`
  (#7536) is the goal document; `workplan-setup-protoprey-research` (#7538) is the
  setup request autolab answered (#7542): workspace `protoprey-research`, `main/`
  pushed as `d263687` on the internal git route with `RESEARCHPLAN.md`, `methods/`,
  `reports/`, `README.md`; `README_PROJECT.md` naming channel, topic and argue.
- Not created: a `game`-pattern project. The human agreed to study first (#7552);
  the project (`GOAL.md`, `main/` `direction/` `devlog/`) is to be opened from the
  study's first results, as Front's close-out says (#7592, next work 3). The
  relation study → game is recorded in the argue and in `README_PROJECT.md`.

## The hand-over

Nobody had asked for the research after the argue closed: Front's close-out listed
the first round as "next work" without a requester. The human's decision existed
(#7552: MVP as proposed, study first, no time or cost limit), so the Omni Agent
relayed it to Front in `#front › front-protoprey-handover-20260920T1500Z` (#7596),
naming the decision by message id and asking for the ordinary route, not for a new
approval. **Omni Agent stand-in: the requester's role** — the plan wants Front to
hand over what the human decided to start; the human's decision was in the argue but
Front does not act on a closed argue by itself.

Front then did the rest on its own:

1. Opened `#pj-protoprey-research › workplan-first-research-round` (#7599) with the
   authorization cited, the two questions, the study's conventions and a report back.
2. autolab planned mission **m7601** "ProtoPrey research round 1" (#7621): four
   tasks in order — Q1a gore-free depiction techniques; Q1b CERO/IARC/Steam/console
   rules; Q2 the smallest-experiment method; synthesis with 1–3 experiment proposals
   — wrote `start.flag` because the authorization was already on record, and opened
   `work-m7601` with a `workrun-task<N>-m7601` topic per task.
3. Front started task 1 (#7623). autolab delivered
   `main/reports/q1a-gore-free-depiction.md` (commit `d013593`, 208 lines: eleven
   techniques with named works and sources, counter-cases, a prototype-fit section
   marked as inference, sources, open issues) in about six minutes.
4. Front read the delivery, **accepted the report's stated gaps itself** (#7636: no
   creator statement on being eaten, no verified fatal non-graphic swallowing, quotes
   through a summarising fetch tool) and asked that they be carried into the open
   issues; autolab resolved the task (#7645) and Front started task 2 (#7648) without
   asking anyone. The chain workplan → task → close → next task runs without the
   Omni Agent.

The `sage:protoprey` tree was attached to the study's `main/` repository and synced
(`archsage sage sync protoprey` → `d263687`, then `d013593`): the council can now
read what the study writes. **Omni Agent stand-in:** the owner's attach step, as
archsage's design has it; the sage's config carries an internal URL and stays
untracked in the archsage checkout.

## What the human can see

- Project Room `/?view=project`: `pj-protoprey-research`, kind study, mission 7601
  with tasks 1 completed / 2–4 open, read from the realm (checked on the relay's
  `/projects/pj-protoprey-research`).
- Zulip: the argue → `researchplan-` → `workplan-first-research-round` → `work-m7601`
  topics, each linking the previous by id.
- Posting from the Project Room is still unexercised: the one conversation worth
  commenting in is the live mission, and a test comment there would be read by
  autolab as an instruction. A real remark from the human in the plan topic is the
  honest test; it is left to the human.

## Cost so far in this step

| Role | Runs | Cost |
|---|---:|---:|
| Front (hand-over, mission driving) | 4 | $0.41 |
| autolab planner | 1 | $0.16 |
| autolab task 1 (two runs: research, then report + push) | 2 | $3.57 |
| **Total** | 7 | **$4.14** |

Task 1's research run was $3.45 for 116 s: web fetches of ~40 sources.

## Findings

- **A closed argue hands nothing over by itself.** Front's outcome names the next
  work but no one is served to do it; a request has to be made in a conversation. The
  relay in `#front` cost one Front run; the plan's "Front hands what the human decided
  to the workplan route" therefore needs either the human's own follow-up post or the
  Omni Agent's relay. Candidate for the argue guide: when the human has authorized a
  start, open the `workplan-` from the argue's close-out.
- **Acceptance attribution.** Front accepted the report's gaps on its own reading and
  said so; autolab's task note wrote "developer-accepted". The plan separates the
  agent's technical check from the human's judgement; here the agent's check was
  recorded as the human's. Small, but worth a word in autolab's report contract.
- autolab's progress posts arrive as many short tool-trace lines in the task topic
  (`🔧 WebFetch …`); `agentchat read` shows some of them as empty bodies. Readable in
  Zulip, noisy for a room.
- The mission created its work channel a moment after autolab first tried to list it
  (`could not list 'work-m7601'` once in the log) — harmless, self-healed.

## Reached

Preparation did not stop at setup: the agreed research is running under the ordinary
mission route, Front drives the task sequence itself, and the human can follow it in
the Project Room and in Zulip. Tasks 2–4 continue in step 4.
