# p17 step 6 — deployment, trials, the waiting example

Plan: [plan.md](plan.md), section 6. Machine-specific details (backup
path, logs, recovery commands) are in the ignored
`pj-agdev/.local/p17/notes.md`.

## Deployment

| What | Before | After | How verified |
| --- | --- | --- | --- |
| WFE (agdevworld) | `ab342dd` | `4fd38ce` → `7a0c181` → `36c6613` | `wfe-release.sh` check and health: the service and `bin/wfe` run `36c6613`; the served bundle contains the review page |
| agautolab wfrun guide | `15258d7` | `cb83634` | live checkout; read at every claim |
| pj-ponderer guide | `949022d` | `46643ec` → `4b0bdb9` | pushed to Gitea (rebased onto origin first: the local checkout was 88 archive commits behind) |
| aghangout | `3fa9a20` | `ad1713d` → `f092b8d` | `deploy.sh status`; the served bundle contains レビューを開始 |
| agpc worker pins | wfe `ab342dd`, executor `15258d7` | wfe `36c6613`, executor `cb83634` | Nautobot placement `agautolab-agpc` via `nctl desired apply`, `nctl render production`, `setup_wfe_worker.yml` (ok=22); on agpc `git rev-parse` matches and `wfe-worker` is active; `nctl drift --host agpc` converged |
| Live node catalog | — | `human-review` r1 | `wfe nodes list` |

- State was read first through nctl: status ok, agpc converged. Both
  workers were idle and WFE was on `ab342dd`.
- Backup before the first release: every sqlite store, the catalog,
  `present.json` and `workers.json`. The present store goes from v1 to v2,
  and `ab342dd` refuses v2. The notes give both ways back (restore the
  store, or drop the two new tables), and how to find run records that use
  `review.*`.
- Every registered run (355) reads under the new release (step 5).

## Trial A — the whole flow with a check actor

Conversation `p17-trial-a`, category check. The person is the identified
check actor `p17-check`. The request was a figure of a sine wave with its
2nd and 3rd harmonics, from pj-misc.

| Time (JST) | What happened | Whose move |
| --- | --- | --- |
| 22:54 | brief #367; ponderer/conduct/run-038 announced "I will look at it, then hand it to you as a human review" (#368) and delegated to misc/freework | autonomous |
| 22:56–22:59 | the child run-008 drew it, presented p1, asked its review question, got ponderer's answer, stopped p1, completed | autonomous |
| 23:00 | ponderer judged it `satisfied`, added `human-review-figure`, prepared rv1 (Japanese summary, subject `d1` → run-008 p1), announced it in the room (#369: "awaits your review … Start the review with the button in the room"); its attempt ended **exited, no hold, no job queued**; status "Work complete · Awaiting human review" | autonomous |
| 23:01 | in the room, **レビューを開始** opened the review page and started the stopped page of the completed child (instance on :8300); from agpc (another LAN host) the identity, the review screen and the review API answered | check actor (browser) |
| 23:01 | on the review page, **Request changes**: 「線をもう少し太くして、背景に薄いグリッド線を入れてください。」; recorded as `p17-check` (synthetic check actor); the instance was handed to ponderer-agent; exactly one continuation was claimed at once | check actor (browser) |
| 23:02–23:07 | ponderer took the changes up and reopened the **same child** with `--follow-up rv1`; the `satisfied` disposition is kept in `reopened`. run-008 received correction c1, presented p2 (thicker lines, grid) and completed again. ponderer judged it, added `human-review-figure-2`, and prepared rv2 (`previous: rv1`, subject p2) | autonomous |
| 23:12 | room button again; p2 served on :8301 and answered from agpc | check actor |
| 23:13 | **Approve** on the review page; cleanup stopped and verified p2's instance **and** p1's instance (rv1 had handed it to the agent; the approval of the answering round stops it); misc's p1 and p2 are recorded `stopped`; run-038 completed | check actor, then WFE |

- Screenshots are in the ignored `pj-agdev/.local/p17/trial/shots-a/`
  and `shots-a2/`.
- The trial script (`room.mjs`) clicks the real buttons and rewrites the
  `by` of those writes to `p17-check`, marking decisions synthetic. The
  room would otherwise post under the area's person (the Developer), and
  nothing was recorded under the Developer's name.

**Found and fixed during trial A**:
- The review page's header was dark text on the dark page (`7a0c181`).
- A presentation served for a review still told the person to answer in
  the child's Zulip request topic. Its panel now points to the round's
  review page (`36c6613`, with a test).

## Trial B — the example left for the Developer

Conversation `p17-example`, an ordinary room. It was requested by "Omni
Agent (for Developer)", the identified actor used for such requests since
p15. The request was a golden rectangle split into squares with the
Fibonacci spiral.

- **What ponderer did unprompted.** It created its delegate node with
  `accepts: person`. When misc/freework/run-009 asked for confirmation with
  its page (p1), ponderer relayed that question live to the room (#371,
  "回答までページは配信し続けます") and waited. The child and its page
  waited for a person.
  - That is the fix6 relay, and it is valid.
  - It is not the deferred review the plan asks for. The conduct guide said
    what to do after the result came back, not what to do with the child's
    confirmation question.
- **Guide fix** (pj-ponderer `4b0bdb9`): a child's confirmation of a
  viewable result is answered after ponderer's own look. The child finishes
  and stops its page, and the person's review is the human review
  afterwards. Questions about intent or scope stay the person's. The fix
  cites this trial.
- **Controlled redirect, not autonomous.** run-039's copy keeps the guide
  it was created with, so Omni, as the room's requester, replied to #371
  (#372): do not wait, answer the child yourself, finish it, and prepare a
  human review for the Developer.
- **What followed was ponderer's own:** it answered the child, run-009
  completed and stopped p1, and ponderer prepared rv1 with a summary of
  what to check.
- aghangout showed the review wait as "Pondererが対応中 … Work complete ·
  Awaiting human review", and an ask naming nobody as "Pondererがに尋ねています".
  Both were fixed in aghangout `f092b8d`.

## The example awaiting the Developer

- Room: `http://agstudio.local:8093/hangout/?conv=p17-example`. Its line
  reads 作業完了 · 人のレビュー待ち（rv1：Golden rectangle + Fibonacci
  spiral）. Press **レビューを開始**.
- Review page (the same thing directly):
  `http://agstudio.local:8093/wfe/#/ws/ponderer/review/conduct/run-039/rv1`.
- State now:
  - ponderer/conduct/run-039 reads "Work complete · Awaiting human review",
    with no attempt, no hold and no job;
  - the child misc/freework/run-009 is completed and its page p1 is
    stopped;
  - nothing serves it until the button is pressed.
- What to check:
  - one press opens the review page and starts the page;
  - the page answers from your PC;
  - the summary says what was made and what to check;
  - Approve (or Request changes with a comment) is recorded under your
    name, and the page stops afterwards.
  - A request for changes continues ponderer in this room and ends in a
    new review.

This waiting example is the product behaviour itself, not an approval gate:
the automated tests and the check actor in trial A have already exercised
approval, requested changes and cleanup on this deployment.

## Autonomous versus controlled, in short

| | Autonomous (agents alone) | Controlled |
| --- | --- | --- |
| Trial A | announcement of deferred review, own review, page stopped, rv1 prepared and announced, attempt ended normally, follow-up to the same child, rv2 prepared | the check actor's start, change request and approval (identified, synthetic) |
| Trial B | answering the child after the redirect, rv1 prepared | the live relay happened unprompted (guide gap); the redirect #372 by Omni as the room's requester |
