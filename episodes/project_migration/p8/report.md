# project_migration p8 — Front as a managed project: report

Steps:
[report1](report1.md) inspection and design ·
[report2](report2.md) operator capabilities ·
[report3](report3.md) conversation continuity ·
[report4](report4.md) the Front project ·
[report5](report5.md) deployment and migration ·
[report6](report6.md) trials.

Machine details, raw evidence and the recovery procedure are in pj-agdev's
ignored `.local/project-migration-p8/notes.md` and `.local/devenv.md`.

## Finding

**Front is now the Developer's conversational entrance, as an ordinary
managed project.** One message at the Front Desk can produce any of these
through the common WFE facilities, with no separate executor and no special
project type:
- an answer from records;
- delegated work across two projects, combined into one result;
- a steer of work in progress;
- a new project with its first result;
- an edit of context or workflows.

The legacy Zulip listener is retired.

## Delivered behaviour

- **One continuing conversation, many bounded runs.** WFE keeps the conversation:
  - the person's words verbatim, with submit tokens;
  - replies that name their run and what they answer;
  - links that record Front's reading of each message, superseded rather than rewritten.
  
  Each message starts one `front/converse` run by receipt. Concurrent work is told apart by run, never by recency.
- **Front decides, the facility routes.**
  - Front's turn reads the conversation and linked runs. It then answers, grows its own run (delegate nodes added by adjustment), steers other runs, answers specialists, or asks.
  - Its guide states facts and keeps the evidence-backed lessons with their trial references. Tool usage lives in `--help`.
- **Operator authority through common operations.**
  - From its attempt, Front records as itself on any run (`operator` provenance); the Developer's words stay theirs (`--from`, `--source-message`).
  - Agent requests are queued idempotently (`wfe request --request`).
  - Definition edits are base-checked against stale writes (`wfe workflow|project show|save`; the editor's saves too).
  - An instruction continues a run nobody executes.
  - The raw ops route no longer records delegations without their checks.
  - No permission framework and no approval gate was added.
- **Responsiveness.** A `conversation` lane on the coordinator's worker gives turns a slot of their own. A question was answered in 18 s while a cagent child held the default slot.
- **Knowledge.**
  - Preferences live in context blocks: system and per project `developer-preferences`, with evidence.
  - Lessons live in Front's `knowledge/`.
  - Run state stays in run records.
  
  A preference stated once shaped later reports. A project preference set through Front reached forge's own run (it delivered SVG without being asked in the request).
- **Delivery.** Replies carry links the Developer opens directly: run views and raw file links where images and sound play.

## Deployment and migration

| Component | Revision |
| --- | --- |
| WFE (service, `wfe`, site, relay) | agdevworld `248726b` |
| Executor (lanes) | agautolab `3ba5abc` |
| Front project | pj-front `main` (guide `cbac8cb` and Front's own integrations) |
| agfront retirement introduction | `befc06d` |
| pj-agdev pins | `870d15e` |

- Configuration: studio's `conversation` lane, `conversations.json` (entrance and person), the Front intake grant removed.
- Stores were backed up before the first release. Exec store v4 means rollback is code *and* data (notes).
- Legacy Front: listener booted out, plists archived, retirement introduction ✔, Nautobot service/agent retired and placement disabled. A fresh observation shows the host converged.
- Routines, argues and re-voicing are retired with explicit outcomes (report5). Their history and the agfront records stay readable. Nothing was in flight to migrate, and no preferences existed to move.

## Trial evidence (report6)

| Trial | Result |
| --- | --- |
| Existing information | answered from playprey's records, 50 s |
| Two-project request | forge + agsfx kit combined, chosen and delivered with links |
| Mid-flight change | "lower, softer chime" reached agsfx through Front's run; original and revision kept |
| Material ambiguity | an undefined term was asked about, not guessed; the answer was applied |
| New project | `glossary` created through `wfe create`, its first run integrated |
| Responsiveness | 18 s reply during a cagent child; no misattribution |
| Direct operator work | forge context block and glossary workflow changed; visible through the browser API |
| Knowledge reuse | system preference followed; forge preference honoured by forge directly |
| Specialist question | Front accepted forge's result itself, with recorded reasons |
| Recovery | executor and service restarted mid-turn: one run, one reply; replayed token deduplicated |
| Delivery | raw links to the PNG and WAV; the Front Desk shows the replies with run chips |

Defects found by real use and fixed:
1. A guide reference that sent Front searching the disk.
2. A question kept inside the run.
3. Ready work beside a wait read as an interruption. Fixing it changed record replay, so the fix was gated per entry, and one record written in between was repaired with the Developer's permission.
4. No openable file links.

Cost of the trials: about $6.7, of which Front's own turns were $3.87.

## Interventions

Omni:
- posted and answered as the person, under its own name;
- killed one runaway `find` (a guide defect);
- repaired one record broken by a WFE regression;
- restarted services for the recovery trial;
- deployed.

Deployment and the record repair were done after the Developer granted
permission; the session classifier had refused them. Omni did no work that
was Front's: it did not answer a specialist, dispose a delegation, create a
project or write a reply.

## Review entry points

- Front Desk: `http://agstudio.local:8093/?view=frontdesk&conv=p8-trial-b` (also `p8-trial-a` … `p8-trial-h`), or a new conversation from the same page.
- Front's runs: `http://agstudio.local:8093/wfe/#/ws/front`.
- The kit:
  - bell `…/wfe/api/projects/forge/runs/provide/run-008/raw?path=deliverables/bell.png`;
  - chime `…/wfe/api/projects/agsfx/runs/create/run-002/raw?path=candidates/D_softer_E5.wav`;
  - gear `…/wfe/api/projects/forge/runs/provide/run-009/raw?path=deliverables/gear.png`.
- Context: `http://agstudio.local:8093/wfe/#/home/context` (system `developer-preferences`), `#/ws/forge/context`.
- The glossary project: `http://agstudio.local:8093/wfe/#/ws/glossary`.

Experiential feedback is pending. Technical completion does not depend on it.

## Completion criteria

| Criterion | State |
| --- | --- |
| Ordinary managed project on common WFE execution, no separate runtime or Front type | met |
| Express, steer and receive results through one entrance without navigating Zulip | met |
| Unambiguous identities across restart and replay | met (receipts, tokens, run-named replies; recovery trial) |
| Meaningful agdevworld operations through common tools | met. Matrix in report2; no browser operation lacks an agent tool. Committing an edited definition is Git work for both |
| Broad authority with truthful provenance, no new gates | met (`operator` provenance, `--from`; no permission framework) |
| Conversation usable while specialist work executes, measured | met (18 s; one known limit below) |
| Preferences and lessons persist in their scopes | met for preferences (system and project, reused). Lessons have their home (`knowledge/`), but no coordination lesson was written in the trials |
| Real two-project work, mid-flight steering, project creation, reuse, delivery without Omni doing Front's part | met |
| Legacy responsibilities have destinations; no stranded callbacks or duplicate consumers | met |
| Deployed, LAN-accessible, prepared for low-effort inspection | met |

## Named limitations

- **One conversation slot.** Front's own long work shares it with new turns. Measured once: a 45 s wait. More capacity needs a second lane executor thread.
- **A shell process an agent leaves behind keeps its attempt open** after the harness ends.
- **Remote workers refuse extra lanes** (their state file has one writer).
- **Definitions Front edits are saved, not committed**, as with the editor's Save.
- **The Front Desk keeps a few legacy labels.** Its progress and contexts panels still read the relay, and the operation room's "Ask Front" buttons now return the relay's retirement message.
- **No coordination lesson was recorded yet.** The mechanism (`knowledge/`, integration) is in place but unexercised by a real lesson.
