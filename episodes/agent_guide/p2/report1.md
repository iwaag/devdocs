# agent_guide p2 — step 1: inventory of the other agents' guides

This inventory uses p1's method (`../p1/report1.md`): one row per paragraph,
with the commit that added it (`git blame -w -M -C` in the repository that
owns the file, read together with the commit subject and body), the trial
or episode behind it, other guides carrying the same text, the command it
describes, and a class. Later steps delete text only where a row below
says so.

## Classes

p1's four classes, and one more for the roles that answer with files.

| class | meaning | destination |
|---|---|---|
| **F** fact | a fact the agent works under; no tool can say it | the role's guide, short |
| **T** tool | describes what one command does | that command's `--help`; the guide keeps at most one line |
| **S** shared | the same text in several roles or agents | written once in pyagag (step 3); per-role follow-up stays |
| **A** anxiety | no trial or failure behind it | may be dropped; listed in `report.md` with the commit to restore from |
| **C** contract | the shape of a structured answer (a file, a flag, a JSON key) that the agent's own code reads | stays in the role's guide; Tool Giving applies to what the role may consult, not to the shape of its answer |

"design" in the Evidence column means the paragraph arrived with a new
feature and has no failure behind it. That is not anxiety: such a paragraph
moves but is not dropped.

## Evidence keys (new in p2)

p1's keys (fd1, as5-8, fs1…fs6x2, rp, sp2, …) are reused where they apply.

| key | what was seen | commit |
|---|---|---|
| as10 | agent_standardize p10 step 4 (Failure Farming): autolab's entrance answered from `pj-simpleshooter` alone, missed `pj-runsmoke1`, and quoted its own earlier reply as the current state | autolab `8b861d3`, forge `04b7b0c` |
| trend7 | 2026-09-08 `workplan-trend7`: a plan with no task file was reported as started and nothing ran | autolab `c06b02c` (the sentence names it) |
| smoke | 2026-08-18 run smoke test: `main/` left uncommitted; the report was written where the handler did not read it | autolab `4dd892b`, `790837a` |
| sub26 | 2026-09-26: a run started five subagents, ended its turn, was killed ten minutes later with one still running; nothing woke the task for an hour | autolab `a971bdd` (failsafe p1 step 3) |
| fs2T1 | failsafe p1 T1 / p2 step 1: a task closed on Front's "continue and report" after a stop | autolab `8f155f1` |
| fs3p | failsafe p3 step 2: a task agreement posted in the plan's topic closed nothing and nobody was told | autolab `2fcaaa4` |
| fs4r | failsafe p4 step 2: a post after a shown result was served as new work; the agreement must bind the reviewed state | autolab `bcc51a1` |
| comfy | 2026-09-01: a run started a comfy watch by running the notifier CLI instead of posting the mention | autolab `3dcc590` (subject only) |
| fs5-1 | failsafe p5 step 1 §5: one fixed refresh topic per study returned every answer to the first request | archsage `fe27390`; routine guides v3 |
| fs5E | failsafe p5 trial E: a refresh request opened with `sage:<name>`, which the sage cannot perform; one extra round | archsage `041e1ac` |
| rw3C | robust_workflow p3 trial C: a ✔ task waiting for the human's acceptance judged a stall twice | agobserver `455c2b3` |
| rw3r | robust_workflow p3 step 6 replays of p2 E3: 2 of 6 judgments read the listener's "Message received" as work running; 3/3 correct after | agobserver `74c881f` |
| fs1t | failsafe p1 step 2 (the 2026-09-26 run above): "still running" was a claim by a serving that had already ended | agobserver `fd9cfd4` |
| ob2 | observer p2 step 3, watch w6866: "BOTH jobs ended" judged `met` with "…BUT … still listed in queue_running" in its own evidence | agobserver `aba1a9c` |
| ob1 | observer p1 steps 1–2: the three-valued look and the intake decision (design, measured over 19 looks) | agobserver `4c26f64`, `87fa4d6` |
| ad0 | 2026-08-14: forge's generator refused a transparent background with the tools already granted and undescribed | agforge `ad0d628` (in `tools.md`, not the guide) |
| si | study_import p1 step 4: knowledge sources for forge's planner and run (design) | agforge `a7297bc` |
| ag2r | adventure_game p2 step 3: references for every working role (design) | autolab `4939ca9`, forge `43730aa`, archsage `b8099ac` |
| gce | give_context_easier p1: the shared reference catalog (design) | agobserver `1f8c177`, cagent `47e0273` |

## How each agent composes a prompt

The mechanism is `agag.topics.prompt_with_guide` everywhere (there is no
`shared_guide` function in pyagag: the agents import `agag.topics.guide`
under that name). Two compositions do not go through it:

| path | composition | who |
|---|---|---|
| `prompt_with_guide(lines, guide, reply=…, continuation=…)` | placement, guide, `REPLY_GUIDE`, `CONTINUATION_GUIDE` | agfront (desk, front, routine_run, argue; present without reply), autolab (superdirector, supercoder, bmining), forge (assetplan_front; the generator without reply), archsage (archsage), cagent (front), agobserver (intake, observe, triage: no reply) |
| `agag.argue.participant_prompt` | placement, desire, conversation, `participant_guide()` (a Python string in pyagag), then `agent/guides/argue/role.md`, then `REPLY_GUIDE` | the argue roles of autolab, forge, agobserver, cagent, archsage |
| `agag.entrance.entrance_prompt` | placement, own channel, conversation, `entrance_front/guide.md` **or** pyagag's `DEFAULT_GUIDE`, `REPLY_GUIDE` | autolab and forge have their own guide; every other agent gets the default |
| archsage `sage_context` | sage guide + the sage's domain guide + tree state | archsage's sages |

`CONTINUATION_GUIDE` reaches agfront's roles only. Step 3 has to change
both `prompt_with_guide` and `participant_prompt` for text that argue
participants share.

## archsage (`archsage/agent/guides/archsage/guide.md`, 189 lines, 10 115 chars)

Served on the `archsage` role (frontier profile; `archsage`, `sagetree`,
`agrefs`, `agproject`, `agroutine`, `agentchat`). Conversational, delegating
(asks autolab through `agproject`, others through `agentchat send`), reply
contract on.

| # | lines | content | commit | evidence | also in | command | class → destination |
|---|---|---|---|---|---|---|---|
| AS1 | 1–3 | who: the knowledge council; designs knowledge and research, knows the sages, establishes studies | 6080fe9, 5f454b4 | design (argue p1, sage p2) | — | — | F → head |
| AS2 | 7–13 | a sage = one domain, one tree, one guide; you read every tree; `archsage ask`; define and maintain (`archsage sage …`) | 6080fe9, 5f454b4 | design | intro (`params/intro.md`) | ask, sage | F+T → head: one line per tool, usage to `archsage --help` |
| AS3 | 15–26 | the three-part analysis: useful knowledge (check the tree; a sage run costs the ordinary model), research questions (queues), missing domains | 6080fe9, 5f454b4 | design | — | queue list | F (the role's contract) |
| AS4 | 28–30 | no fixed checklist of domains | 6080fe9 | design | — | — | F (one sentence) |
| AS5 | 34–36 | a study is the kit a sage's knowledge comes from; you decide, the tools make it | 5f454b4 | sp2 | — | — | F |
| AS6 | 38–47 | project channel/plan/workspace: write the plan; `agproject open --kind study --doc --about`; pending until autolab answers; end with `intent=progress`; autolab's answer brings you back | 5f454b4 | sp2 | agfront `requests.md` (project), `agproject open --help` | agproject open | T+F → `agproject open --help` already has the document and states; the guide keeps "pending until autolab answers, and its answer brings you back" |
| AS7 | 48–50 | `agroutine create study-<slug> --guide-file`; starts nothing | 5f454b4 | sp2 | — | agroutine create | T → `agroutine create --help` |
| AS8 | 51–58 | `sage add … --project`, `sage attach`; `main` default, `--source publish`; commits and re-posts the introduction | 5f454b4, b57f71a | sp2 | intro | sage add/attach | T → `archsage sage add/attach --help` (the re-post claim is right: `cli._after_change` re-posts the introduction on add/update/attach/remove; only `sync` does not, p1 report5) |
| AS9 | 60–65 | check before you create (`agproject status`, `agroutine show`, `sage list`); reuse; repeats complete, never duplicate; each tool's `--help` says inputs/outputs/states | 5f454b4 | sp2 | — | status, show, list | F+T → one fact ("check what exists; repeating a create completes it"), usage in helps |
| AS10 | 67–76 | three shapes: new study; study not connected; sage without study | 5f454b4 | sp2 | — | — | F (keeps; each shape is a fact about the kit) |
| AS11 | 78–84 | report what exists precisely; setup complete ≠ research complete ≠ knowledge refreshed | 5f454b4 | sp2 (setup ≠ research) | agfront `requests.md` (setup ≠ research) | — | F (archsage's own; Front's line is the requester's side) |
| AS12 | 88–114 | a study routine's guide: first line, display line, "one mission", goal quote, operational context, acceptance, the refresh in a topic of the run's own addressed to archsage (not `sage:`), the report | 5f454b4, fe27390, 041e1ac | sp2, fs5-1, fs5E | — (the three routine guides in Zulip) | — | F (the contract of a document archsage writes; the two trial sentences stay word for word) |
| AS13 | 116–117 | keep it under 10 000 chars; `agroutine` fails on truncation; update with the whole guide | 5f454b4 | design | — | agroutine update | T → `agroutine update --help` |
| AS14 | 121–133 | `sage sync --require <commit>`; the `[selfnote][sagesync]` record: `for=`, `includes=`/`missing=`; say so when it misses; then the queue: `queue list`, `queue resolve --answered-by`; asking for research is not an answer | fe27390, 5f454b4 | fs5-1 | — | sage sync, queue list/resolve | T+F → `sage sync --help` gets the record and what `missing=` means; `queue resolve --help` gets "files must be in the tree; asking for research is not an answer"; guide keeps "a refresh that misses the commit has not refreshed that request's knowledge — say so" |
| AS15 | 137–143 | every topic in your channel is yours; plainly one sage's question → `sage:<name>`; the listener names the requester (no mention); a requester may be an agent: `intent=progress` for pending, `intent=report` for finished | 5f454b4 | design (explicit_reply) | REPLY_GUIDE (the intents) | — | F (the intent sentence is REPLY_GUIDE's; one clause left) |
| AS16 | 145–149 | when you delegate, the answer comes back beside the chatlog; continue, report | 5f454b4 | design | agfront `work.md` ("each serving ends") | send | S → shared "each serving ends" text (step 3) |
| AS17 | 153–161 | argue: read all; the desire is placed above; give the analysis; complete enough to stand; a sage via `@**archsage** sage:<name>`; do not name others (a mention costs a run) | 6080fe9, 5f454b4 | design; ag1 (mention cost) | `participant_guide()` (read all, mention cost), agfront argue | — | F+S: "read all", "mention costs a run" are already in `participant_guide()` for the argue path; the guide keeps what archsage's analysis is for |
| AS18 | 163–167 | you have no conversation of your own in an argue: do not establish a study there; the facilitator asks in your channel | 5f454b4 | sp2 (the routing that made archsage the establisher) | — | — | F |
| AS19 | 171–174 | honesty: a tree that does not contain it does not answer; general knowledge labelled; never edit a tree | 6080fe9, 5f454b4 | design | sage guide (same facts for the sage) | sage sync | F |
| AS20 | 176 | your reply is the closing message and is posted for you | 6080fe9 | design | sage guide; REPLY_GUIDE says it | — | A (duplicate of REPLY_GUIDE, which this role receives) → drop |
| AS21 | 180–184 | agrefs pointer (5 lines) | 4c94d42 | ag2r | 7 more files (below) | agrefs | S → pyagag shared section |
| AS22 | 186–189 | a reference is a project's input, not a study's finding; a seed for a plan is cited as `<source>@<rev>:<path>` | 5f454b4 | design | — | — | F (archsage's follow-up) |

`--help` mentions: 2. Tools named but not pointed at: `archsage`,
`agroutine`, `agproject`, `sagetree`.

## sage (`archsage/agent/guides/sage/guide.md`, 34 lines)

Structured by design: one bounded reader (`sagetree`), no chat tool.

| # | lines | content | commit | evidence | class → destination |
|---|---|---|---|---|---|
| SG1 | 1–8 | one sage: domain, tree, guide; answer from the tree, cite with revision; a tree with only the plan has no findings | 6080fe9, b57f71a | design; sp2 (plan ≠ findings) | F → head |
| SG2 | 10 | read the conversation first | 6080fe9 | none | A? It is the one line saying where the question is; the sage has no other placement line saying so. Keep as head fact |
| SG3 | 12–16 | `sagetree ls/cat/grep/find/revision`; nothing else reachable; cite by path | 6080fe9 | design | T+F → `sagetree --help` gets the command list; guide keeps "everything you may read is there and nothing else" |
| SG4 | 18–25 | tree does not answer → say so; `sagetree queue list/add`; appends on the same slug; archsage removes a note only when answered | 6080fe9, b57f71a | design | T+F → `sagetree queue --help`; guide keeps "queue a researchable question in your domain" |
| SG5 | 27–29 | outside your domain: general knowledge, labelled | 6080fe9 | design | F |
| SG6 | 31–34 | never mention anybody (costs a run); the reply is posted for you under a header | 6080fe9 | ag1 (mention cost) | F (the mention half); the "posted for you" half duplicates REPLY_GUIDE where it is appended — the sage's own-channel serving: see below |

A sage served in archsage's channel is prompted by `archsage.roles.run_sage`
through `sage_context`; in an argue, through `participant_prompt`. The
argue path appends REPLY_GUIDE; the own-channel path is checked in step 4
before SG6's second half is touched.

## agobserver

Observer's roles act on a schedule or a program's call, not on a
developer's word. None of them converses: intake, observe and triage write
a JSON file that code reads (class **C**), and they run on the local model
through agcode with the `read`/`list`/`run` tools. Their "head" is
therefore *what the role looks at, what it may write, and who reads it*;
there is nobody to speak with.

### intake (`agent/guides/intake/guide.md`, 67 lines)

| # | lines | content | commit | evidence | class → destination |
|---|---|---|---|---|---|
| OI1 | 1–3 | one watch request → three short answers; not observing, not replying; read by a program | 4c26f64 | ob1 | F → head |
| OI2 | 5–28 | `decision.json` and its keys; destination verbatim | 4c26f64 | ob1 | C |
| OI3 | 30–47 | when to refuse: no condition, target, or destination; a phrase is not a destination; one question | 4c26f64 | ob1 | C+F |
| OI4 | 49–54 | read the whole conversation; newest posts answer our question | 4c26f64 | ob1 | F |
| OI5 | 56–59 | you may use `read`/`list`/`run`, but need not | 4c26f64 | ob1 | F (head: what it can see) |
| OI6 | 61–65 | agrefs variant: `agrefs list` through `run`, `agrefs show …`; keep the exact reference | 1f8c177 | gce | S → shared agrefs section (the local role's `run` tool is the one fact that stays) |
| OI7 | 67 | finish by writing `decision.json`; final message ignored | 4c26f64 | ob1 | C |

### observe (`agent/guides/observe/guide.md`, 82 lines)

| # | lines | content | commit | evidence | class → destination |
|---|---|---|---|---|---|
| OO1 | 1–3 | one condition, once, now; not fixing, not answering | 87fa4d6 | ob1 | F → head |
| OO2 | 5–23 | `result.json`; met / not_met / unable with evidence | 87fa4d6 | ob1 | C |
| OO3 | 25–28 | unable ≠ not_met (the one mistake that matters) | 87fa4d6 | ob1 (design; measured in p1) | F |
| OO4 | 30 | nothing else in the file; final message ignored | 87fa4d6 | ob1 | C |
| OO5 | 32–47 | a condition with several parts: every part, same look; verdict must agree with evidence ("but") | aba1a9c | **ob2** | F (Evidence-Driven, stays word for word) |
| OO6 | 49–57 | `run`, `read`, `list`; `agentchat read <channel> <topic>`, `--count 30` | 87fa4d6 | ob1 | T+F → the `agentchat read` usage is `agentchat read --help`; the guide keeps the three tools and one line for `agentchat` |
| OO7 | 59–62 | look at the target only; do not explore, read this agent's code, or post | 87fa4d6 | ob1 (design: "no chatlog, no other watch") | F |
| OO8 | 64–69 | the condition means what it says, including how to tell (`.part`) | 87fa4d6 | ob1 | F |
| OO9 | 71–75 | a conversation condition: the recipient matters; progress/ack/others' questions are not the thing | 87fa4d6 | ob1 | F |
| OO10 | 77–79 | the previous observation is read; look at what changed | 87fa4d6 | ob1 | F |
| OO11 | 81–82 | nothing to look at → `unable` | 87fa4d6 | ob1 | F (restates OO3; may merge) |

### triage (`agent/guides/triage/guide.md`, 41 lines)

| # | lines | content | commit | evidence | class → destination |
|---|---|---|---|---|---|
| OT1 | 1–3 | stuck or waiting for a reason; everything is above; you only answer | 6a2abc5 | design (robust_workflow p1) | F → head |
| OT2 | 5–12 | `stall` / `legit` / `unclear` with their meanings | 6a2abc5 | design | C |
| OT3 | 14–18 | a ✔ does not stop anything; look at the request's own conversation: asked and unanswered → legit, nobody told → stall | 455c2b3 | **rw3C** | F |
| OT4 | 20–23 | "Message received. Please wait for the reply." is the listener's pick-up, not work | 74c881f | **rw3r** | F |
| OT5 | 25–33 | "still running" is a claim; the trace says whether a serving is open; believe it only while fresh work appears | fd9cfd4 | **fs1t** | F |
| OT6 | 35–36 | judge from the messages, not elapsed time; quote ids | 6a2abc5 | design | F |
| OT7 | 38–41 | `result.json` shape with an example | 6a2abc5 | design | C |

### argue (`agent/guides/argue/role.md`, 18 lines)

| # | lines | content | commit | evidence | class → destination |
|---|---|---|---|---|---|
| OA1 | 1–5 | who: the agent that waits; every minute on a local model; posts once | ed86fbc | design | F → head (Observer's own pitch; also in its introduction) |
| OA2 | 7–12 | contribution: waiting, watching, noticing; what can/cannot be observed; no watch from here | ed86fbc | design | F |
| OA3 | 14–18 | agrefs short variant | 1f8c177 | gce | S |

`--help` mentions: 1 (in the agrefs variant). Observer's roles send no
tool to a `--help` otherwise; `agobserver.withdraw`, `hold`, `disposition`
and health are operator commands, not something its model roles run.

## agautolab

Roles served: `superdirector` (the `workplan-` conversation, reply
contract), `supercoder` (a `workrun-` task, reply contract; `review.md`
appended after a shown result), `director` (`bmining-`), `front` (the
entrance), `argue`. All working roles hold the same grant, including
`agentchat` and `agrefs`; no role holds `agproject` or `agrun` (they are
not autolab's tools).

### workplan_superdirector (`guide.md`, 70 lines, 6 327 chars)

| # | lines | content | commit | evidence | also in | command | class → destination |
|---|---|---|---|---|---|---|---|
| WP1 | 2–3 | the topic is about planning the next mission; the reply goes to the developer | bc77f4d, 64e5ab9 | design | bmining (l.3), workrun (l.3) | — | F → head, rewritten ("who you speak with": the requester, often another agent) |
| WP2 | 7–10 | `README_PROJECT.md` explains the folders; write it when missing; edit only on structural change | bc77f4d | design (project_pattern) | workrun (l.5) | — | F |
| WP3 | 12 | `autolab doc patterns` explains layouts by pattern; follow it; unknown pattern → say so | 09fbdf6 | design | — | autolab doc | T → `autolab doc --help`; one line |
| WP4 | 16 | read the project index in `main/` first; an existing investigation → point at it | 0dca020 | design ("planner checks the index first") | — | — | F |
| WP5 | 18 | write `plan.md` when the mission is clear; posted as the current plan | c06b02c | design (refactor p1) | — | — | C |
| WP6 | 20 | one `task[N].md` per task; a mission runs only through task files; a one-piece mission still has `task1.md` | c06b02c | **trend7** | agfront `work.md` ("no task files → ask for a re-plan") | — | C+F (Evidence-Driven, stays) |
| WP7 | 22–23 | first line is a heading → title; the rest → description | a468621, 1a7d01d | design | forge generator, cagent front (same sentence) | — | C (each agent's own parser; reworded so it is not a repeated paragraph) |
| WP8 | 25 | asked to execute → it is the planning phase | bc77f4d | none | — | — | A? It is the one sentence that tells the planner it does not run tasks; kept as a head fact ("you plan; tasks run elsewhere") |
| WP9 | 27–29 | `start.flag`, `cancel.flag`, `accept.flag` and what each does; accepting one task ≠ the mission | 85f53a3, 64e5ab9, 2e2c039 | design (robust_workflow p1, p3) | — | — | C |
| WP10 | 31 | a task closes only in its own topic; `status.md`; an agreement here closes nothing — say so and point; never say closed unless `completed` | 2fcaaa4 | **fs3p** | agfront `work.md` (closed = record `completed`, fs3A2) | — | F (Evidence-Driven; the planner's side of the same fact) |
| WP11 | 35–39 | adjusting vs replacing; `replace.flag` with the plan in the same run; what retiring does | 83dbcc2 | design (refactor p2) | — | — | C |
| WP12 | 41 | need more discussion → ask, edit nothing | 64e5ab9 | design | — | — | F |
| WP13 | 43 | you plan in the project folder (integrated work only); tasks run in a mission copy; what you write is committed as the plan's notes | 09615fb | design (robust_workflow p3 ex1) | workrun (WR5) | — | F |
| WP14 | 47–51 | a task may be a request to another agent; name the agent and what to ask; one request per task | ffa8ef2 | design | — | — | F |
| WP15 | 55–59 | agrefs pointer | 2efc711 | ag2r | 7 more | agrefs | S |
| WP16 | 61–70 | a named reference: `agrefs sync`, record in `direction/REFERENCES.md`; tasks name paths at that revision; `agrefs changes` on a new adoption | 4939ca9 | ag2r | — | agrefs sync/changes | F+T (autolab's follow-up; the command usage is already in `agrefs --help`) |

The planner is told nothing about what it can *see*: that the board is
readable (`agentchat`), that other agents' introductions are placed
(`tools/agents.md`, named by the prompt's placement), and that another
project's state is `agentchat topics pj-<slug>` away. That is the p1 head
the step-4 rewrite gives it.

### workrun_supercoder (`guide.md`, 75 lines, 5 306 chars) and `review.md` (28 lines)

| # | lines | content | commit | evidence | also in | command | class → destination |
|---|---|---|---|---|---|---|---|
| WR1 | 2–3 | the topic is the work of this session; the reply goes to the chat | 4dd892b | design | WP1 | — | F → head |
| WR2 | 5 | `README_PROJECT.md` explains the folders | bc77f4d | design | WP2 (same sentence) | — | F (said once per role; reworded) |
| WR3 | 7 | do the work following the request | 790837a | smoke | — | — | F (head) |
| WR4 | 9 | `agag init <name> --yes --provision --like <sibling>`; `agag --help` | 2b2a0dc | design | — | agag init | T → `agag init --help`; one line |
| WR5 | 11 | the mission copy: others' work is not in it; commit freely; a commit is not acceptance; you cannot push | 09615fb | design (robust_workflow p3 ex1) | WP13 | — | F |
| WR6 | 13 | show the result; `intent=report`; deliverables inside a repository, not the copy's own folder; scratch ignored; the listener integrates and says so — do not say pushed/closed | bcc51a1 (+790837a, 4dd892b) | smoke, fs4r | REPLY_GUIDE (intents) | — | F+C (Evidence-Driven) |
| WR7 | 15 | README_PROJECT says which folders are repos; `main/direction/devlog` published at close; `publish.flag` names others | bcc51a1 | design | — | — | C |
| WR8 | 17 | a refused close (another mission changed the files) → `git merge`, test, show the combined result | bcc51a1 | design (robust_workflow p3 ex1) | — | — | F |
| WR9 | 20 | a task started by autolab itself; "start" for a task already done/doing → say where it stands | 85f53a3 | design (robust_workflow p1) | — | — | F |
| WR10 | 24–33 | work in flight ends with the reply: wait for every subagent/background command; say "still going" only when something will wake the task | a971bdd | **sub26** | — | — | F (Evidence-Driven, stays whole) |
| WR11 | 35–41 | resuming after such a stop: start from what the chatlog and the copy say; check nothing still runs; a resume is not the developer agreeing | a971bdd, 8f155f1, bcc51a1 | sub26, **fs2T1** | agfront `work.md` (resume ≠ agreement, fs2) | — | F (the worker's side of the fact; stays) |
| WR12 | 46–48 | the introductions file says how to reach each agent; `agentchat` (`--help`) | ffa8ef2 | design | agfront `board.md` | agentchat | S → shared board section |
| WR13 | 50–51 | post the request and finish; you are called again when they answer | 4f30af4, ffa8ef2 | as5-8 | agfront `work.md`, archsage AS16 | — | S → shared "each serving ends" |
| WR14 | 53–60 | a long ComfyUI generation: post `@**Comfy Notifier** watch <id>` in this topic, report what is pending, finish; the notifier posts back two lines; public channels only; quote the command in a fence | 3dcc590 | comfy | forge's generator does it through `pending.json` (a different mechanism) | comfynotify | T+F → the watch command's usage belongs to the notifier's own introduction (`agentchat intro comfy…`); the guide keeps "a long job is handed to a watcher, not waited on" and the pointer |
| WR15 | 64–68 | agrefs pointer | 2efc711 | ag2r | 7 more | agrefs | S |
| WR16 | 70–75 | read references at the named revision; images via `agrefs path`; name `<source>@<rev>:<path>` to forge; report reused/transformed/new | 4939ca9 | ag2r | — | agrefs path | F (autolab's follow-up) |
| RV1–RV4 | review.md 1–28 | this serving answers a shown result; do not re-run; four moves (agree → `close.flag` / `hold.flag`; agree but copy differs → repair only that; change → make it, show it; question → answer) | bcc51a1 | fs4r | — | — | C+F (Evidence-Driven, whole; appended only on a review serving) |

### entrance_front (`guide.md`, 13 lines)

| # | lines | content | commit | evidence | also in | class → destination |
|---|---|---|---|---|---|---|
| EA1 | 1 | the entrance: answer about your own work from the chat; start no work | 49f337c | design | pyagag `DEFAULT_GUIDE` l.1, forge EF1 | S → the default's first line |
| EA2 | 3–6 | `channels --prefix pj-`, `--prefix work-`, `topics`, `read`; ✔ is finished | 49f337c, 8b861d3 | as10 | DEFAULT_GUIDE, forge | T+S → `agentchat` helps say what each prints; autolab's vocabulary (`pj-`, `work-`, `workplan-`, `workrun-task<N>-`) stays as the per-agent part |
| EA3 | 7–8 | list **every** `pj-` channel; your own earlier answers are history | 8b861d3 | **as10** | DEFAULT_GUIDE ("list the channel every time"), forge | S (Evidence-Driven; shared in pyagag with its trial reference) |
| EA4 | 9 | development work goes in a `workplan-` in the project's channel | 49f337c | design | DEFAULT_GUIDE `request_line` | F (per agent) |
| EA5 | 11 | close out when asked: read first, `agentchat resolve`, then `uv run python -m agautolab.mission_done` | c06b02c | design | DEFAULT_GUIDE, forge | S+T → the "when asked, read first, `resolve`" half is the default's; `mission_done` is autolab's (an operator-level module; its `--help` exists) |
| EA6 | 13 | your reply is posted for you; never `agentchat send` into this channel (posts twice) | 15e8d98 | explicit_reply p1 (double post) | DEFAULT_GUIDE, forge (word for word) | S |

### bmining_director (`guide.md`, 8 lines)

| # | lines | content | commit | evidence | class → destination |
|---|---|---|---|---|---|
| BM1 | 2–4 | the topic is about what the developer wants to create; fuel their imagination | 1bc25fd, 64e5ab9 | design | F → head |
| BM2 | 6–7 | every file is project information; record new information before replying; web search | 64e5ab9, d752394 | design | F |
| BM3 | 9 | asked to do other work → decline; this chat is for discussion | d752394 | none | F (the role's scope; one sentence) |

### autolab-front (`guide.md`, 3 lines)

| # | content | commit | evidence | class → destination |
|---|---|---|---|---|
| AF1 | `brainmining.flag` / `mission.flag` | fbab8c2 | none | **dead**: no code in agautolab reads this guide or either flag (`grep` over `src/` and `agents.toml`). Delete in step 4 (not a paragraph any role receives) |

### argue (`role.md`, 23 lines)

| # | lines | content | commit | evidence | class → destination |
|---|---|---|---|---|---|
| AA1 | 1–6 | who: develops software projects; a project is a `pj-` channel plus a workspace; studies are projects too | c242118 | design | F → head |
| AA2 | 8–15 | contribution: what building or studying would take; read `README_PROJECT.md` and `main/` of the workspaces named; do not plan from here | c242118 | design | F |
| AA3 | 19–23 | agrefs pointer | 2efc711 | ag2r | S |

## agforge

Roles: `front` (assetplan_front and the entrance; `agforge`, `agentchat`,
`agrefs`), `generator` (the planner `guide_plan.md` and the run
`assetrun_generator`; files as output, no chat tool), `argue`.

### assetplan_front (`guide.md`, 20 lines)

| # | lines | content | commit | evidence | class → destination |
|---|---|---|---|---|---|
| FP1 | 1–5 | a request to create → `required_items.md` (quote the requester, name the message); `agforge toolsets --list`; `toolsets.csv` | 19fa4ea, a7297bc | design | C+T (`toolsets --list` usage to `agforge toolsets --help`) |
| FP2 | 7 | otherwise ask them to clarify | 87ef0c7 | none | F (one clause; the role has no other branch) |
| FP3 | 11–15 | agrefs pointer | 73c9c8a | ag2r | S |
| FP4 | 17–20 | a named reference goes into `required_items.md` as given, with what it establishes | 43730aa | ag2r | F |

### assetplan_generator (`guide_plan.md`, 30 lines)

| # | lines | content | commit | evidence | class → destination |
|---|---|---|---|---|---|
| FG1 | 1 | `tools/` describes what you can use | 19fa4ea | design | F → head |
| FG2 | 3 | `agforge knowledge list/show/search/path`; general vs local; states (`verified` only means run here); nothing pre-selected | a7297bc | si | T+F → `agforge knowledge --help` gets the states and what each command prints; guide keeps "look at what is known first; only `verified` has run here" |
| FG3 | 5 | precedence: requester's words > `required_items.md` > local > general | a7297bc | si | F |
| FG4 | 7–9 | `plan.md` names knowledge as `<source>/<path>` and whether verified; ask a question in the reply; `idea.md` for enabling ideas and trade-offs | a7297bc, 7bd54c6 | si | C |
| FG5 | 11–12 | first line is a heading → title | 19fa4ea, ad0d628 | design | C (see WP7) |
| FG6 | 14 | otherwise reply that it is impossible | 87ef0c7 | none | C (one clause: the third outcome of the contract) |
| FG7 | 18–22 | agrefs pointer | 73c9c8a | ag2r | S |
| FG8 | 24–30 | `--init-image "$(agrefs path …)" --init-creativity`; say in `plan.md` how a reference is used; precedence | 43730aa | ag2r | T+F → the flag usage is `agforge image generate --help`; guide keeps "a reference may steer generation; say how in the plan" and the precedence |

### assetrun_generator (`guide.md`, 52 lines)

| # | lines | content | commit | evidence | class → destination |
|---|---|---|---|---|---|
| FR1 | 1–4 | do `plan.md` with `tools/`; `chatlog.md`'s last message is what was asked now | 19fa4ea, b6f6c8e | design | F → head |
| FR2 | 5–10 | `knowledge.md` says the revisions and whether they moved; `agforge knowledge show/search/path`; another approach is allowed, say what you used | a7297bc | si | T+F (usage → `agforge knowledge --help`) |
| FR3 | 11–19 | `result/`, `intermediate/`, `failure.flag`; name the kind of failure (environment / implementation / knowledge gap) | 5958b10, 089609b | design (study_import p1 step 6) | C |
| FR4 | 21–32 | long jobs: `video/music submit` → `pending.json` → finish; re-run with `watching.json`; `comfy fetch --into result` | 5f5b67a | design | C+T (`submit`/`comfy fetch` usage → their helps; `pending.json` contract stays) |
| FR5 | 34–37 | do not post the notifier line; you have no chat tool | 5f5b67a | design | F |
| FR6 | 39–40 | `image generate` returns in seconds; use it directly | 5f5b67a | design | F |
| FR7 | 44–48 | agrefs pointer | 73c9c8a | ag2r | S |
| FR8 | 50–52 | report which reference went in and how | 43730aa | ag2r | F |

### entrance_front (`guide.md`, 11 lines)

| # | lines | content | also in | class → destination |
|---|---|---|---|---|
| EF1 | 1–9 | the entrance; `topics`, ✔, `read`, list every time, a new request is an `assetplan-` topic; close out when asked | DEFAULT_GUIDE almost word for word (567f829, 04b7b0c: as10) | S — forge's guide **is** the default with `assetplan-`/`assetrun-` filled in; delete it and let `default_guide(spec)` serve (`plan_prefix`/`run_prefix` already carry the vocabulary) |
| EF2 | 11 | reply posted for you; never `send` into this channel | DEFAULT_GUIDE, autolab | S |

### argue (`role.md`, 19 lines)

| # | lines | content | commit | class → destination |
|---|---|---|---|---|
| FA1 | 1–4 | who: media assets to order; planned in your channel; delivered as a URL | 8b1de98 | F → head |
| FA2 | 6–11 | contribution: what can be made and how; `agforge toolsets --list`; no planning from here | 8b1de98 | F+T |
| FA3 | 15–19 | agrefs pointer | 73c9c8a | S |

## cagent (pj-clusterintent) — not in the plan's table, but in its duplicate check

| file | lines | content | class → destination |
|---|---|---|---|
| `front/guide.md` | 1–13 | `required_info.md` / `requested_change.md`; heading → title; a change is recorded, not carried out | C |
| `front/guide.md` | 15–19 | agrefs short variant (47e0273, gce) | S |
| `operator_read/guide.md` | 1–2 | provide `required_info.md` with `tools/` | C |
| `operator_read/guide.md` | 4–8 | agrefs short variant | S |
| `argue/role.md` | 1–13 | who; cluster reality; `tools/toolset_nctl.md`; no change from here | F |
| `argue/role.md` | 15–19 | agrefs short variant | S |

## agfront: routine_run and argue (waste only)

p1 left both as they were in length (report.md: 282 and 195 lines after
p1's step 4; 121 and 159 lines as guide text now that the shared files
hold the rest). Checked for duplicates and transcriptions only:

| # | where | content | finding |
|---|---|---|---|
| RR1 | routine_run 7–14 | the opening post, the routine guide (`agentchat read <channel> guide`), `tools/budget.md`, `agbudget --help` | facts of the run; the `agbudget` line is already a pointer. **No waste** |
| RR2 | 18–23 | one serving; delegate with `send` into the named entrance; a new topic per delegation | the "a new topic per delegation" and "resume is the exception" facts are routine-specific; `work.md` does not say them. **No waste** |
| RR3 | 25–32 | execution preference at every delegation | rp; `use --help` has the mechanics, the guide has the rule. **No waste** |
| RR4 | 34–47 | acceptance, never post into `guide`, `agrun adopt`, the entry | fs5; routine-specific. **No waste** |
| RR5 | 49–64 | conditions about usage | rp; after p1 these are the run's facts (the reading rules went to `agbudget --help`). **No waste** |
| RR6 | 66–121 | ending the run | rt1, whole; the listener contract. **No waste** |
| AR1 | argue 39–58 | inviting agents by the roster's `bot:` line; selectors; mention costs a run | ag1; Front's facilitator side. `participant_guide()` has the participant side. **No waste** |
| AR2 | 105–122 | setting it up: `GOAL.md`/`RESEARCHPLAN.md` (`agproject open --help` holds the contents), the three routes | already pointers since p1. **No waste** |
| AR3 | 155–159 | references | argue-specific, 3 lines. **No waste** |

Step 1 agrees with p1: routine_run and argue carry what their roles need,
and nothing in them is transcribed from a help or repeated elsewhere. They
are left as they are unless the shared sections of step 3 replace a
sentence they carry (none do: `board.md` is appended to both already).

## The same text across agents

| text | files | lines each | class → destination |
|---|---|---|---|
| agrefs pointer, long form ("`agrefs` reads what the developer has published for agents to build from — …") | archsage `archsage`; autolab `workplan_superdirector`, `workrun_supercoder`, `argue/role.md`; forge `assetplan_front`, `assetplan_generator/guide_plan.md`, `assetrun_generator`, `argue/role.md` — **8 files, 3 agents** | 5 | S → one pyagag section; each file keeps its follow-up |
| agrefs pointer, short form ("The developer publishes shared context — …") | agobserver `argue/role.md`, `intake` (variant with `run`); cagent `front`, `operator_read`, `argue/role.md` — **5 files, 2 agents** | 4–5 | S → the same pyagag section (one text for both forms) |
| agfront's `board.md` bullet on `agrefs` | agfront `shared/board.md` | 2 | S → replaced by the pyagag section |
| entrance: ✔ is finished; list every time, earlier answers are history (as10); close out only when asked; never `send` into this channel | autolab and forge `entrance_front`, pyagag `DEFAULT_GUIDE` | 4–6 | S → `DEFAULT_GUIDE` is already the shared text; forge's guide is deleted, autolab's keeps only its vocabulary lines and `mission_done` |
| "each serving ends; you are called back when they answer" | agfront `work.md`, autolab WR13, archsage AS16 | 2–8 | S → a pyagag section for delegating roles (step 3 decides the shape) |
| the board is Zulip, `agentchat --help` is the index, reading is free, a post runs its addressee | agfront `board.md`; autolab WR12 (one line); archsage has none | 2–10 | S → a pyagag `board` section for every conversational role that holds `agentchat` |
| "the first line is a Markdown heading and becomes the title, and the rest of the file becomes the description" | autolab WP7, forge FG5, cagent front | 2 | C — three parsers in three agents; reworded per role, not shared |
| "Your reply to this conversation will be sent to the developer" | autolab WP1 and BM1 | 1 | same agent; replaced by each head |
| "The file README_PROJECT.md explains how the folders …" | autolab WP2 and WR2 | 1 | same agent; each head says it once in its own words |

## Commands the guides send to a `--help`, or describe inline

For step 2. "inline" means the guide carries usage that a help should.

| command | guide(s) | pointed at `--help`? | what the guide carries inline |
|---|---|---|---|
| `archsage sage add/attach/update/sync --require`, `sage list` | archsage AS8, AS9, AS14 | no ("each tool's `--help`" in general) | sources, the re-post, the sagesync record and `includes=`/`missing=` |
| `archsage queue list/resolve` | archsage AS3, AS14 | no | `--answered-by` files must be in the tree |
| `archsage ask` | AS2 | no | — |
| `agroutine create/show/update` | AS7, AS9, AS13 | no | the 10 000-char bound, update with the whole guide |
| `agproject open/status` | AS6, AS9 | yes (p1 improved it) | the pending state |
| `sagetree ls/cat/grep/find/revision/queue` | sage SG3, SG4 | no | the command list, queue append semantics |
| `autolab doc patterns` | WP3 | no | — |
| `agag init` | WR4 | yes (`agag --help`) | the flags |
| `agautolab.mission_done` | EA5 | no | — |
| `agentchat channels/topics/read/resolve/send` | EA2–EA6, WR12, OO6 | once (WR12) | prefixes, ✔, `--count` |
| `agforge toolsets --list`, `knowledge list/show/search/path`, `image generate --init-image`, `video/music submit`, `comfy fetch` | FP1, FG2, FG8, FR2, FR4, FA2 | no | knowledge states, init-creativity range, submit/fetch |
| comfynotify `watch` | WR14 | no (it is a mention, not a CLI) | the command, two-line callback, public only |
| `agrefs` | all S rows | yes | the pointer |

Observer's `agobserver.withdraw` and health commands, named as candidates
in the plan, are not used by any guide: they are operator commands run
from this shell (devenv) or by Observer's own code. Step 2 checks their
helps anyway, since the plan lists them.

## Heads, per role

What step 4 gives each role as its head, decided here:

| role | converses with | head |
|---|---|---|
| archsage `archsage` | the requester in its channel (a person or another agent), or an argue | who; whom it speaks with; what is assumed (every sage, study, routine on the board is its to look up); what it can see (the trees placed above, `archsage`, `sagetree`, `agproject`, `agroutine`, `agentchat`, `agrefs`, one line each, `--help`) |
| archsage `sage` | the asker, through archsage's account | unchanged shape: one domain, one tree, `sagetree` only |
| autolab `superdirector` | the requester in `workplan-` (Front, a routine run, archsage via `agproject`, a person) | who; whom it speaks with; what it can see (the project folder, the board via `agentchat`, `tools/agents.md`, `agrefs`, `autolab doc`); then the plan contract |
| autolab `supercoder` | the requester in `workrun-` | who; the mission copy; what it can see; the result contract |
| autolab/forge entrance | whoever asks in the instance channel | pyagag's default, plus the agent's vocabulary |
| forge `front` (assetplan) | the requester in `assetplan-` | who; what it writes; what it can see (`agforge`, `agrefs`) |
| forge `generator` (plan, run) | nobody: files | who reads the files; what it may consult (`tools/`, `agforge knowledge`, `agrefs`); the contract |
| Observer `intake`, `observe`, `triage` | nobody: a program reads a JSON file | what it looks at; what it may write; who reads it. These are already written that way (OI1, OO1, OT1); they gain no board facts |
| argue participants (autolab, forge, Observer, cagent) | the argue | pyagag's `participant_guide()` is the shared head; `role.md` says only what the agent contributes |

## Tally

| agent | F | T (to a help) | S | A | C |
|---|---|---|---|---|---|
| archsage | AS1–AS5, AS10–AS12, AS15, AS18, AS19, AS22 | AS2, AS6–AS9, AS13, AS14 (parts) | AS16, AS17 (part), AS21 | **AS20** (duplicate of REPLY_GUIDE) | — |
| sage | SG1, SG2, SG5, SG6 | SG3, SG4 (parts) | — | — | — |
| agobserver | OI1, OI4, OI5, OO1, OO3, OO5, OO7–OO11, OT1, OT3–OT6, OA1, OA2 | OO6 (part) | OI6, OA3 | — | OI2, OI3, OI7, OO2, OO4, OT2, OT7 |
| autolab | WP1, WP2, WP4, WP6, WP8, WP10, WP12–WP14, WP16, WR1–WR3, WR5, WR6, WR8–WR11, WR16, BM1–BM3, AA1, AA2 | WP3, WR4, WR14 (part), EA5 (part) | WP15, WR12, WR13, WR15, EA1–EA3, EA5, EA6, AA3 | — (AF1 is a dead file, not a paragraph any role reads) | WP5, WP7, WP9, WP11, WR7, RV1–RV4 |
| forge | FP2, FP4, FG1, FG3, FR1, FR5, FR6, FR8, FA1, FA2 | FP1, FG2, FG8, FR2, FR4 (parts) | FP3, FG7, FR7, FA3, EF1, EF2 | — | FP1, FG4–FG6, FR3, FR4 |

No Evidence-Driven paragraph falls in class A. The one A row (AS20) is a
duplicate of the appended reply section, not a rule without evidence. The
Evidence-Driven paragraphs of these agents — WP6 (trend7), WP10 (fs3p),
WR6 (smoke, fs4r), WR10–WR11 (sub26, fs2T1), RV (fs4r), EA3 (as10), AS12
(fs5-1, fs5E), AS14 (fs5-1), OO5 (ob2), OT3 (rw3C), OT4 (rw3r), OT5 (fs1t) —
stay with their trial references wherever step 3 or 4 puts them.
