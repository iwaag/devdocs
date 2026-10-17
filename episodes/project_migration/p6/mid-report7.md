# project_migration p6 — Step 7 mid-report: trials paused for review

Status: **paused** (2026-10-17 ~19:10 JST) at the Developer's request, before the playprey trials resume.
Both open playprey runs are held (`wfe run stop`), and nothing playprey-related is running. The NFS repair
that the cross-host trial needed is complete. Raw evidence is in the ignored
`pj-agdev/.local/project-migration-p6/step7/`.

## Decisions taken during this step

| When | Decision (Developer) | Effect |
| --- | --- | --- |
| ~13:50 | The Omni Agent answers the trial runs' questions, except the final review. No cost cap; report costs | Plan agreements answered by Omni |
| ~14:24 | The different-anatomy input is a model the Developer supplies: `shared://agshare/playprey/inputs/snake/` (one anaconda inner-mouth GLB). "No tongue and no cheeks, so construction should be easy" | Trial T2 |
| ~14:30 | "返答は全て私の代わりにしてacceptしてください" ("answer everything for me and accept") | Omni also gives final acceptance, including for cagent run-007 |
| ~18:40 | The Developer added "Omni Agent" to the pj-cagent approvers and allowed `setup_nfs_client.yml` | cagentctl accepts Omni's agreements and person-step confirmations |
| ~18:50 | "最終的にomni agentは全プロセスで人間と同権限のapproverにする" ("ultimately, make the Omni Agent an approver with the same authority as a human in every process") | Permanent. Not to be removed at phase close |

Every answer is recorded as the Omni Agent's, with the delegation cited. No answer was posted as the
Developer.

## Trials

| Trial | Run(s) | State | What it showed so far |
| --- | --- | --- | --- |
| T1 existing case | playprey `onboard/run-001` | **Rejected** (by Omni, delegated) | See "T1 in detail" below |
| T1 successor | `onboard/run-003` (predecessor run-001) | **Held**; `present` failed | See below. Its `present` failed because `delivery.py build` refuses uncommitted changes (open defect 4) |
| T2 different anatomy | `onboard/run-002` (snake → profile `anaconda`) | **Held** at `present` (its last attempt was interrupted when the executor exited at 08:06Z) | See "T2 in detail" below |
| T3 coexisting approaches | — | Not started | Planned as `compare`: eagle_beakfit vs eagle_beak2 |
| T4 cross-host delivery | — | Not started; **now possible** | The client mount on the Linux VM works again (run-007) |
| T5 resume and publication | Private fixture on the deployed WFE and the real share | **Done** | See "T5 in detail" below |
| T6 knowledge reuse | (run-003, run-002) | Partial evidence | run-003 reused run-001's findings by name. run-002 cited the eagle case's lipless/toothless lessons from `knowledge/onboarding.md`. A dedicated later run that cites the snake lesson is still to come |

### T1 in detail

- The rebuild was byte-identical: the GLB matched the recorded asset `632c3b97…`. Report1 had left this
  open.
- All checks passed.
- It was rejected because the run had committed four material files of 510–523 MiB each, about 2 GB, to
  its branch. Fast-forward integration would have put them into main's history.

### run-003 in detail

- From the fixed main, it gives the same byte-identical rebuild.
- The materials stay small (about 700 B each; empty diff).
- All checks pass, including persistence, because this time only one editor server was used.

### T2 in detail

- A real hidden assumption surfaced: every runtime check required a tongue part. The agent added a
  declared "tongue-absent" capability (validator, frames, guards) instead of a placeholder tongue. An
  actual engine launch found two more sites that a static audit had missed.
- No regression: all 9 profiles pass the profile check, and the eagle P2 suite passes.
- It records "no teeth either" (one closed shell), and the static-stage checks pass.
- `knowledge/cases/anaconda.md` is written, and onboarding.md has a "first tongueless case" section.

### T5 in detail

- A publish was killed mid-copy. It left only a hidden partial folder: nothing visible, nothing listed,
  nothing recorded.
- The retry published cleanly, and the same receipt was recorded once.
- An in-place input changed during a hold was recorded as `changed` on resume, while the staged copy stayed
  `verified`.
- The leftover partial folder was a defect; it is fixed (see the next section).

## Defects found and fixed during this step

| # | Defect | Found in | Fix |
| --- | --- | --- | --- |
| 1 | A clean playprey checkout never got the textures that Godot extracts from the GLBs: the project tracked 91 orphaned `*_Image_*.jpg.import` sidecars, which blocked extraction. Materials then loaded without textures. In a run, Godot re-saved them with about 2 GB of embedded images | T1 (my independent check of the agent's persistence failure) | playprey `45b3ceec`: stop tracking the sidecars, ignore them, and add `evidence/.gdignore`. Verified: one import gives 0 errors, and both the persistence check and the P2 suite pass |
| 2 | A publish or input staging killed mid-copy left its unfinished folder for good | T5 | WFE `a1b217f`: unfinished folders are named `<host>-<pid>` and swept by the next copy on that host |
| 3 | Nothing stopped a run branch from carrying huge files into Git | T1 | WFE `e19fa04`: a checkpoint leaves files over 50 MiB uncommitted and says so, and a branch whose unpublished commits carry one is not pushed. Deployed as the live release (379/379 checks) |
| 4 | **Open.** `tools/delivery.py build` refuses a dirty tree, but inside a run the agent's work is committed only at checkpoints. The agent failed `present` instead of committing first | run-003 | To decide on resume: either the guide and node text tell the agent to commit its work before building (the run branch is the agent's to commit to), or `delivery.py` records the run's working-tree state |
| 5 | The agent's persistence check timed out because it started a second editor server on :8765 (`tools/godotw` manages one) | T1 | Diagnosed by Omni; run-003 confirmed. A one-line note in the guide is a candidate |

## The NFS repair (cagent change/run-007, completed and integrated)

The Linux VM's mount hung again soon after Step 3's `noresvport` fix. run-007 found and handled the actual
mechanism:

1. **Mechanism.**
   - The Linux client closes its idle NFS connection after about 5 minutes. macOS nfsd never completes
     its side (CLOSE_WAIT), so the client hangs in FIN-WAIT-2 and cannot reconnect.
   - The Mac's 3600-second idle drop was not the trigger.
   - The failure reproduced within about 6 minutes of a fresh mount.
2. **Fix.**
   - A systemd automount idle timeout of 300 s on the client (`x-systemd.idle-timeout=300s`) makes the
     client unmount before it can be left half-closed.
   - The macOS `nfs.conf` setting from run-006 is now declared through the role.
   - The client role no longer blocks on a hung mount point.
   - Observation classifies a timed-out mount as unusable. This came from the develop child run-004,
     integrated as pj-clusterintent `fff9a4c`.
3. **Proof.**
   - After 360 s idle, the client had unmounted and held no connection.
   - After `nfsd restart`, probe, write, rename, readback and a 512 MiB hash passed.
   - Fresh observation and drift were converged.
   - My own check also passed: list, write, read-back and delete on the share from the client.
4. **Limits recorded.**
   - The 11 CLOSE_WAIT sockets on agstudio are not cleared by an nfsd restart. Whether they grow
     unbounded is untested.
   - The protection only works while the share is truly idle.
   - nctl emits no reconcile action when only a placement's config changes. That is why the first gate
     tested the old configuration, and why `setup_nfs_client.yml` had to be allowed and run directly.
5. **Friction.**
   - The run asked many re-agreement questions (each edit of plan.md needs fresh agreement).
   - cagentctl accepted only the configured approver until the Developer added the Omni Agent.

Person steps were run by the Omni Agent through Ansible with privilege escalation: two lazy unmounts, the
read-only gates and the nfsd restart. The record states this, after a correction I required before
accepting.

## Costs and time (attempts, as recorded by the executor)

| Run | Attempts | Cost | Attempt time |
| --- | ---: | ---: | ---: |
| playprey onboard/run-001 (T1, rejected) | 7 | $10.24 | 102 min |
| playprey onboard/run-002 (T2 snake, held) | 5 | $22.93 | 77 min |
| playprey onboard/run-003 (T1 successor, held) | 2 | $3.43 | 16 min |
| cagent change/run-007 + develop/run-004 | 27 | not reported (Codex, gpt-6-luna; the harness gives no dollar cost) | 59 min |
| **playprey total so far** | 14 | **$36.60** | 195 min |

The Step 4 and Step 6 rehearsals and the T5 fixture used no agent.

## Omni interventions (deus ex machina notes)

- Diagnosed T1's two problems: the embedded materials and their cause (the sidecars), and the
  environmental persistence timeout. The fix went into main. Handoff candidate: the agent should check
  `git diff --stat` and file sizes before finishing a node.
- Wrote and deployed the WFE guards (defects 2 and 3).
- Ran every cagent person step and gate check, and gave the failure mechanism and the missing-option
  diagnosis (`TimeoutIdleUSec=0`) to run-007. Handoff candidate: cagent reading its own host facts through
  a read-only playbook.

## Leftovers that need a decision (not removed by me)

- **Rejected T1 delivery on the share:** `shared://agshare/runs/playprey/onboard/run-001/o1/` (2.06 GiB).
  Not an accepted output, but deleting it is a retention decision.
- **Gitea branch `wfe/playprey/onboard/run-001`:** carries the four 510–523 MiB blobs. Deleting the branch
  frees Gitea storage. It is the record of a rejected run.

## When the trials resume

1. Settle defect 4 (commit before building a delivery), then resume run-003. It only needs `present`,
   `confirm` and `cleanup`.
2. Resume run-002 (snake) at `present`. Apply the same delivery rule, and verify the delivery myself before
   accepting.
3. T4: verify a published delivery from the Linux VM through `/mnt/agshare` with `delivery.py verify`.
   It needs only Python, and the VM's Python 3.12 is enough.
4. T3: a `compare` run, eagle_beakfit vs eagle_beak2, which also gives T6 a run that cites the earlier
   cases.
5. The lightweight review page and the Developer's experiential review (plan items 7.3 and 7.4), then
   report7.

## Deployed state at this pause

| Component | Revision |
| --- | --- |
| WFE (live release) | agdevworld `e19fa04` (includes `a1b217f`) |
| playprey main | `45b3ceec` (run branches of run-002/003 not integrated) |
| pj-clusterintent (live) | `fff9a4c` (run-007) |
| agautolab wfrun guide | `9104ca9` |
