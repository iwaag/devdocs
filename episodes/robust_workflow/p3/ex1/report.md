# Robust workflow p3 ex1: isolated work and deliberate integration

Plan: [plan.md](plan.md). Steps:
[1](report1.md) contamination reproduced and the contract,
[2](report2.md) a working copy per mission,
[3](report3.md) integration and its record,
[4](report4.md) fixtures and live trials.
Executed 2026-09-24 (JST) by the Omni Agent. Observer stayed on its local model; Front and
autolab ran on Sonnet 5 through claude_code, unchanged.

## The contract

A mission works in **its own copy** of the project:
- `.local/missions/<slug>/m<id>/`, with one git worktree per project repository on branch
  `autolab/m<id>`;
- found again by the mission's id;
- pushes refused.

The project folder holds **integrated work only**. Planning runs there, and what a planning run
writes there is committed as its notes.

| Outcome | Meaning | Recorded as |
|---|---|---|
| checkpointed | Saved on the mission branch or in its copy. The worker commits whenever it likes. It is not acceptance and publishes nothing | Commits on `autolab/m<id>`; `[selfnote][change] checkpoint` after each serving that changed the copy |
| accepted | The requester agreed in the task topic (checked in code since p3). Everything in the copy is committed and bound to exact commits **before** anything shared moves | `[change] accepted <repo>=<commit> #<post>` |
| integrated / published | Under a per-project lock, all repositories or none: fast-forward if the shared branch has not moved, merge if the two sides touched different files, **returned** for a combined review if they touched the same file. `main`, `direction` and `devlog` are published by the push that integrates them (never forced); other repositories only when the worker names them in `publish.flag` | `[change] integrated …` or `[change] returned …` |
| complete (task) | Only after integration | `## Result` + `[state] completed` |
| done (mission) | The requester's recorded acceptance, unchanged from p3. It refuses while a task is open, so it never leaves integration outstanding | `[selfnote][acceptance]` + `[state] done` |

Rules that cover retries and endings:
- **Retries are safe.** Integration is recognized by commit ancestry. A close-out cut short is
  finished from its `accepted` note with no new run. A second "done" on a closed task changes
  nothing.
- **Cancel, replace, last close-out, project archive** release the copy. Its uncommitted work is
  committed to its branch, the worktrees are removed, and the branch is kept. A re-plan
  re-attaches it.
- **No approval round was added.** The task agreement that already authorized "commit" and the
  close-out's push now authorizes integrating exactly the agreed content. Only a combined result
  the requester has not seen comes back to them.

## Deployed

- **agautolab `3a67d63`** (pj-agdev pointer `agautolab 3a67d63`), listener and gateway restarted.
  The introduction was re-posted.
- All other components are unchanged from p3: pyagag `0ef3c61`, agfront `99ad106`, agobserver
  `74c881f`.
- `nctl drift` converged=46.

## Evidence

**Fixtures:** 293 tests (236 before), 21 of them on real git.

**Live, on the fixed revision:** seven missions through Front:
- B∥C and D∥E, the same-file conflict in both finishing orders;
- F, cancelled with a checkpoint and dirty files;
- G, two tasks, with a real process crash between the push and the record, plus a duplicate
  acceptance;
- A, the normal two-task cycle with a listener restart after a checkpoint.

Results:
- **No cross-mission contamination.** Each copy held only its own edits throughout, and every
  `main` commit is an accepted change or its combined merge.
- **No cancelled work integrated.** F's work exists only on `autolab/m10859`.
- **No accepted work lost.** All six options are on `main` once, and the suite passes.
- **No duplicate integration.** The crash retry reads `already/level`, and there is one result
  and one devlog record per task.
- **No false completion.** Every task was `completed` only after `integrated`.
- Front recorded all six missions `done` unprompted, and Observer released all seven requests.
- No rescue, no unnecessary approval round, no wrong-scope change.
- 78 runs, $7.62.

## Is same-project parallel development ready?

**Yes, for practice.** Concurrent missions on one project no longer share files, an index or
publication. A same-file collision surfaces at integration as a returned change, and its
resolution goes through the worker and the requester's review of the combined result. Two live
rounds did that without help.

This is readiness to practice, not proven unattended reliability. The things that are unproven or
limited:
- **Disjoint-file merges and outside pushes are fixture-only.** No live mission merged beside
  another without a conflict, and nobody pushed to a project from outside during the trials.
- **Pattern projects** (several repositories, ignored generated data under `.local/`) have not
  run a mission on this code.
  - A copy does not carry ignored data. A run that needs another mission's raw outputs reads the
    project folder by path.
  - Repositories other than `main`/`direction`/`devlog` are published only when the worker writes
    `publish.flag`. Before this, workers pushed them themselves.
- **The project folder is protected by the prompt, not by permissions.** A run could still `cd`
  there and write. Integration then refuses only the files it would overwrite, and the reply says
  so.
- **A returned change costs a review round**, which is correct when both sides touched the same
  file. When they touch the same file in different places (a clean textual merge), it is still
  returned. That is deliberately conservative and can be relaxed on evidence.
- **The reviewed-state comparison is advisory.** The close-out says whether the accepted content
  is the last checkpoint the requester saw. The worker is told not to close after unrequested
  changes, but the code does not refuse.

**Temporary operating limit:** none needed for correctness. For a pattern project's first mission
on this code, check that its runs still find their generated inputs and that the repositories its
README says are pushed get named in `publish.flag`.

## Starting, cancelling and resuming missions

- **Starting:** unchanged. Ask Front, or open a `workplan-…` topic in `pj-<slug>`. "You may start"
  starts task 1, and the task's own copy is made on its first serving
  (`.local/missions/<slug>/m<id>/`).
- **Reviewing:**
  - The worker's commits are checkpoints on `autolab/m<id>`.
  - Nothing is on `main` until you say the task is done. That agreement integrates exactly what
    the copy holds and publishes it.
  - If the reply says the change was **returned**, the worker merges `main` into its copy on your
    word, and you agree to the combined result.
- **Cancelling:** say so in the `workplan-…` topic. The tasks stop, the copy is released, and the
  work stays on `autolab/m<id>` for anyone who wants to look (`git log autolab/m<id>` in the
  project repository).
- **Resuming after a restart or crash:** nothing to do. A serving finds the mission's copy by its
  id. A close-out that was cut short is finished from its own note at the next serving, which the
  listener's journal requeues.
- **Looking at the record:** a task's `[selfnote][change]` notes. `agentchat read` hides
  selfnotes; the host notes say how to read them.
- **Faults for trials:** `agautolab/.local/faults/exit-after-integration` (one-shot), beside p1's
  `skip-next-start`.

## Next project request

None was supplied or selected during this extension, so none was invented. The next real request
goes through Front as usual. During real work, record these as improvement candidates:
- rescue follow-ups;
- missed completion records;
- change contamination;
- duplicate work;
- silent stalls;
- returned changes: how many, and whether a clean same-file merge would have been safe to take.

## Follow-ups (evidence-driven)

1. **Pattern project on the new code.** The first real mission there should confirm ignored-data
   access and `publish.flag` use, or show what is missing.
2. **Front relay accuracy.** After E's close-out, Front reported "Not integrated", carried over
   from its previous message, while the record said integrated and published. Watch whether this
   recurs before touching Front's guide.
3. **Startup replays.** About 11 old ✔ mentions are re-examined and ignored at every autolab
   restart. They cost nothing but log noise, because an ignored mention writes no receipt.
4. **Deferred as planned:** the Sonnet comparison (report6 triggers unchanged, none met), wider
   discovery, and queue identity by id.

## Omni Agent work for in-system agents

- **Requester's part in every trial:** requests, acceptances, the review of both combined
  results, the cancellation, and the duplicate acceptance. This was done for Front as the
  Developer's stand-in. That is the Developer's role, not a handoff candidate.
- **Operator fault injection** (the crash hook, the restart after a checkpoint) and the
  deployments. Operator trial work, not a handoff candidate.
- **Set aside one unattributable uncommitted file** in `mediagen/gentest-actionDatasets` at
  rollout (report2), for agent autolab. A one-time rollout step, not a handoff candidate.
- **Re-ran one mission acceptance** (`agentchat accept 11200`) with Front's credential, as the
  repeated-request check. It wrote nothing, and it was a trial probe, not a handoff candidate.
