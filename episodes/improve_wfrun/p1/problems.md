# improve_wfrun p1: problems found

Found by the Omni Agent on 2026-11-02 while investigating a question from the developer: "I added a
workflow to gimmickimg, so why doesn't it appear on the WFE project screen?" The question came from
the aghangout conversation `20261102-005359-ibkbjr` (#560–#563). Observations were made against WFE
`47a83cc` on the coordinator host. Unless stated otherwise, references are to that release's `docs/`
and `server/`.

## Background: how a run stores its work

`docs/storage.md` (p9) defines the following:

- Each run gets its own working copy at
  `<project workspace>/.local/wfe/runs/<workflow>/<run>/tree/`. The copy is a full clone, including
  submodules, on the run's own branch `wfe/<project>/<workflow>/<run>`.
- The copy starts from the workspace `HEAD`. If the workspace is only behind its upstream, the copy
  starts from the upstream tip after a best-effort fetch.
- A copy pushes only to its run branches. When the run is integrated, its results reach the remote
  default branch, never the workspace: "Nothing in the person's workspace is changed."
- The person's project workspace "stays theirs. Editing workflows, the project and its own Git state
  happens there."

## Problem 1: integrated results never reach the project workspace

Runs integrate into Gitea `main`, and nothing ever fast-forwards the registered workspace. Each
workspace therefore drifts further behind with every integrated run until someone pulls it by hand.

Evidence, collected 2026-11-02 after `git fetch` in every workspace. Every workspace was only behind
its remote (0 commits ahead):

| Workspace | Commits behind `origin/main` |
|---|---|
| pj-ponderer | 31 |
| pj-agops | 18 |
| pj-aghangout | 7 |
| pj-gimmickimg, pj-front | 6 |
| pj-forge, pj-protoprey | 5 |
| pj-aesthete, pj-agsfx, pj-curator | 3 |
| the other 10 | 0 |

Seven of these workspaces also had uncommitted changes: aesthete, agsfx, archsage, misc, ppt,
protoprey and rts-vs-bot. Those changes were not inspected.

pj-gimmickimg was at `1482d0e`. It was missing freework/run-017 (`3295e1e`…`6becd97`) and
freework/run-018 (`d6cc87d`…`d4d8ea9`). run-018 is the run that added
`devdocs/workflows/partial-animation.yaml`.

## Problem 2: what a workflow is depends on where you read it

Different readers use different sources, and nothing reconciles them:

| Reader | Source | gimmickimg workflows seen |
|---|---|---|
| `wfe projects show` (default) | Gitea default branch (`docs/access.md`: "committed and authoritative, even when a workspace here is behind") | freework, partial-animation |
| Editor project screen, `GET /wfe/api/workspaces/gimmickimg/workflows` | workspace working tree (`Workspace.workflowFiles()`, `server/workspace.ts`) | freework only |
| Starting a run | workspace: "The definition the run follows is still read from the project workspace" (`docs/storage.md`) | freework only |

Consequences:

- A workflow that a run creates is invisible in the editor. It also cannot be started, even though
  `wfe projects show` reports it as registered. That run-start refusal is inferred from the docs and
  from `server/runs.ts` (`no workflow "<id>" in devdocs/workflows`). It was not reproduced.
- When the developer asked, ponderer correctly suspected that the workspace was stale (#563), but it
  had no way to confirm what the UI showed and no permission to move the workspace.

## Problem 3: the UI does not show that a workspace is stale

The access API already computes `atCommitted` for each workspace (`server/access.ts`), and the CLI
prints "(not at the committed tip)". The editor's project screen shows nothing like this, so the only
symptom a person sees is a missing workflow.

## Problem 4: the delegated work ran under freework, not the new workflow

The conductor (ponderer/conduct/run-068) asked gimmickimg's freework to both define
`partial-animation` and rebuild the flower scene. Delegation d1 became freework/run-018. The scene
was produced inside freework, and `partial-animation` has never run (#561). ponderer's own account
of the cause: "my request was phrased as 'have freework make it'". This interacts with Problem 2:
even a correctly phrased follow-up would have failed to start the new workflow while the workspace
was stale.

## Problem 5: run copies and run branches are never cleaned up

`docs/storage.md` *Lifetime* says: "Copies are kept when a run is held, completed or cancelled.
Deleting a copy, deleting its branches and collecting garbage are not part of p9." Nothing was ever
added afterwards. A copy removed by hand leaves the run reported as missing.

Coordinator host, 2026-11-02: about 480 copies, and 75 GB under the project workspaces in total.

| Project | Copies | Size |
|---|---|---|
| gimmickimg | 18 | 22 GB |
| playprey | 15 | 20 GB (onboard run-001/002/004 ≈ 15 GB) |
| protoprey | 10 | 2.4 GB |
| jevlab | 12 | 2.3 GB |
| lowerpoly | 7 | 1.9 GB |
| forge | 44 | 1.6 GB |
| cagent | 36 | 967 MB |
| agops / front / ponderer | 101 / 72 / 72 | 151 / 295 / 476 MB |

Copies are large because each one is a full clone. For example, gimmickimg freework/run-018 is
1.9 GB, of which `.git` is 1.3 GB and `experiments/` is 1.7 GB.

Disk space is not yet a problem (52% used). Cleanup cannot simply delete copies, for two reasons:

- A finished run's record is read through its copy.
- Ignored outputs and environments exist only in the copy ("they stay in the copy, which is kept").

Outputs already published to the share (`wfe run publish`) live elsewhere and survive deletion.

Not checked:

- Worker copies on agpc (`docs/workers.md`). The ssh connection failed host key verification.
- How many run branches have accumulated on Gitea.

## Open directions (not decided)

- Fast-forward a workspace after integration when it is only behind and clean. Otherwise show that it
  is stale and offer a pull on the project screen.
- Alternatively, read workflow lists and run definitions from the committed default branch, and keep
  the workspace for editing only.
- Give conductors a way to delegate *to* a newly created workflow, or to verify it, once it is
  reachable.
- Retire finished run copies (and later their branches) without losing the run's record or
  unpublished outputs.
