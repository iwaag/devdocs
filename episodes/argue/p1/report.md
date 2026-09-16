# argue p1 — final report

Date: 2026-09-16 JST. Source: [braindump.md](../braindump.md). Plan:
[plan.md](plan.md). Steps: [1](report1.md) the conversation contract ·
[2](report2.md) archsage and its sages · [3](report3.md) study planning or
project setup · [4](report4.md) deploy, demonstrate, report.

## Result

An **argue** exists: `#argue › argue-<stem>`, a conversation in which a
human develops a vague desire with every agent. Front opens it from an
ordinary conversation, is the only agent served automatically there, asks
until a human's own post is the desire on record, invites the agents whose
knowledge or capability would help by naming them, calls the knowledge
council, judges study-first against project, sets the outcome up with one
command, and finishes with an outcome the listener verifies against the
realm before resolving the argue. Every other agent — cagent, Observer,
autolab, forge and the new **archsage** — takes part when named and answers
in the same topic, on a shared participation guide that travels with
`pyagag` (`agag.argue`).

**archsage** replaces arxivsage: one deployed agent, one Zulip account,
many logical sages, each a directory holding a domain guide and the clone
of one study's published knowledge, run on a shared profile with one
bounded reader (`sagetree`); the council itself runs on Claude Fable 5.1,
reads every tree, asks a sage in-process and defines new ones. Addressing
is `@**archsage** sage:<name>`; a sage's reply carries its header; a
request for a sage costs no archsage run.

The live demonstration ran end to end in **ten minutes and $2.14**:
desire → archsage's analysis (which defined a new sage, `growbox`, with an
empty tree) → a direct sage answer and cagent's cluster reality, with one
invitation left outstanding across a listener restart and served by its
recovery → Front's judgement (project) → `#pj-desk-garden`, its goal, and
the workspace autolab prepared, all linking back to argue 7149 → the
outcome, the origin told, the argue resolved and quiet across a restart.
Nothing downstream was started.

## Delivered

| Repository | Commits | What |
|---|---|---|
| `iwaag/pyagag` | `92f6839` … `ac1262a` | `agag.argue` (the contract: invitations, selectors and speaker headers, outstanding-until-answered, the desire and argue notes, `participate`), the mention route judged by the newest unanswered mention above the served mark, `agentchat argue open`, per-invitation role context, `intro_text` extra placeholders, case-insensitive mentions, one reply per speaker per serving; 626 → 632 tests |
| `iwaag/agfront` | `d07d742`, `dbe0d6b`, `03d2682` … | the `argue` role and guide, `agproject`, outcome verification and resolve, the argue home on the mention route; 177 tests |
| `iwaag/agautolab`, `iwaag/agforge`, `pj-agdev/agobserver`, `pj-clusterintent/cagent` | step 1 commits + lock bumps | mention routes that answer an argue invitation only, read-only `argue` roles, role contexts, "In an argue" in every introduction |
| `iwaag/archsage` (new) | `6080fe9` … | the council, the sages, `sagetree`, the `archsage` CLI, launchd template, 28 tests |
| `iwaag/pj-agdev`, `iwaag/pj-clusterintent`, `iwaag/devdocs` | pins and reports | |

## Costs

| Where | Runs | Cost |
|---|---|---|
| The live argue (step 4) | 12 | $2.14 |
| The arxiv sage test run (step 2) | 1 | $0.17 |
| Frontier-model probes (step 2) | 2 | $0.26 |
| **Total** | | **$2.57** |

Per logical role in the argue: Front 8 runs ($0.73), archsage 1 ($0.76),
sage:growbox 1 ($0.10), cagent 1 ($0.18), autolab setup 1 ($0.37). One
council run for the whole conversation; the only waste was one Front
serving that announced an invitation instead of making it.

## Limitations and open items

- **Human authorship was checked mechanically, not proven.** The demo's
  human turns were posted with the Developer credential by the Omni Agent;
  the human-only checks rest on fixtures (step 1). A real human argue is the
  first thing to run next.
- **The sage boundary is a bounded reader, not a sandbox** (step 2): a
  compound shell command escapes a claude_code `Bash(sagetree:*)` grant,
  visibly in the transcript. No container or OS isolation was added.
- **A new study gets its channel, plan and workspace, not its routine**;
  the `#routine-study-<x>` guide stays human-written and is named as next
  work. The `study` and `plan` outcome branches are pinned by tests only;
  the live branch was `project`.
- **The growbox sage has no study repository**: an empty tree by design
  until the owner creates one and attaches it with `archsage sage sync`.
  The sage queued the project's question in its `tostudy/`.
- **autolab's first serving of a new workspace lays out `direction/` and
  `devlog/` before it reads the request** (finding 5 in report4). A
  pattern marker or a pattern named in the setup request would avoid it.
- **Front's judgement of study versus project is one live sample**, and
  it deferred to the human once before deciding. Whether it chooses study
  first for an exploratory desire without prompting is untested live.
- **Chains of argues** (a later argue picking up a study's results) and
  automatic study-to-sage refresh are outside p1, as planned.
- The Front Desk voice (`character_talk`) can open an argue but has not
  been tried; the Observer and forge mention routes were wired and tested
  but not invited live.
