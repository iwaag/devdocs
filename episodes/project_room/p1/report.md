# Project Room p1 — phase report

Source: [braindump.md](braindump.md) · plan: [plan.md](plan.md) ·
steps: [1](report1.md) [2](report2.md) [3](report3.md) [4](report4.md) [5](report5.md).

## What was delivered

A game-style room, `agdevworld` `/?view=project`, for following projects
and studies from purpose through plans, runs and results, and talking
directly in the selected plan or run — beside the Operation Room (who owes
a reply) and the Arguing Room (where desires are developed).

- **Read model on the relay** (`agentroom/projectroom.py`, `GET /projects`,
  `GET /projects/<key>`): every `pj-` channel, kind read from its
  description or `unknown`, `goal` / `researchplan-` documents,
  `workplan-setup-` setup that plans no mission, missions and tasks by
  their anchor ids (`[mission]`, `[task]`, `[doc]`, `[state]`,
  `[replaces]`), recorded work state beside the ops engine's reply state,
  task counts with an explicit *incomplete* when a work channel is not
  mirrored, links found in the source, mirror health and Zulip narrow URLs.
  Gaps are named, not filled: setup only; a research plan is not work; no
  document; archived; resolved without a recorded end.
- **Talking** (`agentroom/projecttalk.py`, `GET|POST /work/<anchor>[/post]`,
  `GET|POST /projects/<key>/topics/<topic>[/post]`): the conversation's
  history as written, the destination and responsible agent labelled, plan
  comments into the planning conversation and run comments into the
  execution conversation, located from the anchor at send time; documents
  refused with the path to Front; ✔'d targets resumed only on an explicit
  `resume: true`; one post per token, uncertain never retried.
- **The room** (`src/projectRoom.ts`, `src/scenes/ProjectRoomScene.ts`,
  `src/projectState.ts`, `src/projectDemo.ts`): four board panels over the
  settings background and the responsible agent's portrait; selection in
  the URL; browser-local read positions (the unread dot) and per-conversation
  drafts; the rooms' IME composer; refresh that keeps selection, scroll and
  draft; a narrow layout; a demo fixture.
- **Backgrounds**: `[rooms.project]` registered in the settings repository
  (`47d5d713fab1`); one shared tint (`0x0d0f14` at 0.32) over every
  image-backed room.
- **Deployed**: relay kickstarted (queue resumed, no sweep), web image
  rebuilt, docs updated.

## Checks

323 relay tests (22 new, mocked poster, fixtures for every failure case in
the plan); `npm run build`; browser flows on the demo fixture and reads on
live data through the deployed relay and web image; 60 board reads cost
the mirror 0 Zulip calls.

## Remaining gaps

- **Live posting validation is pending**: no comment was posted to the
  realm through the new door (it buys an autolab run and the plan asks for
  an explicitly authorized test conversation). Everything up to the send is
  validated with a mocked poster and the demo.
- The room shows original speech only; Front's rendering worker is not
  extended to project conversations (a later option, as the plan says).
- Archived `work-` channels are not hydrated on a board read, so
  pre-`refactor` missions show no tasks and say so.
- `pj-mediagen` m6770 reads `resolved-unfinished` because its acceptance
  was written in prose and the topic resolved by hand — a realm fact the
  room now makes visible, and a case for accepting through the completion
  door.
