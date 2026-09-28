# agent_guide p1 — step 4: the guides rewritten

agfront `a203731` on `agent-guide-p1`. It is not deployed yet: see report3
for why the work is on a branch.

## Shape

Each conversational role gets its own guide, followed by the shared files
from step 3:

| role | own guide | + shared | head says |
|---|---|---|---|
| desk | 8 lines | board, requests, work | you are Front at the Front Desk; you speak with the developer; their language, exact names; this conversation is the record; no role-play |
| front | 6 lines | board, requests, work | you are Front, the developer's entrance; you speak with the developer; exact names |
| routine_run | 121 lines | board, work | you drive one run; this topic is your record; the opening post wins over the guide |
| argue | 165 lines | board | unchanged head (facilitating an argue) |

What the plan asked the head to say, and where it is now:

1. **Who you are and who you speak with**: each role's own first
   paragraph.
2. **What is assumed of you**: `board.md`, first paragraph. "The developer
   speaks to you as someone who knows every project, study, routine, sage
   and agent on the board. A name that is new to you is yours to look up,
   not the developer's to explain."
3. **What you can see and how**: `board.md`. The board is Zulip, reached
   by `agentchat`, not the filesystem (run-0160's refused `find`);
   `agentchat --help` / `<command> --help`; reading is free, a post runs
   its addressee; one line each for `intro`, `channels --prefix
   pj-|routine-`, `topics`, `agrefs`. `agproject status` and `agrun` are in
   `requests.md`, because routine_run has no `agproject` grant and argue
   has no `agrun`. `agbudget` is in routine_run's own guide, the only role
   granted it.
4. **Facts no help can hold**: proxy authority, acceptance holders, holds
   and dispositions in one line each (`work.md` § Whose decision it is);
   no `@**mentions**` in `#front` (`requests.md`); this conversation is the
   record (desk head, routine_run head).

The body is written as facts, not branches. "What to do in one run" with
its five `If …` branches is gone. What each branch carried is now a fact:
proposing before delegating (prop), reporting where you posted, and "most
questions are about something that already exists: answer them from the
board".

## Where every desk paragraph went

"help" means that command's `--help` (step 2).

| rows | now |
|---|---|
| D1, D2 | desk head |
| D3 | `send --help` (one sentence) |
| D4 | `work.md` § Whose decision it is |
| D5 | `requests.md` |
| D6 | `board.md` (working directory) |
| D7 | **dropped** (the "just reply" branch) |
| D8, D9 | `requests.md` (Work) |
| D10, D11 | **dropped** |
| D12 | `requests.md` (argue: the fact) + `argue --help` (usage, stem, do not post again) |
| D13 | `requests.md` (project: only on a stated decision) + `agproject open --help` (GOAL.md contents, first work as `workplan-`) |
| D14 | `requests.md` (study: archsage's, per its introduction; setup ≠ research; research run, then sage refresh) |
| D15 | `requests.md` (never post into `guide`), `agrun --help` (what a routine is), `board.md` (`--prefix routine-`) |
| D16 | `agrun --help` (how a run is opened, the opening post, no schedule) |
| D17 | `requests.md` (the whole reply, with rt2's mechanism, now in both desk and front) |
| D18 | `work.md` (acceptance holders) + `accept`/`reserve --help` |
| D19–D21 | `agrun --help` and its subcommands; `work.md` (`agrun continue` is the resume for your own run) |
| D22 | `requests.md` (how you record the reading) + `agbudget --help` (how the run judges it) |
| D23 | `board.md` |
| D24 | `board.md` (reading is free) + `read --help` (✔ bare name, `--since`) |
| D25 | `work.md` § Observer |
| D26–D29 | `recheck --help` (verdicts, UNOWNED, "do not post"); `work.md` (run it first, act on it, quote it) |
| D30–D35 | `work.md` § Observer (facts, trial references kept) |
| D36, D37 | `receipt --help`; one sentence in `work.md` |
| D38, D39 | `hold`/`disposition --help`; one sentence each in `work.md` |
| D40–D43 | `work.md` (first three paragraphs) |
| D44 | `send --help`, `relation --help` |
| D45 | `work.md` (judge on evidence) |
| D46 | `requests.md` (last paragraph, seen live twice) |
| D47, D49 | `agrefs --help` |
| D48 | `requests.md` |

front's own rows: F3 and F4 → `requests.md` (desk now gets them too). F5
→ `requests.md`. F6–F8 → `requests.md` (one sentence) + `options`/`use
--help`. F9 → `requests.md`. F10, F11 → `work.md` (desk now gets them
too). F12 → `board.md`. F13 → dropped.

routine_run: R1–R3, R6, R8, R12, R13 stay in its own guide. R4 is kept as
facts, with the mechanics in `use --help`. R5 → `work.md` + `send --help`.
R7 → `work.md` + its own guide ("ask them in your report instead"). R9,
R10 → `work.md`. R11: the reading rules → `agbudget --help`; what the run
writes and does stays (name the window, met at the start, hold with what
would resume, a reset, reaching mid-run, ending ≠ achieving).

argue: only A8 (document contents → `agproject open --help`; `status`
replaces the `pj-study…` naming, which no study follows: `pj-aisvgs`),
A10 (agrefs → `board.md`) and A11's context-panel line (→ `agrefs --help`)
changed. The rest is its own contract and stays.

## Dropped as Anxiety-Driven

| paragraph | text | why it may go |
|---|---|---|
| D7 (desk), F2 (front, first half) | "If the last message is small talk or something plain text answers, just reply." | no trial behind it. It is the branch run-0160 took: read literally, it permits answering without looking |
| D10 (desk), F2 (front, last line) | "If nothing on the board can do it, say so kindly." | no trial; the proposal paragraph and the board cover it |
| D11 (desk), F13 (front) | "If the work is already done, say so." | no trial; "answer from the board" covers it |

A later phase can restore any of these from agfront `ba28e90^` (the last
commit before the rewrite on the branch; the same text is on `main` until
deployment).

## Evidence-Driven paragraphs: all still reach the agent

Every evidence key in report1 was checked against its new home:

| key | now in |
|---|---|
| inc (run-0160) | `board.md` head; `agentchat --help` board paragraph |
| prop | `requests.md` |
| fd1 | `requests.md`, routine_run ("never post into `guide`") |
| fd-wr | `work.md`; argue ("never post into a `workrun-` topic") |
| fd2 | `requests.md` (no preface) |
| as5-8 | `work.md` (each serving ends) |
| as9 | `board.md` (a post makes them run); `agentchat --help` ("how is it going?") |
| rt1 | routine_run § Ending the run (unchanged) |
| rt2 | `requests.md` (mechanism kept) |
| ag1 | argue (unchanged) |
| agm1 | `requests.md` + `agproject open --help` |
| agm2 | `work.md` ("never write 'I played it'", kept) |
| fs1–fs3A2 | `work.md` § Observer (trial names kept) |
| fs4, fs5E | `recheck --help` + `work.md` |
| fs5 | `work.md` + `accept`/`reserve`/`agrun --help` |
| fs6r | `receipt --help` + `work.md` |
| fs6h, fs6x1 | `hold`/`disposition`/`send`/`relation --help` + `work.md` |
| fs6x2 | `work.md` (proxy paragraph, word for word) |
| pp1 | `agrun finish --help` ("until the end record exists…") |
| rp | `requests.md`, routine_run, `options`/`use`/`agbudget --help` |
| sp2 | `requests.md` (setup ≠ research) |

## Sizes (guide text only; the conversation is not counted)

| role | old chars / lines | new chars / lines | with reply + continuation sections (+4 198) |
|---|---|---|---|
| desk | 21 798 / 367 | 13 607 / 236 | 25 996 → 17 805 |
| front | 22 731 / 362 | 13 436 / 234 | 26 929 → 17 634 |
| routine_run | 16 647 / 284 | 15 479 / 271 | 20 845 → 19 677 |
| argue | 9 653 / 191 | 9 688 / 195 | 13 851 → 13 886 |

The line counts for the new guides are measured with the shared files
included, since those are what each role reads. The shared text is written
once: `board.md` 35 lines, `requests.md` 77, `work.md` 113, plus the desk
and front heads (8 and 6 lines); routine_run's own 121 lines and argue's 165
come on top. run-0160's desk prompt was 27 730
characters with the conversation; the same serving would now be about 8 200
characters shorter. routine_run and argue barely shrink, because they now
also receive what the board is and what each serving is judged on, text
they did not have before. What moved out of them is about the size of what
came in.

## Tests

On the branch, agfront gives 194 passed and 1 failed. The failure is
`test_listener_is_the_skeleton…`, which needs `.local/zulip.env` and
passes in the main checkout. New and changed tests:

- `test_every_conversational_guide_says_where_the_board_is`: all four
  roles are told "yours to look up", "not the filesystem" and "Reading
  costs nobody anything", and none says "just reply".
- The routine-opening and budget tests read the composed guide and the
  tools' `--help` (the facts moved there).
- The size test from step 3 passes again.
