# fix6 step 1 — evidence model and shared assessment

Plan: [plan.md](plan.md) step 1. Code: agdevworld `4888ea3` (steps 1–3 went
out as one change because the records, the assessment and the correction
share the reducer).

## Operation boundaries traced

| Boundary | Where | What it holds |
| --- | --- | --- |
| Dispatch | `server/delegate.ts` `dispatch` → `delegation.dispatch` | target resolved, request by hash, the node waits |
| Take-up | `delegation.take-up` (CLI `wfe delegate take-up`) | facts read; the node resumes |
| Disposition | `delegation.dispose` — before fix6 a bare `runOp` from the CLI; no HTTP route | closes; `satisfied` lets the node complete |
| Child completion relay | `shared/delegation.ts` `childFacts` + `DelegateStore.relayOne` | one `completed` per completing entry, told after the completing attempt ended; manifest from the child's copy |
| Result manifest | `DelegateStore.manifest` | outcome, report, artifacts by hash, decision, branch |
| Outward report | `cli/conversation.ts` `reply|contribute --kind result|synthesis [--completing]` → `adviseRun` (run lock, read only) then the post | conversation posts are CLI-only; no agent HTTP route |
| Run completion | `runOpLocked` / `adjustRun --completing` / HTTP `ops`, `adjust` | fix5's advice |
| Run reopening | `run.adjust` on a completed run (no state check), `node.start` on a completed node; `instruction.receive` on a completed run is kept but continues nothing; the intake refuses posts for a completed child (`closed`) | — |
| Continuation | `ExecStore.enqueue` by receipt; `afterAnswer/afterInstruction/afterDelegation`; `reconcile` re-queues lost ones; `claim` drops held/cancelled/open runs | — |
| Integration | `finished` → `queueIntegration` on completion or an acceptance in the attempt; acceptance binds the copy's tree | — |
| Remote worker | `cli/remote.ts` `forward`: every other command (delegate, conversation, run) runs in the coordinator's `wfe` with its flags | `--yes` travels with the argv |

Locks: one cross-process lock per `run.json` (`server/lock.ts`). The relay
writes the parent under the parent's lock and reads the child without its
lock. The child's history is append-only and replayable, so the assessment
freezes its facts by replaying the child's own history to a position
(below) instead of locking two runs at once.

## The records

- Parent `delegations.<d>.revisions[]` (present once used): the result
  returned (event, manifest digest, the child's completing seq), instruction,
  reason, whose words (`because`).
- Child `corrections.<c>` (present once used): delegation uid and request
  number, the result it returns (completing seq), reason, text, from,
  source, `during`, `takenUp`, `addressed {seq, how: work|no-change}`.
- `presentation.retain` → `presentations.<p>.retained {note, owner, because}`:
  the person's request to keep a page, with who stops it.
- `delegation.dispose` accepts `manifest` and refuses a mismatch
  (`result-superseded`) and a returned result (`result-returned`).
- Optional fields only: existing records replay unchanged (checked on the
  host's records, see step 5), so no migration was needed.

## The assessment (`shared/assessment.ts`)

`assessResult(parent, delegation, resultEvent, childEvidence)` returns
checks with `status` (`ok`, `missing`, `serving`, `unknown`), `facts`, the
frozen catalog `expectations` the check is about, a one-line `summary`, and
the parent's `repair`s. The file layer (`server/delegate.ts`
`assessDelegation`) gives it:

- **frozen**: the child's record as of the result — its history replayed to
  the first entry at or after the completing one after which no attempt is
  open (`resultPosition`, `recordAt`); the revision in force then;
- **now**: the child's record as it is, for serving state, corrections and
  integrations, listed apart as "now";
- **catalog**: the child's frozen snapshot;
- **changed**: files the child's copy changed outside its records, in
  every repository it owns, attributed to the node running when each was
  written (`server/reminders.ts` `changedPaths`, `pathsByNode`);
- **unavailable**: what could not be read, said explicitly.

Binding: delegation id/uid, result event, manifest digest, child completing
seq, frozen position, revision, catalog sha256. Revalidation at mutation:
the reducer checks the event is the latest and the manifest matches
(`dispose`, `revise`); `DelegateStore.dispose` assesses and records under
the parent's lock.

| Check | Implemented as |
| --- | --- |
| Presentation missing | work = `work` template, or an untemplated `do`/`study` node the run added with files beyond notes (`substantiveNodes`); viewable candidates by extension from manifest, node artifacts and changed files; `missing` when work completed after the latest presentation (an earlier page does not cover later work) or viewable files with no presentation |
| Person's review not recorded | only `accepts: person` with a deliverable; `ok` when the person's own answer is in the child's questions, or this delegation's answers/corrections relay their words (`because`); else `unknown` |
| Presentation still serving | open presentations now: `serving` only for an answered/never-asked review with no cleanup and no `retained`; an open review or a kept page is a fact |

The child's own fix5 checks stay, with the same wider `work` role:
`roleOf(…, substantive)`; `evaluate` receives `changedByNode` from the file
layer.

## Read-only replay on the host's records

A scratch script assessed every delegation of every registered project
(the deployed records, read only; a scratch delegate store):

- `presentation-missing`: ponderer/conduct/run-031, 032, 034, 035 →
  misc/freework/run-001…004 (run-002's hand-written `work-2d` counted; run-003's
  6 screenshots and spin video found in `wedo/wedo-misc`; run-004's
  `dambreak_comparison.mp4`), conduct/run-029 d3 → gimmickimg run-010,
  front/converse/run-049 → gimmickimg run-001 (work inside the talk with
  viewable files). run-036 → misc run-005 (presented as p1): `ok`.
- No false alarm on studies, consultations, syntheses, boast, cagent
  change/report, forge provide.
- `review-unrecorded` first fired on every `accepts: person` result,
  consultations included; it was narrowed to results with a deliverable.
  It remains `unknown` for run-005 (q1 answered by ponderer-agent with no
  `because`), forge/provide children answered by front's agent, and
  gimmickimg freework results relayed by ponderer: the person's answer is
  not traceable in those records.
- `presentation-serving`: no forgotten page among current delegations.

## Not done here

- Expectations stay prose; nothing compiles them into rules.
- Files are read from the coordinator's copy of the child; a remote
  child's files appear there after its publication (fix5's limit).
