# agent_guide p2 — the other agents' guides, cross-agent sharing, and trials we can repeat

## Goal and scope

p1 rewrote Front's four conversational roles the Tool Giving way. p2 does
the same for the other agents, writes text that several agents share once
for the whole system, removes the frictions p1's trials hit, and builds the
fixture that makes a before/after comparison of a guide change repeatable.

Read `../p1/report.md` first, then `../p1/report1.md` (the per-paragraph
inventory method) and `../p1/report6.md` (trials). `devpolicy/terms.md`
defines Tool Giving, Shackle, Anxiety-Driven and Evidence-Driven Guidance.
`devpolicy/styles.md` § *Agent guides* and `devdocs/README_DEV.md` § *How a
guide is put together* are p1's conventions.

This is a private experimental, breaking-change phase: backward
compatibility is unnecessary. Implementers choose layout, wording,
command names and refactoring scope. Fixed rules, both carried from p1:

- **No Evidence-Driven paragraph is deleted.** It may move, but its content
  must still reach the agent at the moment it is needed.
- **Length is not a target.** A guide that is long because it carries facts
  its role needs stays long. Cut only what is duplicated, transcribed from
  a `--help`, or Anxiety-Driven. routine_run (~280 lines served) and argue
  are in scope only for that kind of waste.

Out of scope, held by the Developer: where a decision about a study is
recorded (p1 open finding 1), and the agautolab1 VM's pyagag version.

## step1 — inventory the other agents' guides

Use p1's method (`../p1/report1.md`): one row per paragraph, with the
commit that added it, the trial or episode it cites, other guides carrying
the same text, the command it describes, and a class (fact / tool usage /
shared / Anxiety-Driven).

| agent | guides | lines | `--help` mentions |
|---|---|---|---|
| archsage | `archsage/agent/guides/{archsage,sage}/guide.md` | 189, 34 | 2 |
| agobserver | `agent/guides/{observe,intake,triage}/guide.md` | 82, 67, 41 | 0 |
| agautolab | `workrun_supercoder` (+`review.md`), `workplan_superdirector`, `entrance_front`, `bmining_director`, `autolab-front`, `argue/role.md` | 75, 70, … | 3, 1, 0 |
| agforge | `assetrun_generator`, `assetplan_front`, `assetplan_generator/guide_plan.md`, `entrance_front`, `argue/role.md` | 52, 20, 30, … | — |
| agfront | `routine_run`, `argue` (waste only) | 121, 159 | — |

Write it as `report1.md`.

Hints:

- Every agent composes its prompt with pyagag's `prompt_with_guide` and
  reads guides through `shared_guide` (`pyagag/src/agag/topics.py`):
  agautolab `zulip_listener.py`, agforge `assetplan_topic.py` /
  `assetrun_topic.py` / `entrance_topic.py`, agobserver `intake.py` /
  `observe.py`, archsage `listener.py` / `roles.py`. The mechanism is the
  same everywhere, so step3 changes one place.
- archsage's guide collected paragraphs through failsafe p5 (commit
  subjects "trial E", "sage sync --require"). Its history is in the
  archsage repo, not pj-agdev.
- Observer's guides are read by roles that act on schedule, not on a
  developer's word. Their "head" is probably different from Front's: what
  the role watches, what it may say, whom it reports to. Decide per role.
- Structured-output roles (a generator answering with files) keep their
  output contract. Tool Giving applies to what they may consult, not to the
  shape of their answer.

## step2 — help texts for the tools those guides use

- For every command the step1 inventory sends to a `--help`, make the help
  say what the command reports or changes, what the output means, and when
  an agent would want it. p1 did this for `agentchat`, `agproject`, `agrun`,
  `agrefs`, `agbudget`. Candidates now: archsage's `sage` commands (`sage
  sync --require`, define/update/attach), autolab's and forge's own CLIs,
  Observer's `agobserver.withdraw` and health commands, `comfynotify` if a
  guide describes it.
- Keep the p1 lesson in the wording: a fact about *recording* something
  also needs the sentence that saying it is not recording it.

## step3 — text shared across agents, written once

- p1's sharing is agfront-local (`SHARED_GUIDES` and
  `agfront/agent/guides/shared/`). The agrefs pointer is still copied
  verbatim into 7 guides across 4 agents ("`agrefs` reads what the
  developer has published for agents to build from — …", 5 lines). Move it,
  and whatever else step1 finds shared, into pyagag, next to `REPLY_GUIDE`
  and `CONTINUATION_GUIDE`.
- Suggested shape: `.md` files shipped as package data in pyagag, appended
  by `prompt_with_guide` through a keyword (for example `refs=True`, or a
  list of shared section names), chosen per role by the agent's code or
  `agents.toml`. Prefer text in `.md` files over Python strings. The
  implementer decides.
- Consider whether agfront's `shared/board.md` belongs in pyagag too: every
  conversational agent reads the board through `agentchat`, and archsage and
  autolab's director roles may lack the same "the board is Zulip, reading
  is free" facts. Grant differences (who has `agproject`, `agrun`) stay in
  the agent's own files, as p1 did with `requests.md`.
- After this step, `find … -path '*/agent/guides/*' -name '*.md' | xargs
  cat | sort | uniq -c` over lines longer than 40 characters should show
  no paragraph repeated across agents.

## step4 — rewrite the guides

- Give each conversational or delegating role the p1 head: who it is and
  whom it speaks with; what is assumed of it; what it can see and how
  (`agentchat --help` and the tools it is granted, one line each); the few
  facts no help can hold.
- Turn procedure branches into facts where a trial stands behind them;
  send tool usage to the help texts from step2.
- routine_run and argue: remove only what is duplicated or transcribed.
  p1 judged their length reasonable; if step1 agrees, say so in the report
  and leave them.
- Grants: a tool a guide now points at must be in the role's
  `allowed_tools`, or the run blocks on a permission prompt
  (`agents.toml`, each agent). A new CLI also needs its script entry and a
  listener restart.

## step5 — frictions p1's trials hit

- **Reading several topics in one call.** The Claude Code harness refuses
  shell loops in Front's runs ("Contains simple_expansion"); runs 0170 and
  0171 each lost turns to it. Give `agentchat read` several topics, or a
  channel's latest topics, in one call. `pyagag/src/agag/chat.py` holds the
  parser (`topics` at line ~924) and dispatch (~1540). Reads should come
  from the mirror, as the other reads do.
- **`agentchat topics --prefix`** was guessed by run-0170 and does not
  exist. Either add it (filter topics by name prefix) or make the error
  suggest the right command. Adding it matches how agents think of
  `workplan-` / `routinerun-` / `argue-` stems.
- Mention both in `agentchat --help`'s index; no guide text needed.

## step6 — a fixture board for repeatable trials

- p1's before-trial did not reproduce run-0160, because the live board had
  changed. Build a synthetic realm that stands in for the board a guide
  trial reads: a few projects/studies (`pj-*`), a routine with runs, an
  archsage topic, other agents' introductions, and one ✔'d past request.
- Realm exports cannot be pushed (the classifier refuses mirror-exported
  messages), so build it in code. `pyagag/tests/study_realm.py` and
  `project_realm.py` are the model: a read-only client stand-in over rows,
  with `cut(id)` for the realm as of a message.
- The fixture must be something a real run can read. Options: point a
  trial `agentchat` at a mirror store built from the fixture (the mirror is
  `agag.mirror`, a sqlite store), or seed a disposable realm. The
  implementer chooses; the requirement is that the same question against
  the same board can be asked with the old guide and with the new one.
- Encode p1's three board probes against it (the #15673 wording, "the grow
  box thing", forge's past work for protoprey) and a pass rule each.

## step7 — trials

- Run the fixture probes for Front with p1's guides (baseline) and for each
  rewritten agent where a probe makes sense: archsage asked about a sage by
  a loose name, an autolab director asked about another project's state,
  Observer asked by triage about a request it did not open.
- **Re-test the moved paragraphs p1 did not.** From `../p1/report6.md`
  § *Not re-run*: receipt repair when Observer asks about an owed answer
  (p6 `exit-before-receipt` fault), and a live hold placed and released
  (`agentchat hold` / `release`). Then every paragraph step1–step4 moves in
  this phase that has a trial behind it: re-run that trial, or state why a
  fixture probe covers it.
- Deploy the way p1 did (README_DEV *How a guide is put together*): code
  and help on a branch, merged in one go, then the pins, `uv sync` in each
  consumer, and the listener restarts. Guides are read from disk per
  serving; help texts are in installed packages. Check the running
  services through Nautobot or `nctl status` (`pj-clusterintent/nctl`).
- Run every consumer's test suite after the pyagag pin: pyagag, agfront,
  agautolab, agforge, agobserver, archsage, cagent, agentroom. Read the
  summary line, not a piped tail.

## step8 — report

- `report.md`: what moved where, sizes before/after per role (and why a
  role did not shrink, where it did not), the fixture and how to run it,
  the trial results with run numbers and cost, the paragraphs dropped as
  Anxiety-Driven with the commit to restore them from, and handoff
  candidates for anything Omni did for an in-system agent.
- Update README_DEV *How a guide is put together* for the cross-agent
  shared sections, and `devpolicy/styles.md` § *Agent guides* only if a
  convention changed.
