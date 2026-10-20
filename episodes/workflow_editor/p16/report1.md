# workflow_editor p16 — Step 1: the boundary

Step 1 of [plan.md](plan.md): inspect what exists, map a discussion onto it,
and settle the contracts before code.

## What exists (inspected 2026-10-20)

| Facility | Where | What a discussion can reuse |
| --- | --- | --- |
| Conversation store | agdevworld `workflow_editor/server/conversations.ts`, store v3 (`conversations`, `messages`, `links`) | Stable client-minted id; verbatim messages with author, time, submit token, `via`, `re`; agent replies keyed per run (`(conv, run, key)`); links with supersession; receipts `conversation:<id>:<msg>` for run requests; `dispatchDue()` called by the executor's `wfe exec reconcile` every minute |
| Run request | `ExecStore.request({workspace, workflow, text, author, receipt, via, source})` | Idempotent by receipt; `input.source` (`ConversationSource`) is recorded on the run |
| Continuation | `instruction.receive` (with `receipt`) + `ExecStore.afterInstruction()` (runs.md *An instruction continues a run nobody executes*) | A run that waits (question, delegation, external `node.wait`) and is not held gets one resume job per instruction; a completed run keeps the instruction and nothing continues it |
| Delegation | `wfe delegate`, delivered through Zulip request topics | Not used: its delivery is the notifier's Zulip path and its child is entrusted *work*, not a voice in a shared discussion |
| Lanes | `workers.json` studio `lanes: {conversation: 1}`; workflows choose `execution.lane` | All conversational turns share one slot |
| Context blocks | `server/contextStore.ts`, `wfe context` | May expose selected argue knowledge later, by reference; not an authority |
| `wfe projects` | read-only access to other managed projects at their committed revision | How a participant reads `pj-argue`'s `knowledge/` |
| Gitea | `Gitea.gitEnv()` (server), `wfe gitea git` (CLI) | Authenticated pushes without a token in config or URL |
| aghangout | `pj-aghangout/app/src` (api.ts, model.ts, room.ts, lobby.ts; 1.3k lines) | Contract client, outbox retried by token, drafts, room presentation, characters per destination |

No discussion concept exists today. The retired Zulip argue (`agag.argue`,
README_DEV *Argues*) is history only and is not imported (out of scope).

## Decisions

### D1. A discussion is a conversation of kind `discussion` in the same store

Store **v4** adds `conversations.kind` (`conversation` | `discussion`) and new
tables. Shared with direct conversations: the id space and format, the
`messages` table (verbatim text, author, time, token, `via`, `re`, run,
attempt, key), `links` and their supersession, run naming
`<ws>/<wf>/<run>`, the view's linked-run status. Different for a discussion:

- it has **members** instead of one destination;
- a person's message does not start "the" destination's turn; it is routed
  into **invitations** (D3);
- agent messages carry the **participant** who spoke and the **invitation**
  they answer;
- `POST api/conversations/<id>/messages` refuses a discussion id (409
  `is-discussion`) and the discussion routes refuse a conversation id, so a
  client cannot use the wrong semantics by accident.

Code: `server/discussions.ts` (a `DiscussionStore` over the conversation
store's database), CLI `wfe discussion`, HTTP `api/discussions`, contract
`docs/discussions.md`.

### D2. Members and membership history

A **participant** has a stable id and a kind:

| Kind | Id | Speaks through |
| --- | --- | --- |
| `person` | the configured person (`iwaag`) or a named trial author | the page / CLI, verbatim |
| `facilitator` | `argue` | runs of `argue/facilitate` |
| `project` | the workspace id (`forge`, `archsage`, …) | runs of that project's discussion workflow (`<ws>/discuss`) |

Agent identity is the run's agent (`forge-agent`); the model is in the run's
own evidence (Agent ≠ Model). Configuration (`conversations.json`
`discussions`) names the facilitator and the participant workflows that may
be invited:

```json
"discussions": {
  "facilitator": { "workspace": "argue", "workflow": "facilitate" },
  "participants": [{ "workspace": "forge", "workflow": "discuss" }, { "workspace": "archsage", "workflow": "discuss" }],
  "archive": { "workspace": "argue" }
}
```

`membership` is an append-only table: `(conv, participant, kind, workspace,
workflow, action join|leave, by, at, note)`. The current members are the
fold of it. The creator, the person and the facilitator join at creation.
Inviting a non-member records a `join` by the inviter first (visible in the
history). A `leave` stops new invitations to that participant; its earlier
speech and invitations stay. Joining again is a new `join` row.

### D3. Routing and invitations

An **invitation** is the only thing that starts a run in a discussion. It is
durable: `invitations(id, conv, participant, workspace, workflow, question,
source messages [], by, by_run, awaited_by, horizon, token, state, run,
error, tries, created, updated, resolved_by_message)`. Its run is requested
with receipt `discussion:<conv>:invitation:<id>` and `input.source =
{kind: conversation, conversation, message: <first source message>,
author, at, invitation: <id>}`. State:

```
pending → started → answered        (the run contributed citing it)
                  → unanswered      (the run ended without contributing)
pending → failed (reason; retried by reconcile up to 3 tries, then by hand)
pending|started → withdrawn          (pending ones on close; by hand)
answered (late)                      (a contribution after unanswered/withdrawn)
```

Routing of a person's message (`to` = addressees, possibly empty):

- **`to` names participants** → one invitation each (by the person, not
  awaited by anyone). The facilitator is not started.
- **`to` is empty** → one invitation to the facilitator (a facilitator
  turn).
- **`to` includes the facilitator** → the same as empty, plus others named.
- An agent's contribution never starts anything by itself (no reply-to-reply
  loop). Only the facilitator may invite from a run; a participant run's
  `invite` is refused with the reason (it may suggest in its text).

`horizon` = the newest message id when the invitation was dispatched; the run
is told it, and `wfe discussion show` marks later messages "after your
invitation" (still readable).

### D4. Facilitator continuation

The facilitator invites with `wfe discussion invite <participant> --question
… --re <msg>…`; from its run, the invitation is **awaited by that run**.
The facilitator then makes its talk node wait (`wfe run wait … --external
discussion:<conv>`) and ends its attempt, freeing the slot.

When an invitation awaited by run F reaches a resolved state (`answered`,
`unanswered`, `failed`, `withdrawn`) **and no other invitation awaited by F
is still pending/started**, the store records one instruction on F (`from:
discussion`, `source` = the discussion and the newest contribution, receipt
`discussion:<conv>:continue:<F>:<ids>`) that lists each resolved invitation
with its outcome, then calls `afterInstruction` — the existing continuation.
The facilitator decides: invite again, synthesize, answer the person, or
complete. Each resolved invitation is marked `reported` so it is told once.
A late contribution (after `unanswered`/`withdrawn`) is reported the same
way, alone. A completed F is told nothing more (runs.md: nothing continues a
completed run); its successor turns see everything in `show`.

### D5. Closing and reopening

`close` stops automatic dispatch: pending (not yet requested) invitations
become `withdrawn` (reason: closed). Started invitations keep running; their
contributions are recorded as usual, but no facilitator continuation is
recorded while closed, and nothing downstream is cancelled. `reopen` (or a
person's message, as for conversations) reopens; withdrawn invitations are
not replayed — they are re-issued explicitly if wanted. Completed/answered
invitations are never dispatched again (receipts).

### D6. Records and archive (pj-argue)

WFE's store is authoritative for live speech. **One owned writer** — WFE's
`DiscussionArchive`, run in `wfe exec reconcile` (every minute) and by
`wfe discussion archive [<id>]`, serialized by a file lock (`lock.ts`) —
keeps its own clone of `pj-argue` at `<area>/.local/discussions/archive/`.
No run and no person edits that clone.

```text
discussions/<id>/
  manifest.json      format, id, title, kind, created, state, members (fold),
                     membership history, invitations (current), horizon
                     {message, link, membership, invitation}, provenance
                     (runs with their workspace/workflow/agent), exports[]
  transcript.jsonl   append-only records: message | link | membership | invitation-event
  summary.md         synthesis, written by argue runs (not by the writer)
knowledge/<topic>.md cross-discussion knowledge (argue runs)
knowledge/index.md   one line per topic
```

An export: fetch `origin/main` → read the **published** manifest's horizon
(the repository is the truth) → append records beyond it → write the manifest
→ commit (trailer `Discussion-Horizon: <id> m<n> l<n> e<n> i<n>`) → push. On
a non-fast-forward it rebases onto the new main (its files are disjoint from
the runs': only `transcript.jsonl` and `manifest.json`) and retries. The
store's `archives(conv, wanted, exported, commit, pushed, state, error,
tries, at)` is bookkeeping for status only: if the process stops after the
push and before recording, the next export reads the pushed manifest, finds
nothing new, and records the observed commit. Nothing is duplicated because
appends start at the published horizon, never at the bookkeeping.

Scheduling: a discussion is due when its live horizon exceeds the published
one (incremental during discussion), and on close (final catch-up). Status
in the view: `archive: {state: current|behind|failed, published: {commit,
horizon, at}, live horizon, error}`; links to the Gitea tree at that commit.

Interpretation is agent work through ordinary runs of `pj-argue`
(`synthesize`): it reads the archived transcript in its run copy, writes
`summary.md` and `knowledge/`, cites `#<message>` and the archive commit, and
integrates with `wfe run integrate` (WFE integration merges the run branch
into main; disjoint from the writer's files). A synthesis is also posted into
the discussion as a message of kind `synthesis`, which clients show apart
from original speech.

### D7. Hand-off and reuse

A hand-off is an ordinary request to the implementing project (`wfe request
… --workspace <p>`, or a delegate node), whose text names the source as
`argue@<commit>:discussions/<id>/summary.md` and the relevant `#<message>`s.
The argue run records the resulting run back into the discussion:
`wfe discussion link <conv> <message> --run <ws>/<wf>/<run> --as handoff
--note …` (new relation `handoff`), which the archive exports as a link
record. A later discussion finds knowledge in `knowledge/index.md` (argue's
own runs have it in their working tree; other projects read it with `wfe
projects files argue knowledge/…`) and cites `argue@<commit>:knowledge/<t>.md`.
A changed conclusion is a new revision of the topic file with a *Revisions*
section naming what changed, why, and the superseded commit.

## Event traces

### T1. Normal exchange (facilitator invites two projects)

```
#1 person: "How should we …?" (to: [])                      → inv-1 facilitator (by person), pending
   reconcile/post: request argue/facilitate run-001 (receipt discussion:d:invitation:1) → inv-1 started
run-001: show; post #2 (facilitator, re #1, inv-1)            → inv-1 answered
run-001: invite forge (re #1) → inv-2 awaited_by run-001; invite archsage → inv-3
         request forge/discuss run-004, archsage/discuss run-007 → started
run-001: wfe run wait talk --external discussion:d; attempt ends (slot free)
run-004: show (horizon #2); contribute #3 (re #1, inv-2)      → inv-2 answered; inv-3 still started: no continuation
run-007: show; contribute #4 (re #1 #3, inv-3)                → inv-3 answered; nothing awaited by run-001 pending
         → instruction i1 on run-001 "inv-2 answered #3 (forge), inv-3 answered #4 (archsage)",
           receipt discussion:d:continue:argue/facilitate/run-001:2,3 → resume job
run-001 (resumed): reads #3 #4; may invite forge again re #4 (conflict) or post synthesis #5 and complete
```

### T2. Direct human question

```
#6 person: "@archsage is X even feasible?" (to: [archsage]) → inv-4 archsage (by person, awaited by nobody)
run-008 archsage/discuss: contribute #7 (re #6, inv-4)        → inv-4 answered; no facilitator turn
```

### T3. Late reply and mid-flight input

```
facilitator run-001 invited forge (inv-5, awaited) and is waiting
#8 person (to: []) mid-flight                                  → inv-6 facilitator (new turn run-002)
run-002: sees inv-5 open and run-001 waiting; answers #8 itself, or forwards with
         wfe run instruct argue/facilitate/run-001 --source-message d:8
forge run-009 ends without contributing                        → reconcile: inv-5 unanswered
         → instruction on run-001 "inv-5 unanswered (forge/discuss/run-009 completed without a contribution)"
forge run-009's later continuation contributes #9 (re #1, inv-5) → inv-5 answered (late)
         → instruction on run-001 "inv-5 answered late #9" (if run-001 still waits; nothing if completed)
```

### T4. Close with an invitation in flight

```
inv-7 archsage started, inv-8 forge pending (forge unavailable)
close                                                          → inv-8 withdrawn (closed); inv-7 keeps running
run-010 contributes #10 (inv-7)                                → recorded; no continuation (closed)
archive due (closed) → final catch-up export includes #10
reopen                                                          → inv-8 stays withdrawn; nothing replays
```

## Execution capacity

Discussion workflows run on the coordinator (`execution.node: studio`) in
the `conversation` lane like the converse turns; the facilitator frees the
slot while it waits. Queued and running times are measured per invitation
(invitation `created` → run `attempt.begin` → contribution) in Step 3; the
lane gets a second slot only if the measurement shows the one slot stops
usable interaction.

## Affected repositories and deployment path

| Repository | Change |
| --- | --- |
| agdevworld | store v4, `server/discussions.ts`, archive writer, API, CLI, docs, reconcile hook, `ConversationSource.invitation` |
| agautolab | wfrun guide: *A run that speaks in a discussion* (shared facts for every participant) |
| pj-argue (new, Gitea) | intent, guide, `facilitate`, `synthesize`, `freework`, `tools/argue` (read/validate archive), `discussions/`, `knowledge/` |
| pj-forge, pj-archsage | `discuss` workflows and guides |
| pj-aghangout | discussion lobby, room, composer with addressees, invitations, records |
| pj-agdev | pins; devenv notes (ignored) |
| devdocs | reports, README_DEV |

Deployment: back up the area's sqlite stores and `conversations.json`; WFE
`wfe-release.sh deploy` (store migration v3→v4 is additive; the old release
refuses a v4 store, so rollback restores the backup); `conversations.json`
gains `discussions`; the executor needs no restart (reconcile runs the new
`wfe`); aghangout `scripts/deploy.sh`. No outstanding conversation turns may
be pending at cut-over (checked with `wfe conversation list` / `wfe observe`).

## Verification cases (Step 2 tests)

1. A discussion's first message creates it with members; same token twice = one message.
2. Unaddressed message → one facilitator invitation and run; addressed → only the named participants.
3. Invitation dispatch repeated (crash between request and record) finds the same run.
4. Contribution from a run is attributed to its participant, run, attempt and invitation; retried = one message.
5. Two concurrent contributions resolve their own invitations; continuation is recorded once, after both.
6. Unavailable participant: invitation `failed` with reason, visible; others unaffected; facilitator told.
7. A run that ends without contributing → `unanswered`; a later contribution → late.
8. Membership: invite of a non-member records a join; a left member cannot be invited.
9. Close withdraws pending invitations, keeps started ones, suppresses continuation; reopen replays nothing.
10. A participant run's `invite` is refused; a conversation route refuses a discussion and vice versa.
11. Archive: export twice = no duplicate records; interrupted after push → next export adopts the commit; concurrent discussions both published; a fresh clone reads both.
