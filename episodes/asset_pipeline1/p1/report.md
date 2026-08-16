# Phase report — asset_pipeline1 p1

**Reconciled.** The braindump asked for three things: widen the autolab
planning context and give it its own agent, teach the run- flow to order
asset-labelled work from agforge instead of coding it blind, and let the two
bots talk when agforge has a question. All three run live on agstudio.

Suites: `agautolab` 117 passed (was 78), `agforge` 173 passed (was 159).
Repos: `agautolab` `fc6ae66`, `agforge` `8806b58`, both on GitHub.

## Step reports

| | | |
|---|---|---|
| [1](report1.md) | Repair stale guide references | every serving died on `GuideError`; tests never noticed because they monkeypatch `GUIDES` |
| [2](report2.md) | superdirector role and mission-flow rework | planning moves into the persistent project folder; `[Asset]` → the `asset` label |
| [3](report3.md) | aesthetics stopgap | one committed line of art direction, presence-guarded |
| [4](report4.md) | question.flag mention, S3 key, `/api/resign` | the two devices that keep an answer from being lost |
| [5](report5.md) | asset state machine on run- topics | three states, ledger in Plane, re-sign right before the run |
| [6](report6.md) | answer agforge on create- topics | the mention gate, which is the loop breaker |
| [7](report7.md) | end-to-end check and deploy | the live evidence, and what it cost |

## The shape that emerged

Three decisions did most of the work:

**Plane is the ledger, the filesystem is scratch.** Step 2 deletes `plan.md`
and `task[N].md` the moment Plane accepts them, and everything downstream reads
Plane instead: the `asset` label survives the deleted `[Asset]` marker (Step 5),
and Step 6 recovers `plan.md` and `task.md` from the parent and child issues.
A wiped `.local/` cannot make autolab order the same asset twice, or forget an
order it placed.

**The durable half of a delivery is the key, not the URL.** A presigned URL
lives 60 minutes; a coding run may take 1200 s and start at an unknown point
in that hour. So every delivery carries `[S3KEY] <key>` in both the chat post
and the Plane comment, and autolab re-signs through `/api/resign` in the
statement immediately before launching the run.

**The mention gate is an asymmetry, deliberately.** agforge answers any
`create-` post that is not its own. If autolab did the same, the two would
answer each other forever at one paid run per lap. Autolab reacting only to a
mention is what makes that loop terminate — and agforge's `question.flag` is
what produces the mention.

## Live evidence, in one line

A mission became a plan, the plan produced an `[Asset]` task, the task became
an `asset`-labelled Plane Work, `run-` ordered the asset instead of coding it,
the ledger held across four separate triggers, agforge generated a 384×384
transparent pixel-art hero in the project's own declared art style, the
delivery carried its key, autolab re-signed it and handed the URL to a coding
run that placed the sprite in the repository, and a follow-up question from
Forge was answered by a superdirector that had read the project's real CSS.

## Known, accepted, not fixed

- An asset work blocks the queue behind it while its asset is made.
- No director check on a delivered asset; the coding run is asked to
  compromise or fail.
- `runcreate-` is fired by hand by the Omni Agent.
- The mention gate reads only the *last* message, so a post landing after
  agforge's mention hides it permanently (observed live — see report7).
- A planning round that fails after the superdirector ran leaves its task
  files, which a later round with fewer tasks would re-register (report2).
- `ansible-playbook --limit agstudio` cannot run: the `claude_code_agent` role
  requires a user-scoped Node toolchain this Mac does not have. Unrelated to
  this phase, and left for the developer. agstudio's checkout is the live
  working tree and is already on `fc6ae66`.
