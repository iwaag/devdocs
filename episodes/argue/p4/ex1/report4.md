# argue p4 ex1 — Step 4: deployed, cut over, argue retired

## Starting state (Nautobot/nctl)

- `nctl status`: Nautobot healthy, worker idle. Host dumps were 9.5 h old,
  so I refreshed agstudio first: `nctl observe agstudio`, job
  `01M5ECSMBQGV92R2J191R6Z35G`, success.
- `nctl drift --host agstudio`: converged, before and after.
- No service placement belongs to argue or ponderer. Discussions run inside
  the WFE service (`agdevworld-web`) and aghangout (`aghangout_web`), and
  neither placement changed, so Nautobot needed no update.

## Backup and recovery

Backup: `.local/agdev/.local/backups/argue-p4-ex1-predeploy-20261021-135158/`.
It holds:

- the conversation store v7 (`.backup`) and `exec.sqlite`;
- `conversations.json` and `registry.json`;
- the archive writer's clone;
- mirrors of Gitea pj-argue and pj-ponderer, and of the argue workspace.

Rollback is written in `pj-agdev/.local/argue-p4-ex1/notes.md`. Code and
data must go back together: release `890b554` refuses store v8, so the v7
store copy has to be restored with it. Nothing reads the backup; it is not
a compatibility path.

## Deployed revisions

| Component | Revision | How |
| --- | --- | --- |
| WFE | agdevworld `5599b0c`, then `153bdad` (from `890b554`) | `scripts/wfe-release.sh deploy`. Each build runs the full check: 480, then 481 tests, `tsc`, vite build. Store migrated to **v8** on first open. |
| aghangout | `545d9f8`, then `6210d30` (from `a573f09`) | `pj-aghangout/scripts/deploy.sh` (13 unit tests on deploy) |
| pj-agdev pins | `9095165`, `17c858f` | pushed |
| pj-ponderer | `7137805` | Gitea main |
| wedo-argue / study-argue | `56cd52a` / `f11bc1d` | Gitea main |
| pj-archsage / pj-forge | `92ad65a` / `c4091ee` | Gitea main |

## Cutover sequence

1. **Final catch-up.** argue's `main` was still `4fd89fd`, which is already
   in pj-ponderer, and all 11 archives were `current`. No catch-up merge
   was needed.
2. **The release before the switch.** WFE `5599b0c` was deployed while the
   archive still pointed to argue. Every record became `unverified`
   because none had a destination yet. No reconcile ran between deploy and
   switch, so nothing was verified against argue.
3. **The switch.** In `conversations.json`, `archive` → `ponderer` and
   `argue/synthesize` was removed from participants. Then `wfe discussion
   archive`:
   - all 11 were `unchanged`, so nothing was appended;
   - all 11 were `current` in pj-ponderer under the same commit ids, which
     are argue's commits and now pj-ponderer's;
   - the writer clone was rebuilt with origin pj-ponderer.
4. **The executor kept running.** It calls `wfe exec reconcile` (the
   current release) every minute, and the writer's lock serialises
   publications. My attempt to stop it for the cutover was refused by the
   session's permission classifier. The deploy-then-switch order made the
   stop unnecessary: the old code never saw the new destination, and the
   new code never treated an old record as current.

## Reconciliation of migrated archives

- Every archive link (`…/developer/pj-ponderer/src/commit/<c>/discussions/<id>`)
  answers 200 and shows the manifest and transcript. A list is in
  `.local/argue-p4-ex1/step4/archive-urls.txt`.
- **Live continuation of an open discussion.** `20261020-233458-r93d5v` was
  opened under argue (facilitator `argue/facilitate`). I posted #241 as
  Omni Agent for the Developer, addressed to nobody.
  - WFE invited `ponderer` (invitation 67), which started
    `ponderer/conduct/run-007`.
  - In 25 s the run posted announcement #242 naming itself as conductor
    and pj-ponderer `discussions/<id>/` plus `HISTORY.md` as the record.
    The run completed.
  - The reconcile published #241–#242 to pj-ponderer (`a147a0f`, then
    `f86bccd`). The status is `current`.
  - Gitea pj-argue `main` stayed at `4fd89fd`: the old repository received
    nothing.
- **Pending results.** There were none (Step 1). The live check shows a new
  outcome reaching its owner, the conductor run, after cutover.

## Retired paths

- `wfe project unregister argue`: workspace, project and root repository
  entry removed from the registry. There is no workflow, participant,
  schedule, conversation target or project-discovery route left.
- The workspace was moved to `.local/agdev/.local/retired/pj-argue`. It is
  clean at `4fd89fd` and still has its ignored run store. A README entry
  says nothing builds, runs or registers from there. The disposable access
  cache `gitea-148.git` was removed.
- argue's Zulip channels (`wf-argue.*`) remain as history. Nothing is
  delivered to them; every delivery was `sent` before the cutover.
- **Gitea `developer/pj-argue` is unchanged** (not archived). Making it
  read-only through the Gitea API was refused by the session classifier as
  a shared-resource change. Its operational retirement does not depend on
  it, because no registry entry, workspace, binding or configuration refers
  to it, and `HISTORY.md` says so. Whether to archive it on Gitea is left to
  the Developer: Gitea → developer/pj-argue → Settings → *Archive this
  repository*.

### A history-resolution gap, fixed narrowly

After unregistering argue, the views of old discussions still linked
argue's runs to `#/ws/argue/run/…`, a run view that no longer opens. This
was concrete, so I extended retirement handling only for it. In WFE
`153bdad`, a run of an unregistered project:

- keeps its run name and participant;
- gets **no** run-view or file link;
- states the reason ("not registered here (a retired or unknown project):
  no run view; the run's record stays in that project's history").

There is no alias or redirect. The test is in `archiveDestination.test.ts`.

### A presentation defect found in the LAN check, fixed

aghangout's To: offered "argue (facilitator)" in old discussions, because it
listed every member. WFE refuses such an invitation. aghangout `6210d30`
offers only members that WFE lists as invitable; the membership history
still shows argue. A unit case was added, and the browser check passes.

## Documentation

- devdocs `README_DEV.md`:
  - *Discussions* (renamed from *Discussions and argue*) marks argue as
    retired;
  - the ponderer section became *ponderer conducts, keeps and learns from
    discussions*, with the new layout, workflows, knowledge repositories,
    WFE v8 and the retirement. The p4 details are kept below as history.
- `pj-agdev/.local/devenv.md` (ignored): *argue p4 ex1 on this host*.
- Historical episode reports are unchanged.

## LAN evidence (agpc)

`.local/argue-p4-ex1/step4/lan-check.mjs` was run on agpc through Ansible,
and all 28 checks passed (`agpc-lan-check-2.txt`, after the To: fix). It
checked:

- the facilitator is `ponderer/conduct`, and argue is not a participant;
- all 11 archives are `current` in pj-ponderer, and every link opens;
- the continued discussion (#242 by ponderer) is archived in pj-ponderer;
- argue's speech keeps its participant and run, with no dead link;
- the conductor run view opens;
- the summary, episodes, `HISTORY.md`, two `argue@` citations through
  pj-ponderer, wedo-argue, study-argue and forge's provenance knowledge all
  open;
- aghangout Records: "Archived through #242" in pj-ponderer;
- Home: archived, none unverified;
- no page errors.

## Review links (from any LAN machine)

- The continued discussion:
  `http://agstudio.local:8093/hangout/?discussion=20261020-233458-r93d5v`
  (Records tab).
- Home: `http://agstudio.local:8093/wfe/` (Discussions card).
- ponderer: `http://agstudio.local:3000/developer/pj-ponderer`
  (`HISTORY.md`, `discussions/`, `episodes/`).
- wedo-argue: `http://agstudio.local:3000/developer/wedo-argue`.
- study-argue: `http://agstudio.local:3000/developer/study-argue`.
