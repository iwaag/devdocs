# argue p4 step 1 — contracts settled

Inspected: WFE agdevworld `978a5ed` (`server/discussions.ts`,
`discussionArchive.ts`, `discussionSchema.ts`, `shared/run.ts` reducer,
`server/exec.ts` continuation hooks, `docs/discussions.md`, `catalog.md`,
`adjustments.md`, `autonomy.md`, `delegation.md`, `runs.md`), agautolab
`797b2d0` wfrun guide (*A run that speaks in a discussion*), pj-argue
`6213c6c` (`facilitate`, `synthesize`, guides), pj-front `discuss`, the
operating area's `conversations.json`, and the p3/ex1 reports.

## Live state before the cutover (2026-10-21)

- Discussions: 3 closed (`argue-p3-trial`, `argue-p3-ex1-trial`,
  `p3-s2-fixture`), 3 open but idle (`p16-s3-room`, `p16-s4-provenance`,
  `20261020-233458-r93d5v`). **No open invitation, no awaited result.**
- `argue/facilitate`: run-001…012 completed, run-009 cancelled. No
  facilitator run waits. `argue/synthesize` run-002…004 completed with
  `_final-report` left ready (no obligation to the discussion).
- Only non-final run in the discussion projects: `jevlab/experiment/run-001`,
  paused by the Developer on 2026-10-18; unrelated, left alone.

So the cutover has no outstanding discussion obligation to transfer. A
person's later message in an old open discussion invites the new
facilitator (ponderer); nothing is replayed.

## Ownership and routing

| Owner | Owns | Does not |
| --- | --- | --- |
| Front | The Developer's purpose; alignment of the discussion with it; which projects take part (`wfe discussion join/leave` from its `discuss` run); commissioning and judging work it takes on (its delegate nodes) | Conduct turns; give a "go" to anyone's own work |
| ponderer (`ponderer/conduct`, agent `ponderer-agent`) | The discussion's progression: current issue, questions and speakers among the members, sufficiency, structure, requests to argue for synthesis, the conclusion and its account of what is left | Specialist work, project selection beyond pointing out a missing perspective (it asks Front) |
| Participants (`<project>/discuss`) | Their domain; their own work through ordinary workflows | Inviting anyone |
| argue (`argue/synthesize`, now an invitable participant) | Archive interpretation: `summary.md`, `knowledge/`, the episode index and analyses | Facilitation (`argue/facilitate` retires) |
| WFE | Identity, invitations, activity records, waits, continuation, recovery, archive writer | Choosing the next step |
| aghangout | Presenting speech, the current activity, questions, waits, decisions, links | — |

**Conductor.** A discussion has at most one *conductor run*: the open
`ponderer/conduct` run that owns its progression. It is recorded on the
discussion (`conductor`), set when the facilitator's invitation starts a run
or when a ponderer attempt creates the discussion. The discussion id stays
independent of runs: a conductor run that completes leaves the discussion
open, and the next untargeted message (or a late outcome) starts a new
conductor run.

**Requester.** Recorded on the discussion (`requester`):
`{kind: person, id}` when a person created it, `{kind: run, run}` when an
agent's run requested the ponderer run that created it (delegation child or
`wfe request --request`). Ponderer's `talk` goes to the requester:

- person: a WFE question on the talk node, posted in the discussion as a
  `question` message; the person's reply is routed to the conductor run as
  the answer (below);
- agent: the same WFE question addressed `--to <requester>`, which the
  existing delegation/request relay carries to the requesting run
  (`wfe delegate answer` comes back as the answer). It is also posted in the
  discussion so readers see it.

**Routing a person's message (changed).** Addressing participants invites
them, as before. Addressing nobody (or the facilitator):

1. an open conductor run exists → the message is **delivered to it**: as the
   answer to the question it names (`re`), or to the conductor's only open
   question to the person; otherwise as an instruction. No second run.
2. no open conductor → the facilitator is invited; that run becomes the
   conductor.

This removes ex1's orphan: run-009 waited, #192 started run-010 instead.

## Activities

Representation: **node metadata `activity`** on ordinary node types, plus
`wfe discussion` commands that bind each activity to its node and records.
No new primitive node type: the reducer's node states, waits and fencing
already express what each needs; what was missing is correlation and
completion evidence, which lives in a new `activities` table of the
discussion store. A run's definition says `activity: ask`; the run view,
`wfe run show` and aghangout show it.

| `activity` | Node type | Command (from the conductor's attempt) | Completion evidence | Completed by |
| --- | --- | --- | --- | --- |
| `announce` | `do` | `wfe discussion announce --node <n> --text … --re …` | The message id (kind `announce`), message key `activity:<run>:<node>` | the command (message, then node); reconcile repairs a crash between them |
| `ask` | `do` | `wfe discussion ask <participant>… --node <n> --question … --re …`, or `wfe discussion await --node <n> --run <r>` for a commissioned result | Every invitation of the activity resolved (answered by #m, unanswered, failed, withdrawn), or settled by ponderer with the open ones named | WFE, once the attempt has ended (start + complete, by `discussion`); then one instruction continues the run |
| `ponder` | `do` | `wfe discussion ponder --node <n> --decision … --source <msg>… [--next <activity>:<id>]… \| --definition <file>` | Decision text, sources, selected next activities and the run revision it recorded (or "no change") | the command, atomically with the revision (`run adjust --completing`) |
| `talk` | `talk` | `wfe discussion talk --node <n> --question … --re …` | A correlated answer (question taken up), a withdrawal, or an unresolved outcome named in the conclusion | the agent after take-up, as any talk |

- **Adaptive graph.** `ponderer/conduct` starts with one `ponder` node.
  `--next` adds a chain after the ponder node (`announce:agenda`,
  `ask:facts`, `ponder:after-facts`); `--definition` is any revision.
  Another round is new nodes; completed nodes and their records stay.
  No cycle, no fixed round, no consensus rule.
- **Waits.** `ask` waits as `external: discussion:<id>:activity:<a>`
  (holder = the invited participants, or the awaited run); `talk` waits on
  its question; a delegate node waits on its delegation. Each frees the slot.
- **Partial outcomes.** One ask node may invite several participants
  (parallel); its node completes when all are resolved. Ponderer wanting to
  move on earlier settles it (`wfe discussion settle --node <n> --reason`):
  the node completes with the open invitations named; their later answers
  are late outcomes told to the conductor (or a new conductor run).
  Unavailable participants fail visibly and resolve at once.
- **Delivered speech, unfinished node** (p3 rts-vs-bot run-001/004).
  `wfe discussion contribute --complete <node>` speaks and completes the
  node in one command, the participants' guide's final step. A crash
  between leaves an activity row whose message exists: reconcile completes
  the node and, when the hold's only cause was that node left running,
  records the repair as a recovery of the hold.
- **Ending.** `wfe discussion announce --conclude [--close]` posts the
  conclusion and records the **obligations** snapshot: open invitations,
  awaited results, unanswered questions, linked runs still working — each
  with who owes it and who receives it later (the conductor run, a new
  conductor run, a delegation parent, or "recorded only: the discussion is
  closed"). Closing still cancels nothing.

## Event traces

Notation: `#n` message, `I<n>` invitation, `A<n>` activity, `P` the
conductor run `ponderer/conduct/run-001`.

**1. Brief, announce, parallel asks.**
1. Person posts #1 (no addressee) → no conductor → `I1` to ponderer → `P`
   created by receipt `discussion:<d>:invitation:1`, recorded conductor.
2. `P` attempt 1: `ponder` → `wfe discussion ponder --node ponder
   --decision "facts first: …" --source 1 --next announce:agenda --next
   ask:facts --next ponder:after-facts` → `A1` (ponder), revision r1,
   node `ponder` completed.
3. `announce --node agenda` → #2 kind announce, `A2`, node completed.
   `I1` answered by #2. No wait.
4. `ask rts-vs-bot jevlab --node facts --question … --re 1 2` → `I2`, `I3`
   (activity `A3`, awaited by `P`), node `facts` waits. Attempt ends
   normally (only waits remain).
5. Participants answer #3, #4 (`contribute --complete talk`): `I2`, `I3`
   answered.
6. Reconcile: `A3` resolved, `P` has no attempt → `facts` start + complete
   (outcome lists `I2→#3`, `I3→#4`) → instruction "A3 resolved …" → job →
   attempt 2 on `after-facts`.

**2. Partial outcome, unavailable participant.** `ask forge lowerpoly`
→ `I4` started, `I5` failed (no discuss workflow) at once. `forge` answers
→ `A` resolved: answered + failed. Continuation as above; ponderer decides
whether the failure matters.

**3. Requester clarification (person).** `talk --node clarify --question
"…?" --re 3` → WFE question `q1` on `clarify`, #5 kind question
(`question: q1`). Attempt ends (waiting on q1). Person posts #6 re #5, no
addressee → conductor `P` open → `I6` delivered to `P` as the answer to q1
→ resume job → attempt takes up q1, completes `clarify`.
Without `re`: the only open question to the person in `P` is q1 → same.

**4. Requester clarification (Front).** Front's `converse` run `F` delegates
to `ponderer/conduct` (child `P`); `P` creates the discussion (requester =
`F`, conductor = `P`). `talk --node clarify` asks q1 `--to front-agent`;
the delegation relays it to `F`; Front answers (`wfe delegate answer`) →
`P` resumes on q1. The discussion shows #question and the answer note.

**5. Delegated work and result review.** Ponderer asks rts-vs-bot to do the
match; `rts-vs-bot/discuss/run-007` replies #8 ("dispatched improve-game
run-012") and keeps waiting on its delegate node. Ponderer's `ponder`
selects `ask:await-match`; `await --node await-match --run
rts-vs-bot/discuss/run-007 --note "the match result"` → awaited `I8`.
When run-007 posts #9 kind result → `I8` answered → node completed →
`P` continued → `ask` for review → `ponder` → `announce --conclude`.

**6. Closing with outstanding work.** `announce --conclude --close` while
`jevlab/experiment/run-9` still runs under `jevlab/discuss/run-4`:
obligations = [{run: jevlab/experiment/run-9, owes: result, owner:
jevlab/discuss/run-4 (delegation parent), receiver: "recorded only (closed)
— reopen to continue"}]. `P` completes.

**7. Late reply.** `I3` was `unanswered` (run ended); the participant's
later run posts a reply naming it → `I3` answered late → if `P` is open it
is instructed; if completed, a new conductor run gets the outcome; if the
discussion is closed, it is recorded only.

**8. Restart / duplicate delivery.** Each step has a receipt or key:
invitation receipts, message keys `activity:<run>:<node>`, activity tokens
`<run>:<node>`, completion receipts `discussion:<d>:activity:<a>:complete`,
continuation receipts by outcome labels. A retried command finds the same
rows; reconcile repeats only what is not recorded.

## Records, published state, analysis

- **Store v7**: `activities` (run, node, attempt, kind, state, text,
  sources, invitations, message, question, decision {next, revision},
  obligations, outcome); `invitations.activity`; `conversations.conductor`,
  `conversations.requester`; `annotations` (intervention, recovery,
  evaluation, correction — by whom, about which messages; never speech).
- **Archive** (`ag.argue.discussion.v1`, extended): transcript records
  `activity` and `annotation`; manifest `conductor`, `requester`, horizon
  `activity`, `annotation`. The writer stays the one writer.
- **Published integration evidence**: `wfe projects run <p> <wf>/<run>
  --integration` — the run's `run.integrate` records, the published default
  branch (Gitea, fetched now, with time) and whether each integrated commit
  is on it, and the local workspace branch head (ahead/behind). Three
  observations, kept apart.
- **Episode index**: pj-argue `episodes/index.json` and
  `episodes/<episode>.md` (analysis: task, conditions, model/profile,
  revisions, guidance, interventions, observations, unknowns), validated
  by `tools/argue episodes`. p3 and ex1 are entered with their recorded
  limitations; unrecorded context is `unknown`.

## Affected components

| Component | Change |
| --- | --- |
| WFE `shared/model.ts`, `validate.ts`, compile digest | `activity` node field, `activity-kind` / `activity-type` checks |
| WFE `server/discussions.ts`, schema v7, archive | conductor, requester, activities, settle, annotations, routing, obligations, repairs |
| WFE `cli/discussion.ts`, API | `announce`, `ask`, `await --node`, `ponder`, `talk`, `settle`, `state`, `annotate`, `contribute --complete` |
| WFE `cli/projects.ts` | `--integration` |
| WFE run view / Home | activity badges; Discussions card shows the conductor and current activity |
| agautolab wfrun guide | participant facts (`--complete`), facilitator → ponderer |
| new `pj-ponderer` | project, `conduct` workflow, guides |
| pj-argue | `facilitate` retired; `synthesize` invitable; episode index |
| pj-front | `discuss.md`: alignment and participation, no "go" |
| pj-aghangout | current activity, questions and respondents, waits, decisions, links |
| `conversations.json` | facilitator `ponderer/conduct`; participant `argue/synthesize` |

## Migration and recovery

- Store v6 → v7 is additive (columns, tables). Old rows keep meaning;
  invitations without `activity` keep the per-run continuation.
- Deploy order: WFE release (store migrates on start) → projects
  (ponderer created, argue/front updated, integrated) → `conversations.json`
  switch → aghangout release. Rollback = previous release **and** the
  pre-deploy store backup (an old release refuses v7).
- No in-flight discussion obligation exists (above); old closed
  discussions are not touched; their `argue/facilitate` runs stay history.
