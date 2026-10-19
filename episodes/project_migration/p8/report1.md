# p8 Step 1 — current system, operation coverage and design

Machine facts and raw observations are in pj-agdev's ignored
`.local/project-migration-p8/notes.md`. Here: **coordinator** (the WFE service
and its area), **legacy Front** (the agfront Zulip listener), **managed Front**
(the project this phase creates).

## Starting point

| Item | State (observed 2026-10-19) |
| --- | --- |
| WFE | installed `ea09c14`; source `cd4ea20` (docs only beyond the release) |
| Workers | `studio` (coordinator host, default) and `gpupc`, one slot each, both idle |
| Non-terminal runs | jevlab `experiment/run-001`, stopped by the Developer since 10-13 |
| Legacy Front | listener running; last serving 2026-09-28 (a p7 fault trial); startup recovery queued nothing |
| Legacy Front's open work | no open routine runs (11 routines, all finished or retired); no open argues (6); no open Front Desk conversation (51); 10 unresolved `front-*` topics, all from 09-12 to 09-23 and about work of the retired autolab mission system |
| Nautobot/nctl | fresh observation of the coordinator host this step; drift converged; placement `agfront-agstudio` active (`manual_toolchain`, process pattern) |
| Unrelated working changes | untracked braindumps in devdocs only |

The cutover risk is low: nothing is in flight in the legacy path.

## The human conversation path today

```
Front Desk (agdevworld ?view=frontdesk, Phaser scene, roomState.ts deskAdapter)
  → agentroom relay POST /frontdesk/<id>/post   (as the Developer's Zulip account)
  → Zulip #front › front-desk-<id>
  → agfront listener (mirror, durable queue, serving journal)  role desk|front|routine_run|argue
  → claude_code run with agentchat/agproject/agrefs/agrun; reply marked <ag-reply>
  → reply posted in the same topic; other agents reached with agentchat send
     ([rootchat] note in their topic; their answer naming Front serves the home again)
  → relay reads it back off its mirror; render worker re-voices it into #memo
```

| Hop | Authoritative record |
| --- | --- |
| Conversation | Zulip messages (mirrored) |
| Whether a post was served and the reply delivered | the listener's serving journal (`listener.sqlite`) plus ack/reply posts |
| Delegation home and callbacks | `[rootchat]` notes in the remote topic; `[served]` marks at home |
| Routine run lifecycle | Front's posts in `routinerun-*`, the `ag-routinerun` block, `[delivered]` |
| Model executions | `agfront/.local/agent/<role>/run-*.json` |

Specialist work moved to WFE since p3–p6: forge, archsage, cagent, playprey,
agsfx, ppt, rts-vs-bot are managed projects. Legacy Front reaches none of them
as projects. It has no `wfe` tool. Its only WFE path is the Zulip intake
grant `intake.agents` (Front → cagent consult/change). Its guides still name
the retired archsage council, cagent's listener, the Observer conversation
monitor and "the developer creates projects".

## Legacy responsibilities and their destinations

| # | Responsibility | Implementation | Destination |
| --- | --- | --- | --- |
| R1 | Direct chat (`front-*`) | listener role `front` | managed Front, through the shared conversation facility (Step 3) |
| R2 | Front Desk (`front-desk-*`) | role `desk`; relay `/frontdesk*`; Front Desk scene | the same scene, new adapter on the WFE conversation API; relay desk routes retired |
| R3 | Routine runs (`routinerun-*`) | `routine.py`, `agrun`, role `routine_run`; relay `/routines`, `/chat`, `/routines/<n>/start` | **retired as an execution model.** Every routine is finished or retired, and their guides delegate to the retired mission system. The routine channels stay readable history. A recurring need becomes an ordinary Front request or a project workflow. The relay's routine board stays read-only; its start and chat writes are removed |
| R4 | Argue facilitation (`argue-*`) | `argue.py`, role `argue`; relay `/argues`, Arguing Room | **retired.** No argue is open, and its participants (archsage council, cagent listener) are retired. Developing a vague desire is an ordinary Front conversation. The Arguing Room remains a read-only history view |
| R5 | Re-voicing into character dialogue | `render.py`, `present.py` (inside the listener process); `#memo` | **retired.** The Front Desk shows posts as written, which it already does when no rendering exists. Old memos stay readable. A future presentation is a separate service over the conversation store, not part of Front's runs |
| R6 | Settings/characters | `settings.py`, agdevworld-settings | kept for the Front Desk scene (portraits); no Front run reads it |
| R7 | Budget observation (`agbudget`) | `budget.py` → relay `/budget` | the relay route stays; Front reads it with `curl` when a usage condition matters |
| R8 | `#agents` introduction | `intro.py` | a retirement introduction (✔) at cutover; managed Front is reached through the conversation facility and WFE, not the board |
| R9 | Execution options | `instance.py` | WFE execution settings (profile per workflow) |
| R10 | Trials and fault injection | `trial.py`, `faults/` | archived with the checkout; the new behavior is tested in WFE |
| R11 | Evidence rendering of chatlogs | `evidence.py` | superseded by `wfe conversation show` |
| R12 | `agrun` controls | `agrun.py` | retired with R3 |
| — | Preferences/memories | none persisted. Everything was in Zulip notes | nothing to migrate. Durable knowledge starts in context blocks and Front's repository (Step 4) |
| — | Cagent intake grant | `zulip.json intake.agents` (user 15) | removed; managed Front requests through WFE operations |

Dependents to change with the cutover:
- relay `frontdesk.py`, `chat.py`, `progress.py` (its request set is `#front`), `closing.py`/`close.py` desk kinds, `routines.py` start/chat, `inflight`/`cost` roots;
- frontend `roomState.ts`, `operationDashboard.ts` ("Ask Front to run it"), `chatPanel.ts`, `detailPopup.ts` ("Ask Front");
- launchd `com.agdev.agfront-zulip`; Nautobot placement `agfront-agstudio`;
- pyagag's argue owner and routine runner lookup. These are inert once no routine or argue is opened, so they need no change.

**Evidence-backed lessons carried forward** (each keeps its trial reference into
Front's new guide or the shared contract):
1. Answer from what was found; ask the named agent instead of reading around it (agent_guide p2 ex1).
2. An ack or "started" is not done; only a completed record is (failsafe p3 A2).
3. Never claim to have run, played or verified something; cite who produced the evidence (adventure_game p1/p2; failsafe p7 run-0183).
4. Bind a delegation to the work that consumes the answer (routine_tests p2 ex1). In WFE this is the delegate node.
5. Do not end work while anything it waits for is in flight (routine_tests p1).
6. A failed or stale usage read is unknown, never 0 (refine_routine p1).
7. A preference acted on once is forgotten; record it where later work reads it (runtime-profile).
8. Begin a reply with the reply (seen live twice, front_desk).

Dropped as obsolete: proposal before acting, "do not open project repositories",
the archsage/cagent/Observer-monitor routing, `#front` mention hazards.

## Operation coverage (UI → agent tools)

Every UI write reaches one HTTP route that calls the same server function as
the CLI. The rules live in one reducer (`shared/run.ts`). Identity is declared:
HTTP takes `by`; the CLI derives the actor from `WFE_ATTEMPT` or `--by`.

| Area | UI | Agent tool today | Gap |
| --- | --- | --- | --- |
| Discovery and reads (projects, workflows, runs, IO, delegations, context, observe, workers, boast) | yes | `wfe projects`, `wfe run show/state/io`, `wfe delegate show`, `wfe context …`, `wfe observe`, `wfe exec workers` | none |
| Create project (Gitea), register, obtain workspace, shared repos, submodules | yes | `wfe create`, `register`, `project register/obtain`, `repo …`, `add-repo` | creation must not duplicate on retry (to verify: `--resume` journal) |
| Edit project.yaml (intent, goals, guides, execution) | yes | file edit only | **G1** no verb; no server-side base revision (UI-only `rev` check) |
| Create workflow | yes | `wfe workflow new` | none |
| Edit workflow graph / execution settings | yes | file edit, `arrange`, `nodes instantiate` | **G1** no verb with a base revision |
| Approve | yes | `wfe approve` | record only (as intended) |
| Request a run | yes (person) | `wfe request --braindump --author` (a person's words) | **G2** no queued agent-authored request (`run create --request` queues nothing); CLI default receipt is not stable |
| Instruct, answer, decide, stop, cancel, resume, integrate, presentations, verify | yes | `wfe run …` | **G3** acting on *another* run from inside an attempt records under the target's agent and fails `attempt-stale`; **G4** an instruction never wakes a waiting run; an instruction on a completed run is accepted silently although the contract says `run-completed` |
| Adjust (add work, delegate nodes) | yes | `wfe run adjust` (+ `--completing`, `--propose`) | **G5** a delegate node added by adjustment is untested end to end |
| Delegation writes | read-only | `wfe delegate …` | none |
| Context blocks | yes | `wfe context set/remove --base` (409 on stale) | none |
| Recover, assign, session retire, workflow basics | no | CLI | none for agents |
| Raw HTTP `ops` | — | — | **G6** `delegation.*`, `run.assign`, `session.retire` pass the ops route without their domain checks |
| Commit/push of definitions | no (Save writes the file only) | no | parity: runs read definitions from the workspace; publishing a definition is Git work outside both |

Conversational gaps (not UI operations):
- **G7**: there is no conversation spanning runs. One Zulip request topic is one run, and a post in a run topic wakes nothing.
- **G8**: one slot per worker, FIFO, no lane. A conversational turn would wait behind any long attempt on its node.

Shared infrastructure (WFE, executor) owns G1–G8. Front's ordinary project
content is its guide, workflow and knowledge.

## Design decisions

**Conversation facility (in WFE, `api/conversations`, `wfe conversation`).**
- A conversation is a durable id. It holds an ordered list of messages:
  - the Developer's messages, with original text, author, time and submit receipt;
  - Front's replies, each naming the run and attempt that wrote it and the message(s) it answers (`re`).
- **A received message starts one bounded Front run** (`front/converse`). The request receipt is `conversation:<conv>:<message>`, so a replay or retry finds the same run. The run's input is the message verbatim as the Developer's braindump, plus the conversation reference. The facility records the link `message → run (turn)`. It interprets nothing.
- **Front decides the meaning in that run.** It reads the conversation (`wfe conversation show`) and its linked runs, then records its interpretation as a link: `new`, `follow-up`, `correction`, `answer`, `question` or `steer`, with the target run and a note. A correction is a new link that supersedes the old one; the message itself is never rewritten. Then it:
  - answers from evidence and completes; or
  - does the work in this run (adjust: do/delegate nodes) and replies when the result is examined; or
  - steers an existing run (`wfe run instruct` with the Developer's words as `from`, Front as the actor) and its child (`wfe delegate instruct`); or
  - answers another run's question with the Developer's words; or
  - asks a concise clarification, leaving the message unbound to any work.
- **Replies** are written with `wfe conversation reply --re <message>`, keyed by `(attempt, key)` so a retried command posts once. A reply always names its run, so concurrent completions are attributed by run and `re`, never by recency.
- The Front Desk reads and posts through this API on the WFE origin (`:8093/wfe/`). Zulip is not on the human path. Front's runs still have their ordinary run/IO topics, as evidence.

**Actor and authority.**
- Front's runs record as `front-agent`, the workflow's `execution.agent`. Acting on another run from inside an attempt records Front as `by` (G3 fix), with the Developer's words kept as `from`/`source` where they are the Developer's.
- No permission framework is added. WFE does not authenticate, and agent tool grants (`wfrun` role, `Bash(wfe:*)`, git) already cover the operations. The Gitea credential stays in the area (`gitea.json`) and is read by `wfe`.
- The only Front-specific facts are configuration: Front's workflow names its agent and lane.

**Responsiveness (G8).**
- Specialist work is delegated, and a waiting parent holds no slot. The open risk is a turn waiting behind a long child attempt on the same worker.
- Decision: **worker lanes**. A worker may offer an extra slot for a named lane (`workers.json`, e.g. `studio: {lanes: {conversation: 1}}`). A workflow declares `execution.lane`, frozen at run creation like `node`. Claim counts open attempts per (worker, lane). The executor runs one process per free lane.
- The default lane keeps exactly today's behavior.
- Considered and rejected:
  - a second "remote" worker on the same host: a second executor with its own copies and checkpoint path;
  - priority: it reorders, but does not help while the only slot is taken.

**Waking on instructions (G4).** An instruction to a run with no open
attempt, no hold, not completed and not cancelled queues a continuation
(receipt `instruction:<receipt>`). An instruction to a completed run is refused
`run-completed`, as the contract says. Front then starts a successor run
(`--predecessor`) or answers from the record.

**Agent-authored requests (G2).** `wfe request <wf> --request <file>
--requester <agent>` queues a run whose input is the agent's request,
`entrusted-by` the conversation message and Front's run, with a stable
receipt. Delegation remains the way to get specialist work whose result Front
judges. A plain request is for work Front only starts.

**Definition writes (G1).** `wfe workflow save|project save --base <rev>`
calls the UI's save functions with a server-side base check. The PUT routes
get the same check, so a stale write reports the current revision instead of
overwriting a UI edit.

**Knowledge.**
- General preferences: system context block `developer-preferences`, manager `front-agent`.
- Project preferences: that project's blocks.
- Coordination lessons: Front repository `knowledge/`, integrated like any run work.
- Active work: run records.

## Trial scope

- **Two-project request.** forge (an image) and agsfx (a short sound) make a small notification kit, delivered together with links. The result is inspectable in the run views and outputs. Fallback if the ComfyUI host is down: archsage (knowledge) and cagent (cluster facts) for a hosting recommendation.
- **Existing information.** A question answered from run records, e.g. what playprey's last runs delivered.
- **Missing project.** A small `glossary` project that keeps the system's terms (from devpolicy/terms.md and README_DEV): Front checks that no existing project serves this, creates it, places its workflow and gets a first result.
- **Responsiveness, mid-flight change, recovery.** These are exercised on the two-project request while its children run.

Ordinary design choices above were made without an approval checkpoint.
