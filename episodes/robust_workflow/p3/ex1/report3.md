# Step 3 report — integrating accepted changes, and recording what happened

Plan: [plan.md](plan.md) step 3. 2026-09-24 (JST), by the Omni Agent. Code: agautolab `ec0b065`
(integration, with step 2), `09615fb` (introduction, guides, docstrings), `d06bacb` (a real-git
recovery fixture). Written, tested and pushed; deployed in step 4.

## Integration is bound to the existing authorization

The task agreement is the requester's post, and since p3 the code checks that it is in the serving's
input. It already authorized "commit" and the close-out's push. It now authorizes exactly this:

1. **Bind.** Everything in the mission's copy is committed (the requester reviewed the copy, not a
   subset of its commits), and `pending_changes` names each repository's commit that the shared
   branch lacks. The note `[selfnote][change] accepted main=<commit> … #<the requester's post>
   +gen=<serving>` is written **before** anything shared moves. Branch names are never the
   identity: the note carries commit ids.
2. **Integrate** (`missionspace.integrate`, under the project lock, all or nothing):

   | Shared branch | Result |
   |---|---|
   | Already contains the accepted commit | `already`. Nothing moves, whatever the retry |
   | Unmoved since the mission's base | fast-forward |
   | Moved, and the other side touched **different** files | merged (`commit-tree` of `merge-tree`'s tree). Both reviewed changes are kept, and neither touched the other |
   | Moved, and the **same file** changed on both sides | `overlap`, or `conflict` when git cannot merge it. **Returned**: nothing moves, and the task stays open |
   | Project folder has unowned uncommitted changes to files the result would touch | `blocked`, returned the same way |

   For a repository autolab publishes (`main`, `direction`, `devlog`, or one named in
   `publish.flag`), the push *is* the integration. It is a plain push of the result onto the remote
   branch, which a concurrent update rejects. The operation then fetches and decides again, up to
   three times, and never forces. The project folder's checkout is fast-forwarded only after the push
   lands, and local commits the remote lacks (a planning note, a failed earlier push) are published
   first.
3. **Record.** `[change] integrated main=<result>/<how>/<published>`, then the result post and
   `[state] completed` (the task is **complete**), the devlog record (its own folder only), a refresh
   of the copy onto the combined result, and the next task's start. When the last task closes, the
   copy is released.

The **mission acceptance** (`agentchat accept`, `accept.flag`, the door) is unchanged. It refuses while
a task is open, and a task is open until its change is integrated, so a recorded `done` never has
integration outstanding. No approval round was added. A returned change needs the requester's
decision only because the result they would accept is new: the combined work.

## Distinct outcomes, in the existing record

| Outcome | Where it is written |
|---|---|
| checkpointed | Commits on `autolab/m<id>`, and `[change] checkpoint <repo>=<head>:<tree>` after each serving that changed the copy (the exact content, dirty files included) |
| accepted | `[change] accepted …` with the requester's post |
| integrated / published | `[change] integrated <repo>=<commit>/<fast-forward\|merged\|already>/<pushed\|level\|local>` |
| returned | `[change] returned <repo>=<overlap\|conflict\|blocked> +files=…` |
| complete (task) | `## Result` + `[state] completed` (as before) |
| done (mission) | `[selfnote][acceptance]` + `[state] done` (as before) |

These are selfnotes in the task's own topic, which is the work record. No second ledger was added.
The close-out line also says whether the accepted content is what the requester saw: the last
checkpoint before their post, compared by tree. It never gates. The worker is told not to write
`report.md` when it changed anything beyond what the agreement asked for.

## Retries and interruption

- **A crash after the push, before the record.** The next serving of the topic finds the newest
  change note is `accepted` or `integrated` with the task still open. It finishes the close-out from
  that note: no supercoder run, integration recognized as `already`, and the report read from the
  serving named by `+gen`. A `returned` note or a newer agreement ends that state. Fixture
  `test_a_close_out_cut_after_the_push_finishes_once_on_retry` on real git: the first close-out dies
  at the record, the remote holds `work` once, and the retry records the task once. The remote log
  is unchanged.
- **A repeated "done" on a closed task.** The run happens (the post buys it). No second result,
  integration or start follows, and anything changed in the copy since stays on the branch until a
  re-plan gives the mission a task for it.
- **A repeated integration.** The ancestry check makes it `already` / `level`.

## Messages and documents changed together

- **Close-out lines**:
  - "integrated into the project: main 1a2b3c4d5e (fast-forward) and published".
  - "… (merged beside newer work)".
  - "(kept local: autolab does not publish it)".
  - "task N … is not closed: its accepted change cannot be integrated as it is — main: main moved
    since this mission began and changed the same files (wordcount.py). Nothing was integrated or
    published. Bring the shared branch into this mission's copy (`git merge <branch>` …), check the
    combined result, and show it".
  - The not-agreed line now adds "Its work is checkpointed on autolab/m<id>; nothing is
    integrated".
- **Introduction** (`params/intro.md`, re-posted at deployment):
  - Each mission works in its own copy, and a commit there is a checkpoint.
  - Closing a task takes exactly what was seen into the project and publishes it.
  - A same-file overlap comes back for a combined review.
  - A second "done" changes nothing.
  - Cancelling keeps the work on its branch.
  - Removed: "'yes, commit it' answers the question I asked", which no longer means anything.
- **Worker guide**:
  - Commit whenever it helps. It cannot push and never needs to.
  - Everything in the copy is what is accepted, so scratch goes elsewhere.
  - No `report.md` after unrequested changes.
  - `publish.flag` names other repositories.
  - How to combine a returned change.
  - Removed: "commit changes … after getting the developer's approval".
- **Planning guide**: planning works in the project folder of integrated work, and what it writes
  there is committed as its notes.
- **Removed code**: `push_main` / `push_main_repository` and their tests. Publication is the
  integration's push now.
- **Developer docs**: `devdocs/README_DEV.md`, autolab section.

## Verification (fixtures)

- `test_missionspace.py`, 21 cases on real git:
  - fast-forward and publish once, with a retry that is `already`/`level`;
  - disjoint files merged when the target moved;
  - the same file returned as `conflict` or `overlap`, with nothing moved;
  - the returned mission merging the shared branch and then fast-forwarding;
  - a remote moved by another clone, fetched and merged;
  - unowned dirt blocking only where it would be overwritten;
  - one refused repository moving no repository;
  - a non-published repository integrated locally, and published when named;
  - refresh onto the combined result;
  - the interrupted close-out.
- `test_zulip_listener.py` (mocked realm):
  - the bind comes before integrate, which comes before the record, with the exact note texts;
  - `publish.flag`;
  - returned leaves the task open, with no result and no resolve;
  - the interrupted close-out is finished without a run;
  - `returned` is not resumed;
  - a completed task is not closed again;
  - a checkpoint is written only on change;
  - the reviewed or changed-after-review line;
  - the last close-out releases the copy;
  - a run that changed nothing integrates nothing.
- Suite: **292 passed**.
