# Handover — finer ways to interrupt a run

Written 2026-10-13 at the close of project_migration p2. The Developer asked whether a run can now be interrupted finely: stopped with its plan kept, or rolled back and abandoned. It cannot. This note says what exists, what was seen, and what is missing, for whoever takes it up in the workflow editor (agdevworld `workflow_editor/`, released `783cfcb`).

## What exists today

Contract: `workflow_editor/docs/runs.md` (*Holds*, *One slot, one workspace*, the operations table). Code: `server/exec.ts` (`holder`, `stop`, `resume`, `cancel`).

| Operation | What it does | What it does not do |
| --- | --- | --- |
| `wfe run stop` | Ends the open attempt and drops queued jobs. The run is **held**; nothing begins until `resume`. | It does not release the workspace. |
| `wfe run resume --reason` | A person's instruction, queued for the next attempt. | — |
| `wfe run cancel [<node>]` | Final. Every pending, running or waiting node becomes `cancelled`, queued jobs are dropped, and presentations are stopped. | It does not touch the run's commits or files. |
| (none) | — | No rollback of what a run published. No way to keep a plan and free the workspace. No way to start a new run from a stopped or cancelled run's plan. |

**The workspace hold.** `ExecStore.holder()` treats every run of the executor that has begun and is neither `completed` nor `cancelled` as the workspace's holder. `request()` then refuses any other request there with 409: "Finish or cancel it, or request the run in another workspace". A held run therefore blocks the whole project workspace. That covers a run stopped on purpose, before any work, with a clean tree.

**Publication.** Every attempt's end publishes a checkpoint commit on the root's `main` and pushes it, for example `experiment/run-001: after attempt …: stopped`. Whatever a run changed in `lab/`, `study/` or a submodule is already on `main` when it is stopped or cancelled.

## What was seen (pj-jevlab, 2026-10-13)

- `experiment/run-001` wrote its plan and asked q1. The Developer answered in the browser: 「ちょっと事情が変わりました、この実験を保留としてください。」
- The agent took the answer up and recorded `run.stop` itself. The run is held (stopped) before plan agreement, with no work done. Its only published changes are `plan.md` and `run.json` (`7343f59`, `b3599ae`).
- The jevlab workspace is now refused to every other request: study, improve-tools and freework alike. The only ways out are resuming it to completion or cancelling it, and the Developer wanted neither.
- **Attribution.** The stop is recorded as `by: iwaag, via: cli`. The person asked for the hold, but the agent's process recorded it. Nothing in the record says that the agent acted on the answer: the `--by` default rules let an agent write in the person's name.

## What is missing

These are candidate behaviours. Each needs a design decision; none has been agreed.

1. **Park: keep the plan, free the workspace.**
   - A held state that does not hold the workspace, because no attempt is open and nothing waits on the executor.
   - Open question: is a clean tree enough? A parked run's later resume would find other runs' commits on top of its own. The contract has to say whether that is allowed, and whether the plan is re-agreed on resume.
2. **Abandon with rollback.**
   - Cancel plus reverting what the run published: the run's checkpoint commits on the root, and the gitlinks it moved in submodules.
   - The run record stays and says what was reverted. A revert is a new commit; history is never rewritten, since everything is public and pushed.
   - Open questions: what to do when a person's commit or another run's commit touched the same files afterwards, and whether submodule repositories are reverted too.
3. **Restart from a plan.** A new run created from a stopped or cancelled run's agreed (or unagreed) `plan.md`, referring to the original. The original stays as it is.
4. **Who stopped it.** When an agent records a stop, resume or cancel because of a person's answer, the record should name both the agent and the answer it acted on. Example: `by: jevlab-agent`, `because: q1 answer 0 by iwaag`, instead of `by: iwaag via cli`.
5. **The vocabulary in the person's words.** "保留" (hold) mapped to `stop`, which blocks the workspace. The browser and the guides should offer the choices by effect: hold and keep the workspace, park and free it, abandon, abandon and roll back.

## Current state to start from

- jevlab workspace (`pj-agdev/.local/agdev/pj-jevlab`): `experiment/run-001` is held (stopped) before plan agreement, and blocks other jevlab requests. The Developer has not decided between resuming and cancelling it. It is a natural first test case for *park*.
- Other workspaces (rts-vs-bot, ppt, agsfx) are unaffected.
- The run guide that agents follow is `agautolab/agent/guides/wfrun/guide.md`, read from the working tree per attempt.
