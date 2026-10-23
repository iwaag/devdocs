# Challenge p2 — Step 7: deployment and report

The cutover took place on 2026-10-23 between 17:20 and 17:22 JST. It was
verified between 17:36 and 17:42 JST (report6). Local evidence, rollback
commands and the backup location are in ignored files
(`pj-agdev/.local/challenge-p2/notes.md`, `.local/devenv.md`).

## Before the change

| What | State |
| --- | --- |
| WFE | `aadee43`, store v11 |
| aghangout | `6e311e4` |
| Slack | `a988fe9` |
| wfrun guide | agautolab `227ef1c` |
| nctl drift | converged=46, drifting=2 (agobserver and cagent-snake-e2e, both pre-existing) |
| Backup | Store, configuration and release pointers (conversations, exec, challenges, dialogue, slack, registry, workers): store v11, max message 315, max request 93 |

The migration was first rehearsed on a backup of this store (report2,
report3).

## Cutover

1. New intake and claims were paused: the executor
   (`com.agdev.agautolab-wfexec`) and Slack (`com.agdev.slack-front`) were
   stopped. WFE kept accepting messages, so pending input stayed in the
   store with its tokens.
2. WFE `21a094e` was activated, and the store migrated from v11 to v12 on
   first open. The result matched the rehearsal: no request was left
   pending and nothing was replayed.
3. `conversations.json` was replaced with the new shape.
4. The guides were committed and pushed: agautolab `ba9e2c1`, pj-front
   `66492de`, pj-forge `415b1a8`, pj-archsage `31c8908`, pj-aghangout
   `9604695`, pj-ponderer `ef7ac65`.
5. aghangout `9604695` was deployed.
6. Slack release `21a094e` was activated; it posts `with: front`.
7. The executor and Slack were started again. The 10 direct trial rooms were
   classified `check` by "Omni Agent (challenge p2)", with the reason
   recorded.

No message arrived between the backup and the cutover. The only messages
since are #316–#321, all in the check room.

## After the change (this step)

- **Repositories.** agdevworld `21a094e` was pushed. pj-agdev `53af114`
  records agdevworld `21a094e` and agautolab `ba9e2c1`.
- **nctl.** Right after the cutover, drift showed agautolab and
  slack-front as `service_missing`. The observation dated from the pause;
  both processes match their patterns. `nctl observe agstudio` refreshed
  it, and drift returned to converged=46, drifting=2 (the pre-existing
  pair).
- **Archive publication.** 63 rooms are `current`: 43 under
  `conversations/` and 20 under `discussions/`.
- **Pending deliveries.** No request is pending or started. The 2 failed
  requests and 1 withdrawn request all predate p2 and involve the retired
  lowerpoly.
- **Services.** WFE, the executor (studio and gpupc both `available`),
  Slack (connected) and aghangout (HTTP 200) are running.
- **Obsolete code.** WFE's direct engine (`post`/`dispatch`/`dispatchDue`/
  `reply`/`view`) and `api/discussions` are gone. aghangout's `meeting.ts`
  is gone. No second conversation engine remains; the only traces are tests
  asserting that `api/discussions` returns 404.

## agpc worker pins (applied after the Developer's permission)

The session's auto-mode classifier first refused `nctl desired apply`. The
Developer then gave permission, and the change was applied with gpupc idle:

- `nctl desired apply` with `pj-clusterintent/nctl/.local/challenge-p2/agpc-pin-batch.yaml`
  updated one placement (`agautolab-agpc`): `wfe_version` `aadee436` →
  `21a094ee`, `executor_version` `227ef1c6` → `ba9e2c1a`. The export before
  the change is `desired-before.yaml`.
- `nctl render production` applied 23 placements (0 not applied).
- `setup_wfe_worker.yml -l agpc`: ok=22, changed=4, failed=0 (log
  `agpc-playbook.log`).
- On agpc, `~/wfe-worker/src` has agdevworld `21a094e` and agautolab
  `ba9e2c1`. Its `wfe conversation` is the new command, and gpupc is
  `available`.
- nctl drift: converged=46, drifting=2 (the pre-existing pair).
- **Rollback:** set the two pins back in the batch, then repeat the apply,
  the render and the playbook.

## Recovery

- **Prefer forward repair.** A code-only rollback is impossible because
  `aadee43` refuses store v12.
- **Full rollback.** First dump the live store, which keeps the
  post-cutover records. Then pause the executor and Slack, activate WFE
  `aadee43`, and restore the backup's `conversations.sqlite` and
  `conversations.json`. Activate aghangout `6e311e4` and Slack `a988fe9`,
  revert the six guide commits, and restart.
  - Data lost today: only the check room's records (#316–#321).
  - Cost: external effects, such as the check runs and the archive commits
    in pj-ponderer, cannot be undone. Archives stay as records and are
    never replayed.
- **Archive restore.** A fresh clone of pj-ponderer restores transcripts.
  This was verified for the check room in report6, and for all 62 rooms
  in the step 3 rehearsal. Pinned challenges need `pj-challenges` (or its
  agpc mirror) beside it.

## Documentation

- `devdocs/README_DEV.md`: new section *One conversation (`challenge`
  p2)*. *Conversations with project agents* and *Discussions* now point to
  it, and the Challenges limits that p2 resolved are updated.
- `pj-agdev/.local/devenv.md`: *challenge p2 on this host*.
