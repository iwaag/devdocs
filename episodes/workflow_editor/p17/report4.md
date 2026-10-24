# p17 step 4 — the one-button review experience

Plan: [plan.md](plan.md), section 4. Code: agdevworld `57ca0c4`
(`workflow_editor/src/reviewView.ts`, `src/runSide.ts`, `src/runCommon.ts`,
`src/canvas.ts`, `src/workflowView.ts`, `server/conversations.ts`,
`server/conversationService.ts`, `server/notify.ts`); pj-aghangout
`ad1713d` (`app/src/room.ts`, `app/src/api.ts`).

## Where the person meets a review

| Place | What it shows | Action |
| --- | --- | --- |
| WFE review page `#/ws/<ws>/review/<wf>/<run>/<rv>` | summary first; each subject with its owning run, revision, state now, the live link (opens in its own tab, on its own origin), newer presentations of its run beside it, other rounds using the same instance; the decision, cleanup and continuation; other rounds of the run | Start review (again), Approve, Request changes (feedback required), with "Recording as" |
| Run view focus card | the status ("Work complete · Awaiting human review", "… Approved · cleanup pending", "Changes requested …") | a primary "Start review (rv1)" / "Continue the review" link to the page |
| Graph | a `review` node card (green, eye icon) labelled with the round's state | — |
| Editor palette and inspector | *Human review* as a node type, with its hint | — |
| aghangout room | "作業完了 · 人のレビュー待ち（rv1：<title>）"; while starting, approved, or changes requested, a line each | **レビューを開始**: opens the review page in a new tab and posts Start review (one action) |
| Zulip run topic (and request topic) | **Work complete · Awaiting human review** with the summary and `Start review: <page>`; the decision; a cleanup failure | — |
| Conversation API | `reviews` (each serving run's rounds: state, summary, subjects, decision, `page`, `api`) | — |

- **Isolation.** Nothing served is framed by the editor: each subject opens
  the presentation's own review screen on its own origin, as before. The
  page only reads the round and posts the person's actions.
- **Coming back.** The page re-reads the round every 8 s (1.5 s while a
  start or a cleanup is under way). Links are resolved from the host's
  state at each read, never from an old address.
  - A `reviewing` round whose instance went away shows "not serving now —
    Start review again".
  - Approval is shown the moment it is recorded, and cleanup has its own
    line (pending, failed with the retry promise, verified).
  - Other rounds are listed apart and linked.
- **No implicit decisions.** Loading the page, starting a review or a page
  stopping records nothing but the start. Approve sends the round's
  revisions, so a page left over from another round is refused
  (`review-superseded`). The generic run `ops` route refuses `review.*`.
- **Synthetic actors.** These are recorded only from the CLI (`--synthetic`)
  and are labelled everywhere (review page, `wfe review show`,
  assessments).
- Situation words for `awaiting-review` were added to every map that has
  one (run view executor line, child words, status).

## Checks

- WFE `npm run check` passes: 584/584, tsc, both Vite builds.
- aghangout `npm run check` passes: tsc, Vite build, 31/31 unit tests.
- No browser was driven at this step. The page, the focus card and the
  room button are checked on the deployed release from another LAN host
  in step 6, with screenshots. Until then they are only built and typed.

## Known limits

- aghangout shows the start progress and the outcome as status lines; the
  pages and the decision are on WFE's review page (one implementation).
- The room's button posts with the room's `person` as `by`, as the room's
  posts do.
