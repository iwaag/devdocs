# agent_guide p1 — preresearch

Facts gathered on 2026-09-28 by the Omni Agent, from one Front Desk failure
and a read of the guides behind it. This file records what happened, what the
guides look like now and what an implementer may find useful. It is not a
plan.

## 1. The incident

`#front › ✔ front-desk-20260928-180233`, served by agfront's `desk` role
(claude_code, `anthropic/claude-sonnet-5`).

| post | who | what |
|---|---|---|
| #15673 | Developer | "Is the AISVGs research sufficient — major researchers, individuals, the community's state of the art — to invent new ideas without reinventing the wheel?" |
| #15675 | Front (run-0160, 2 turns, 20 s) | "I find no record of any AISVGs research in this conversation or my memory. Tell me the channel, or whether it has not started." |
| #15679 | Developer | "Isn't there something like study-aisvgs in routine?" |
| #15681 | Front (run-0161, 16 turns) | Correct and complete: `routine-study-aisvgs`, `#pj-aisvgs`, rounds 1 and 2 (m11579, m11741) accepted, `sage:aisvgs` refreshed to `f57eed1a27de`, remaining gaps. |

Everything Front needed existed before #15673 and was readable with the tools
it already had (`agentchat`, granted in `[roles.desk]`).

What run-0160 did (its transcript, generation 1 of the topic workspace):

1. One Bash call: `ls` of its own workspace and `find` over the topic's
   parent directory. The `find` was refused — outside the session's allowed
   directory.
2. It replied. It never ran `agentchat`, and never read `tools/agents.md`,
   which was in its workspace.

What run-0161 did: `agentchat channels --prefix routine-` first, then
`agentchat read routine-study-aisvgs guide`, `agentchat topics …`, the two
`routinerun-` topics, `#pj-aisvgs`, archsage's topics and the round-2
`workrun-`. The Developer's word "routine" matched the one paragraph of the
guide that names a discovery command.

## 2. Why the first run did not look

### 2.1 The guide hands out tools only inside procedures

The desk guide mentions each discovery command only as a step of some
procedure, under a condition:

| command | what it shows | where the desk guide mentions it |
|---|---|---|
| `agentchat intro [<agent>]` | every agent and its contract | nowhere (a pre-rendered `tools/agents.md` instead) |
| `agentchat channels --prefix pj-` | every project and study | nowhere |
| `agentchat channels --prefix routine-` | every routine | only in "Asked to run a routine…" |
| `agentchat topics <channel>` | a channel's conversations, ✔ state | only for argue and routine stems |
| `agentchat read <channel> <topic>` | one conversation | only under "Evidence" for threads Front itself opened |
| `agproject status <slug>` | where a project/study stands | nowhere (only `open`) |
| `tools/` | other agents' intros, Front's runs | only "If the developer asks for work" |

The first branch of "What to do in one run" is: *"If the last message is
small talk or something plain text answers, just reply."* A question about
the state of existing work matches no other branch, so the run took this one.
Read literally, it permits answering without looking.

### 2.2 The guide never says what the developer assumes

Nothing states that the developer speaks to Front as someone who knows every
project, study, routine, sage and agent on the board. Without that, an
unknown name reads as the developer's missing information, not Front's.
run-0160 asked the developer for the channel name.

### 2.3 The one listing it had was stale

`tools/agents.md` (built by `agag.intro.write_agents_md` from the agents'
introductions in `#agents`) did list the sage:

> `sage:aisvgs` — creating SVG images with AI: … *(study `aisvgs`: no findings yet)*

The "no findings yet" label is wrong. archsage renders it in
`archsage/src/archsage/intro.py` (`sages_lines`) only when it posts its
introduction. It posts that introduction at start-up and after a sage
*definition* change (`_after_change` in `archsage/src/archsage/cli.py`: define,
update, attach, remove), but not after `sage sync`. The last post was #11668,
2026-09-26 21:47 JST, before either round's findings reached the tree.
Rendered now, the same function prints `*(study `aisvgs`)*`. So even a run
that read `tools/agents.md` could have concluded there was nothing yet.

### 2.4 A refused first probe ended the search

The run's only probe was a filesystem `find`, which the harness refused.
The guide says nothing about where the board lives (Zulip, through
`agentchat`) as opposed to the workspace, so after that refusal the run had
nowhere else to look.

## 3. The guides against Tool Giving

`devpolicy/terms.md`: **Tool Giving** is giving tools *and sufficient usage
information*, with minimum shackle. **Tool Implantation**, **Shackle** and
**Anxiety-Driven Guidance** are the failure modes.

The Developer's reading of this incident (2026-09-28), which this phase
starts from:

- A guide should be **short**.
- It should give tools, not name the situations they are for. A tool is
  given by saying *what it tells you* and *how to get its help*: a
  top-level `--help` that is enough on its own, or, for a broad tool like
  `agentchat`, per-subcommand `--help` — the guide says how to call it and
  what information each part yields.
- It should state the facts the agent works under, briefly, such as "the
  developer assumes you know the projects and the other agents".
- It should not transcribe subcommands at length.

### 3.1 Size and growth

Guide lengths now (lines):

| guide | lines |
|---|---|
| agfront `desk` | 367 |
| agfront `front` | 363 |
| agfront `routine_run` | 284 |
| archsage `archsage` | 198 |
| agfront `argue` | 191 |
| agautolab `project_pattern.md` | 184 |
| agautolab `workrun_supercoder` / `workplan_superdirector` | 84 / 79 |
| agobserver `observe` / `intake` / `triage` | 82 / 67 / 41 |
| agforge `assetrun_generator` / `assetplan_front` | 61 / 29 |
| archsage `sage` | 34 |
| agfront `present` | 54 |
| agautolab `entrance_front` / `bmining_director` / `autolab-front` | 13 / 8 / 3 |
| agforge `entrance_front` | 11 |

The shortest guides belong to the agents that are given CLIs (autolab,
forge). The long ones are Front's.

The desk guide's history (`agfront`, `agent/guides/desk/guide.md`, earlier
`character_talk/guide.md`):

```
2026-09-08   71  → 161   front_desk p1–p2 (mostly "seen live" prohibitions)
2026-09-16  190          argue p1
2026-09-18  132          argue p2 (character moved out)
2026-09-26  154 → 177    give_context_easier, sage p2
2026-09-27  217 → 295    failsafe p1–p5 (10 commits)
2026-09-28  315 → 367    failsafe p6 (5 commits)
```

It grew by 213 lines in three days, and every addition was a new paragraph
describing a procedure: receipts, holds, dispositions, relations, `agrun`,
recheck, Observer stops. Each was evidence-driven when written. Together they
bury the few sentences that tell Front what it can see. `desk` and `front`
differ in 252 lines of `diff` output but carry the same procedures, so every
addition is written twice.

The desk prompt of run-0160 was 27 730 characters. By section:

| section | chars |
|---|---|
| conversation + continuation | 2 575 |
| What to do in one run (incl. argue, project, study, routine, accept, agrun) | 7 692 |
| When Observer says work has stopped | 4 395 |
| How your reply is posted (appended by `agag.reply`) | 3 303 |
| Each run ends; the conversation does not | 2 642 |
| Human-authored references | 1 387 |
| Evidence | 1 185 |
| When an answer reads as not taken up | 1 150 |
| When a request is ended… | 986 |
| Carrying the conversation forward (appended by `agag.continuation`) | 922 |
| When a person keeps a decision… | 884 |
| Last thing | 194 |

The only part written the Tool Giving way is **Human-authored references**:
it says what `agrefs list`, `sync`, `show` and `path` each give you and
points at `agrefs --help`.

### 3.2 The help texts are uneven

Help that already reads as Tool Giving:

- `agentchat --help` opens with what Zulip is and a list of examples, each
  under a question it answers: "Who else is there, and what does each of them
  do?" → `agentchat intro`; "Which channels are there…?" → `agentchat
  channels --prefix`. But it is 237 lines long, so it is a manual rather than
  an index.
- `agentchat <sub> --help` exists for every subcommand and states what the
  output is (`channels`: "every public channel … as `<name> — <description>`";
  `topics`: "most recently active first … '✔' is … resolved"; `intro`: "an
  introduction is the agent's contract…"; `trace`: what each state means).
- `agrun --help` and `agproject --help` explain their purpose in their first
  lines.

Gaps:

- `agproject status --help` has no description at all, only usage and
  options, so it does not say what "status" reports.
- There is no command that lists projects or studies as such. Discovery is
  `agentchat channels --prefix pj-`, and only `agentchat --help` shows that
  pattern in general terms.
- No help says that the routine, project and study channels, the sages
  (through `agentchat intro archsage`) and the other agents' topics are, taken
  together, "the board", and that all of it is readable at no cost to anyone.

## 4. Suggestions for the implementer

These are observations, not decisions.

1. **Open each conversational guide with what the agent can see and what is
   assumed of it**, a few lines long. For Front, roughly:
   - The developer speaks to you as someone who knows every project, study,
     routine, sage and agent. When a name is new to you, it is yours to look
     up.
   - `agentchat --help` lists what you can see; each `agentchat <sub>
     --help` says what that part returns. Reading costs nobody anything.
   - One line each for `agproject --help`, `agrun --help` and `agrefs --help`:
     what the tool is for.
2. **Move procedure out of the guide and into help.** Most of the failsafe
   paragraphs describe one `agentchat` subcommand (`receipt`, `hold`,
   `release`, `disposition`, `relation`, `recheck`, `accept`, `reserve`) or
   `agrun`. The subcommand's `--help` is the natural home. The guide keeps
   only the facts that no help can hold, such as who holds an acceptance or
   what a proxy's authority means.
3. **Drop the "just reply" branch**, or make it about small talk alone.
   Remove discovery commands buried in procedures once item 1 exists.
4. **Write shared text once.** `desk` and `front` (and parts of `argue`,
   `routine_run`) could share a common file, the way `agag.reply` and
   `agag.continuation` already append shared sections.
5. **Fix the stale sage label** separately (archsage, not guides): re-post
   the introduction after `sage sync`, or drop the findings state from the
   introduction so it cannot go stale.
6. **Give `agproject status` a description** of what it reports.
7. **Measure before and after.** The incident above is a ready trial: the
   same #15673 wording in a fresh desk topic, pass = Front finds
   `routine-study-aisvgs` / `#pj-aisvgs` in its first reply. Other cheap
   probes: a question naming a project only by a loose name ("the grow box
   thing"), and one naming an agent's past work without a channel.
8. **Watch what the shortening removes.** Many desk paragraphs were added
   after a seen-live failure (commit subjects say so: "seen live", "trial
   A2", "trial E"). failsafe p1–p6's reports name the trial behind each. A
   paragraph that moves into `--help` should still be reachable at the moment
   it is needed. Check each against its trial rather than deleting by length.

## 5. Where to look

- The incident: `#front › ✔ front-desk-20260928-180233` (#15673–#15716).
  Run records are `agfront/.local/agent/desk/run-0160.json` and
  `run-0161.json`. Transcripts are in the harness's project store, under the
  topic workspace names `…front-desk-20260928-180233-1-desk` and
  `…-2-desk`.
- Guides: `pj-agdev/agfront/agent/guides/{desk,front,argue,routine_run,present}/guide.md`;
  roles and grants in `pj-agdev/agfront/agents.toml`.
- Shared prompt sections: `pyagag` `agag.reply` ("How your reply is posted"),
  `agag.continuation` ("Carrying the conversation forward"),
  `agag.intro.write_agents_md` (`tools/agents.md`).
- Help texts: `agentchat` (pyagag), `agproject`, `agrun`, `agrefs`.
- Stale sage label: `archsage/src/archsage/intro.py` (`sages_lines`),
  `archsage/src/archsage/cli.py` (`_after_change`, `sync`).
- Terms: `devpolicy/terms.md` (Tool Giving, Tool Implantation, Shackle,
  Anxiety-Driven / Evidence-Driven Guidance, Failure Farming).
