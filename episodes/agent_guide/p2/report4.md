# agent_guide p2 — step 4: the guides rewritten

Every role now opens with p1's head where it has somebody to speak with:
who it is and whom it speaks with, what is assumed of it, and what it can
see, one line per tool with its `--help`. What follows are the facts it
works under. Tool usage lives in the helps (step 2), text several agents
share in pyagag (step 3). All on `agent-guide-p2`: archsage `d176ec8`,
agautolab `51d7973`, agforge `db17875`, pj-agdev `c539f80` (agobserver),
pj-clusterintent `0cf66c3` (cagent).

## Per role

| role | head now says | body changes | rows (report1) |
|---|---|---|---|
| archsage | knowledge council; speaks with whoever asks in its channel (the developer, Front, a routine run) and with everybody in an argue when named; the requester assumes it knows every sage, study and routine, so a new name is its to look up; what it can see: the trees (placed above), `archsage`, `agproject`, `agroutine`, `agentchat`, `agrefs` | "Establishing a study" states what the kit is and what is archsage's to write (the plan, the routine guide), and points at the commands; the three shapes, "report what exists precisely" and setup ≠ research ≠ refreshed stay. The refresh paragraph keeps its fact (a refresh that misses the commit has not refreshed that request's knowledge — say so) and leaves the record's syntax to `sage sync --help`. The argue paragraph drops "read the whole conversation" and "a mention costs a run" (pyagag's participant guide says both) | AS1–AS22 |
| sage | unchanged: one domain, one tree, `sagetree` only | the command list goes to `sagetree --help`; queueing is one sentence; "your reply is posted for you" leaves (REPLY_GUIDE says it); the header fact stays | SG3, SG4, SG6 |
| autolab planner (superdirector) | autolab's planner; this topic is one mission; the requester is the developer or an agent (Front, a routine run, archsage through `agproject`); what it can see: the project folder and `README_PROJECT.md`, `autolab doc patterns`, the introductions file, `agentchat`/`agrefs`, and that another project's state is on the board, not in the folder | the flags become a list of "the requester's decisions are files you write"; "asked to execute → planning phase" (WP8) is the fact "you plan; you never run a task". trend7 (task files) and fs3p (an agreement posted here closes nothing) stay word for word, fs3p now with its reference. The title-line contract is in its own words | WP1–WP16 |
| autolab worker (supercoder) | autolab's worker for one task; the requester (developer, or the agent that asked for the mission) reviews here; what it can see: the mission copy and `README_PROJECT.md`, the introductions file, `agentchat`/`agrefs`, `agag --help` | the paragraphs from smoke, fs4r, sub26 and fs2T1 are untouched; `review.md` is untouched | WR1–WR16, RV |
| autolab bmining | autolab's partner in a `bmining-` conversation with the developer | "decline other work" says where other work goes (a `workplan-` topic) | BM1–BM3 |
| forge plan front | forge's plan front; speaks with the requester (developer, autolab's task runs, Front); files start planning | the numbered steps stay (they are the output contract); `toolsets.csv` says "by the names `agforge toolsets --list` prints" | FP1–FP4 |
| forge planner | answers mostly with files: what `plan.md`, `idea.md` and the final message become; what it may consult: `tools/`, `agforge knowledge` (help), `agrefs` | the four outcomes are one list; knowledge states and `--init-image` usage are in the helps | FG1–FG8 |
| forge run | carries out `plan.md`; files and report are the delivery | what it may consult in one paragraph, knowledge usage → help; the long-job, notifier and failure paragraphs stay | FR1–FR8 |
| Observer intake, observe, triage | unchanged: their heads already say what they look at, what they write and who reads it (report1). No board facts: they converse with nobody | observe's `--count 30` → `agentchat read --help` | OO6 |
| cagent front | unchanged | the title-line contract in its own words | — |
| agfront desk, front, routine_run, argue | unchanged | step 3 only | — |

routine_run and argue: step 1 found nothing duplicated or transcribed in
them (report1 § agfront), so they are left as they are, as p1 judged.

## Grants

Every tool a rewritten guide points at is already in its role's
`allowed_tools`: archsage (`archsage`, `sagetree`, `agproject`,
`agroutine`, `agentchat`, `agrefs`), autolab's working roles (`autolab`,
`agentchat`, `agrefs`), forge's front (`agforge`, `agentchat`, `agrefs`)
and generator (`agforge`, `agrefs`). No new CLI, so no script entry and no
grant change. One existing mismatch was left as it was: the worker's line
about `agag init` predates this phase, and autolab's grant has no
`Bash(agag:*)` (it reaches `agag` through `uv`, if at all). Not touched
here: nothing in this phase points at it anew.

## Sizes

Characters (lines) of what the role is told besides placement and the
conversation: its own guide files plus the shared sections it now gets,
without the reply and continuation sections (which did not change).
"before" is the live checkout, "after" the branch. Line counts are not
comparable for autolab and forge: their old guides were one paragraph per
line.

| agent | role | before | after |
|---|---|---|---|
| agfront | desk | 14 077 | 15 173 |
| agfront | front | 13 906 | 15 002 |
| agfront | routine_run | 15 949 | 17 045 |
| agfront | argue | 9 688 | 10 723 |
| archsage | archsage | 10 114 | 11 289 (own guide 8 834) |
| archsage | sage | 1 837 | 1 430 |
| autolab | planner | 6 325 | 7 949 |
| autolab | worker | 5 304 | 7 650 |
| autolab | bmining | 736 | 701 |
| autolab | entrance | 1 441 | 1 846 |
| autolab | argue context | 1 307 | 1 481 |
| forge | plan front | 1 208 | 1 633 |
| forge | planner | 2 821 | 2 639 |
| forge | run | 2 972 | 3 139 |
| forge | entrance | 944 | 1 120 |
| forge | argue context | 1 067 | 1 241 |
| Observer | intake | 3 011 | 3 285 |
| Observer | observe | 3 830 | 3 795 |
| Observer | triage | 2 344 | 2 344 |
| Observer | argue context | 1 107 | 1 293 |
| cagent | front | 1 333 | 1 511 |

Why most did not shrink: the roles now **receive facts they lacked**, and
the plan does not make length a target.

- agfront's four roles gain about 1 100 characters net: pyagag's board
  and callback carry the ✔ and "how is it going?" trial references and
  the section headings that agfront's `board.md` and `work.md` had
  without them; agfront's own files lost the same facts.
- archsage gained the board and callback sections (it had neither) and
  lost 1 280 characters of its own.
- autolab's planner and worker gained the board (neither was told that
  another project's state is readable on the board) and a head; the
  worker also the callback. Their own guides are about the same size.
- The entrances gained the as10 trial reference and the second-person
  wording "whoever asks here — a person or another agent".
- The argue contexts: the references pointer moved from each `role.md`
  into one shared text that is slightly longer than the short variant it
  replaces.

## Paragraphs dropped

None of these had a trial behind it (report1, class A or stale):

| text | where | why dropped | restore from |
|---|---|---|---|
| "Your reply is the closing message of this run and is posted for you." | archsage | REPLY_GUIDE, appended to the same prompt, says it | archsage `df6e0af` |
| "Keep it well under 10,000 characters; `agroutine` reads it back and fails on truncation." | archsage | the number is Zulip's pre-failsafe-p4 limit (100 000 now); the read-back fact stays | archsage `df6e0af` |
| "Every definition change is committed … and the introduction is re-posted." | archsage | moved into `archsage --help` and each subcommand's help, not dropped | — |
| `autolab-front/guide.md` (`brainmining.flag` / `mission.flag`) | autolab | read by no code | agautolab `aab6810` |
| forge's `entrance_front/guide.md` | forge | pyagag's default vocabulary with forge's prefixes, word for word | agforge `0558c6e` |

## Duplicate check

After this step, over all 33 guide files of the six agents plus
`agag/guides/*.md`, no line longer than 40 characters occurs twice
(`… | awk 'length>40' | sort | uniq -c | awk '$1>1'` prints nothing).

## Tests

With the pyagag branch installed: archsage 40, agautolab 331, agobserver
177, cagent 204 passed; agfront 194 passed + the one `.local/zulip.env`
test; agforge 252 passed + the four sibling-path tests (all worktree
artefacts, report2/report3).
