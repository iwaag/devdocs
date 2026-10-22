# Challenge p1 — final report

## Outcome

Problems found during work can now be kept as **challenges**: durable,
shared records in one Git repository (Gitea `developer/pj-challenges`),
recorded from a conversation with little effort, found from any project,
and carried into later conversations, discussions and requests at the
revision that was meant. Completing the original work resolves none of them.

| Part | Where | Delivered |
| --- | --- | --- |
| Store and contract | agdevworld `workflow_editor/server/challenges.ts`, `docs/challenges.md` | journal + serialized writer clone; Git authoritative; revisions, pinned reads; conflicts keep drafts; unsynced is never reported as published; crash-after-push recognised; derived index rebuilt from a clone; backup mirrors |
| API and CLI | `server/challengeApi.ts`, `cli/challenge.ts` | equal HTTP and `wfe challenge` operations (list/search, show, history, evidence, create, edit, resolve, withdraw, reopen, journal, status, reindex) |
| Into work | conversation store v11, `runs.ts`, `exec.ts`, `discussions.ts` | attachments on chat and discussion messages and on requests; the receiving run's input and folder hold the pinned revision; replies to working runs carry references; archives keep them |
| UI | aghangout `app/src/challenges.ts` | 課題 tab (chat, discussion) and page: cross-project list, record view, editor, record-from-message, attach to draft, start a discussion with it |
| Guidance | agautolab wfrun guide, pj-front guide | facts, no mandatory steps |
| First record | `ch-0001` | the gimmickimg edge-remnant feedback with its evidence; the context block points to it |

Releases: WFE agdevworld `aadee43` (from `25c74c4`), aghangout `6e311e4`
(from `b28f432`), agautolab `227ef1c`, pj-front `cfca3e5`; the agpc
execution node pins wfe `aadee43` and executor `227ef1c`. Step reports:
`report1.md` … `report6.md`.

## Completion criteria

- **Recorded from a conversation without a questionnaire** — 「課題に記録」
  on any chat or discussion message opens title / projects / state / free
  body, prefilled with the quote and its source; nothing else is asked.
  Attribution, time, run and source are recorded by WFE (step 4, deployed
  check step 6).
- **A successful publication is in the shared remote repository, with
  history and essential evidence** — only `published` (with its commit) is
  reported as recorded; `ch-0001` is commit `8176fd6` with four evidence
  files; history is the repository's log (steps 1, 5, 6).
- **Another project can discover and read it through the same API/CLI and
  UI** — forge's room listed and opened gimmickimg's `ch-0001`; `wfe
  challenge show` works on the coordinator and, forwarded, on the agpc
  node (step 6).
- **Chat and argue attach it, and the receiving run sees the recorded
  revision** — `forge/converse/run-008` and `forge/discuss/run-013` each
  received `ch-0001@8176fd6` in `input.attachments` and their folders, and
  answered naming it (step 6); unit checks cover later edits not changing
  earlier turns (step 3).
- **The first real challenge is preserved with source and evidence; its
  project context points to it** — `ch-0001` (source conversation
  `20261022-230244-zlcew6` #310, evidence from `freework/run-006`);
  gimmickimg `improvement-feedback` revision 2 is a pointer (step 5).
- **The deployed LAN UI supports the whole flow with few operations; the
  report includes evidence and recovery** — the flow was driven on the
  deployed UI and re-read from another LAN computer (12/12); backup mirror,
  recovery drill and rollback are recorded (step 6; commands in the ignored
  local notes).

## For the Developer to try

- `http://<host>:8093/hangout/` → lobby 「課題を見る」 → `ch-0001`.
- In any room, the 課題 tab; 「課題に記録」 on a line; 「添付」 then send.
- `wfe challenge help`.

## Notes and limits

- Store v11: rolling WFE back needs the pre-deployment backup; rolling
  forward is preferred.
- Evidence through the browser route is limited by nginx's default body
  size; the CLI has no such cap. Gitea has LFS disabled: ≤ 32 MiB per file,
  larger material belongs in a durable store linked from the body.
- Direct conversations have no archive export yet.
- Fixture records and trials remain visible: `ch-0002` (resolved fixture),
  the closed check conversation and discussion by "Omni Agent (challenge p1
  check)".

## Next phase

Unifying chat and argue into one conversation model with optional
facilitation (plan, *Subsequent phase*) can reuse `ChallengePanel` and the
`attachments` records as they are: both kinds already share the message
table and the attachment table.
