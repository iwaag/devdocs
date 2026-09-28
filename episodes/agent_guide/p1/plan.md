# agent_guide p1 — Tool Giving guides, one source per fact

## Goal and scope

Rewrite the conversational guides so that an agent is told what it can see
and what is assumed of it, and is handed its tools by pointing at their
`--help`, instead of being walked through procedures under conditions. Make
each fact live in one place (a `--help`, one shared file, or a generated
section) and let the guides shrink as a consequence, not as a target.

Read `braindump.md` and `preresearch.md` beside this file first. The
incident there (`#front › ✔ front-desk-20260928-180233`, run-0160) is the
trial this phase is measured by. `devpolicy/terms.md` defines Tool Giving,
Shackle, Anxiety-Driven and Evidence-Driven Guidance.

This is a private experimental, breaking-change phase: backward
compatibility is unnecessary. Implementers choose file layout, section
names, help wording and refactoring scope. One rule is fixed by the
Developer: **no Evidence-Driven paragraph is deleted.** A paragraph that was
written after a seen-live failure or a named trial may move (into a
`--help`, a shared file, a generated section) but its content must still
reach the agent at the moment it is needed. Everything else is discretion.

Primary subject: agfront's `desk`, `front`, `argue`, `routine_run` guides
(`pj-agdev/agfront/agent/guides/<role>/guide.md`). Secondary: the help texts
they point at (`agentchat` in pyagag, `agproject`, `agrun`, `agrefs`), and
the same duplicated paragraphs in agautolab, agforge and agobserver guides.
`present` and the entrance guides are already short; leave them unless a
shared file makes them shorter for free.

Where guides come from: `agfront/src/agfront/zulip_listener.py` pairs a
role with its guide and calls `prompt_with_guide` (`pyagag/src/agag/topics.py`),
which appends `REPLY_GUIDE` (`agag/reply.py`, "How your reply is posted")
and `CONTINUATION_GUIDE` (`agag/continuation.py`, "Carrying the conversation
forward"). That is the existing mechanism for text every conversational
role receives once; `tools/agents.md` (`agag/intro.py`
`write_agents_md`) is the existing mechanism for a generated file in the
workspace. Nothing in `agents.toml` includes one guide from another today.

## step1 — inventory: where does each paragraph come from, and where is it going

- Split the desk guide into paragraphs and, for each, record: the commit
  that added it (`git log -L` or blame in `pj-agdev`), the episode/trial it
  cites (commit subjects say "seen live", "trial A2", "trial E", "failsafe
  p6 step 3"), which other guides carry the same text (desk/front share 223
  non-empty lines; the `agrefs` paragraph is in 8 guides across 4 agents;
  the `receipt` paragraph in 3), and which `agentchat`/`agrun`/`agproject`
  subcommand it describes, if any.
- Classify each: **fact the agent works under** (stays in the guide, short),
  **tool usage** (goes to that tool's `--help`), **shared across roles**
  (goes to one file or an appended section), **Anxiety-Driven** (no trial
  behind it — may be dropped, list it so the report can say what went).
- Do the same, faster, for `front`, `routine_run`, `argue`, and for the
  duplicated paragraphs in the other agents' guides.
- Write the inventory as `report1.md`: one table, one row per paragraph.
  It is the evidence for every later deletion.

Hints: the failsafe p1–p6 `report*.md` name the trial behind each
paragraph and what it fixed. `devdocs/README_DEV.md` carries the
system-level version of many of them. The desk guide's own sections are
listed in `preresearch.md` §3.1 with their prompt sizes.

## step2 — make the help texts a sufficient home

- For every subcommand the inventory sends to a `--help`, check that the
  help says three things: what the command reports or changes, what the
  output means, and when an agent would want it. Add what is missing. Known
  gaps: `agproject status --help` has no description at all; nothing lists
  projects/studies except `agentchat channels --prefix pj-`; no help says
  that routine/project/study channels, the sages (`agentchat intro
  archsage`) and other agents' topics together are "the board", readable at
  no cost.
- Turn `agentchat --help` (237 lines today) into an index: one line per
  subcommand with what it yields, a few lines of orientation (what Zulip is
  here, that reading is free, that a post costs the addressee a run), and a
  pointer to `agentchat <sub> --help`. Move the long explanations into the
  subcommand helps. Consider a `--help` for the `--intent` vocabulary if it
  does not fit anywhere else.
- Moved paragraphs keep their wording where the wording carried a trial's
  lesson; help text may be plainer than a guide, but not vaguer.
- Where a fact belongs to no command (who holds an acceptance, what a
  proxy's authority means, the developer's assumptions), leave it for step3.

Hints: pyagag's `agentchat` help is assembled in its CLI module; each
subparser has its own description, so per-subcommand help is a text edit,
not a design. `agrun --help` and `agproject --help` already open with their
purpose and are a fair model. Check `agrefs --help` too: the desk guide's
"Human-authored references" section is the one already written the Tool
Giving way, so it should become one or two lines pointing at it.

## step3 — write the shared text once

- Decide the mechanism for text shared by desk, front, argue and
  routine_run: either a shared guide file the listener concatenates
  (extend `guide()` / `prompt_with_guide`), or a new appended constant next
  to `REPLY_GUIDE`. Prefer whichever keeps the text in a `.md` file a human
  edits, not a Python string. Do the same for the `agrefs` and receipt
  paragraphs in autolab/forge/observer, or replace them with one line
  pointing at `agrefs --help` / `agentchat receipt --help` once step2 makes
  that sufficient.
- Do not add a second copy of anything that is already generated
  (`tools/agents.md`, the reply and continuation sections).
- Decide whether `desk` and `front` remain two guides. They share 223 lines
  and the same procedures; the difference is the audience (developer
  directly vs. the ordinary entrance) and the character rule. A single
  shared body plus a short per-role head is the expected outcome, but the
  implementer decides.

## step4 — rewrite the guides

- Open every conversational guide with a short head, roughly:
  1. who you are and who you are speaking with;
  2. what is assumed of you: "the developer speaks to you as someone who
     knows every project, study, routine, sage and agent on the board; a
     name new to you is yours to look up, not the developer's to explain";
  3. what you can see and how: `agentchat --help` lists it; `agentchat
     <sub> --help` says what each part returns; reading costs nobody
     anything; one line each for `agproject`, `agrun`, `agrefs`, `agbudget`
     with what the tool is for;
  4. the few facts no help can hold (authority of a proxy speaker, no
     `@**mentions**` in `#front`, this conversation is the record).
- Then the body: the Evidence-Driven facts the inventory kept, each as a
  fact, not as a branch. Where a paragraph used to say "if X, do `cmd`",
  keep the fact ("X means the work has stopped; only the owner's post in
  its own conversation shows it moving again") and let the head's tools do
  the rest.
- Drop the "if it is small talk, just reply" branch, or restrict it to
  greetings. It is the branch run-0160 took.
- Remove discovery commands that only appear inside procedures once the
  head lists them.
- Expected size: the desk prompt is 27 730 characters today. Halving it is
  plausible; do not aim at a number.

Hints: the guide is read by `claude-sonnet-5` under `claude_code`; the
harness refuses reads outside the topic workspace (run-0160's `find`), so
the head should say plainly that the board lives in Zulip and is reached by
`agentchat`, not by the filesystem. `tools/` is the workspace's own copy of
introductions and Front's runs; say what is there in one line.

## step5 — the stale sage label (archsage, separate change)

- `tools/agents.md` said `sage:aisvgs — … (study aisvgs: no findings yet)`
  after both rounds had been accepted. archsage renders that line in
  `archsage/src/archsage/intro.py` (`sages_lines`) and re-posts its
  introduction only from `_after_change` in `archsage/src/archsage/cli.py`
  (define/update/attach/remove), not after `sage sync`. Either re-post after
  `sync`, or drop the findings state from the introduction so it cannot go
  stale. Keep it small; it is not a guide change but it broke the one
  listing run-0160 could have read.

## step6 — trial and roll-out

- Before deploying, run the same #15673 wording ("Is the AISVGs research
  sufficient — major researchers, individuals, the community's state of the
  art — to invent new ideas without reinventing the wheel?") in a fresh
  `front-desk-` topic against the old guide once more if no record of a
  clean failure exists beyond run-0160; then deploy and run it again. Pass:
  Front names `routine-study-aisvgs` or `#pj-aisvgs` in its first reply
  without being told the channel.
- Two more cheap probes: a project named loosely ("the grow box thing"),
  and an agent's past work named without a channel ("what did forge deliver
  for protoprey?"). Pass is the same: found on the first reply.
- Re-run the failsafe probes whose paragraphs moved (the p3 "stopped work"
  trial and the p6 receipt/hold trials are the largest); their reports say
  how they were driven. A moved paragraph passes if the behaviour it bought
  is still there.
- Restart agfront's listener after the guide change (`agag` kickstart as
  the devenv notes describe; guides are read per serving, but the help
  texts are in installed packages — check `uv sync` in the agent projects
  that depend on pyagag). Check the run record
  (`agfront/.local/agent/desk/run-NNNN.json`) for turns and cost; the
  before/after prompt size is in the transcript.
- Run the affected test suites (`pj-agdev/agfront/tests`, pyagag's tests for
  the CLI help and prompt assembly). Read the summary line, not the tail.

## step7 — report

- `report.md`: what moved where (from the step1 inventory), guide and prompt
  sizes before/after, the trial results with run numbers, and the list of
  paragraphs classified Anxiety-Driven and dropped, so a later phase can
  restore one if a failure returns. Note anything done for an in-system
  agent by hand as a handoff candidate.
- Update `devdocs/README_DEV.md` where it describes the guide layout, and
  `devpolicy/styles.md` if a guide-writing convention came out of this
  (head/body shape, "one fact, one home") — one paragraph, not a rulebook.
