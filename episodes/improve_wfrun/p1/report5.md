# improve_wfrun p1 step 5: verify, release and document

Date: 2026-11-02. Released WFE agdevworld `145c852` (went out as `db4e7ae` → `011006b` → `145c852`;
previous `47a83cc`). Machine-specific logs, addresses and recovery details are in the coordinator's
ignored `pj-agdev/.local/improve-wfrun-p1/` (notes.md, release/, check/, shots*/, trial/) and
`pj-clusterintent/nctl/.local/improve-wfrun-p1/`.

## Documentation

agdevworld `db4e7ae`: `docs/runs.md` (*Definition snapshot*: which `devdocs/workflows/`,
`definition.source` table), `docs/storage.md` (published copies, integration targets, *The workspace
and the published source*, fixtures), `docs/access.md` (*Published workflows*: readers and
freshness, invalidation), `docs/delegation.md` (`target.source`, dispatch resolution on the published
branch, the child at the dispatched commit), `docs/intake.md` (routes from published workflows),
`README.md` (entry and commands). CLI help: `wfe request` (`--editing`), `wfe workspace`, `wfe
delegate request` and `targets`. devdocs `README_DEV.md` has a *Published workflows* section.
Executor guidance (agautolab) is unchanged: the facts live in tool help.

## Records before the release

- Inventory and replay: `wfe run check --workspace <each>` for all 20 workspaces with the new code
  and with `47a83cc`: 40/40 usable.
- Reducer change: `delegation.dispatch` keeps `target.source`. Records without it replay as before.
  `definition.source` and `copy.json` `source`/`baseFrom: published` need no reducer change.
- Backup of every area store and the area JSONs before activation (path in the local notes).
  No store schema change.

## Release

| Step | Result |
| --- | --- |
| `wfe-release.sh build` `db4e7ae` | `npm run check` 673/673 |
| activate `db4e7ae` (03:50 JST) | service and `bin/wfe` healthy |
| agpc worker pin `db4e7ae` | first playbook failed: the commit was not pushed to the agdevworld remote; pushed, rerun ok=22 changed=3 failed=0 |
| **regression** | the executor's `wfe intake process` and `wfe notify due` exceeded their 300 s limit from about 04:14 to 04:33 JST: every validation re-read every project's peers with Gitea API calls per read |
| hotfix `011006b` (04:33) | cache window without Gitea calls, one peers reading per pass, per-view memo; 673/673; `wfe intake status` 3.2 s (`47a83cc`: 17.6 s); the next intake pass caught up by cursor (post 28691 → the trial's child) |
| `145c852` (04:48) | "not in this workspace" instead of "deleted" for a run's workflow the workspace lacks; 673/673 |
| agpc worker pin `145c852` | ok=22 failed=0; read back `145c852`, executor `a9b82f8`, service active |
| pj-agdev pin | `e3c2052` |

The regression was mine and is the main lesson of the release: the full suite does not exercise
discovery over the live registry's 21 projects. After the hotfix, `wfe notify status` takes 32 s on
both `47a83cc` and `145c852`.

## Deployed checks

- **Live discovery without restart** (production, gimmickimg): two minutes after activation the
  executor created `#846 wf-gimmickimg.partial-animation` (the workflow integrated by freework/run-018
  that the old release never saw). `wfe delegate targets gimmickimg` reports it `NOT READY` because
  of its own validation error (`confirms-type`); `freework` ready. The gimmickimg workspace is still
  6 commits behind and was not pulled.
- **End-to-end trial** in the dedicated project `wfrun-trial` (created by `wfe create` as the check
  actor `improve-wfrun-p1-check`; its workspace kept at its first commit `9e6f4a8` throughout):
  1. freework/run-001 (requested by the check actor) created `stamp` and did not run it; its
     definition source is the published `9e6f4a8`. I read `stamp.yaml` in the run copy and accepted;
     it was integrated to main (`66b4262`). Integration happened before my acceptance, as the
     integration contract has it (acceptance is a record, not a gate).
  2. `wfe delegate targets wfrun-trial/stamp`: ready (valid, requests, channel `#849` provisioned)
     within three minutes of the integration (checked 19:03 UTC; integrated 19:00 UTC).
  3. A `check` conversation (`improve-wfrun-p1-trial`) led by ponderer: the brief asked for the new
     workflow (not freework) to stamp "heron" and for which run, workflow and revision did it.
     conduct/run-069 dispatched d1 to `wfrun-trial/stamp` at `developer/pj-wfrun-trial@66b4262`.
  4. Checked in the records, not from the conductor's summary: child `stamp/run-001`, `entrustedBy`
     d1, `definition.source` published, selector `commit`, commit `66b4262`; copy base `66b4262`
     targeting main; `stamps/heron.txt` = `heron\n` (6 bytes, sha256 1981f5df…), integrated to main
     `61517e4` and readable there with `wfe projects read`; the assessment's line `ran —
     wfrun-trial/stamp (run-001) with the published definition and code of
     developer/pj-wfrun-trial@66b4262272, the revision dispatched`; d1 judged satisfied by the
     conductor. Its reply (#565) states the same run, workflow, revision and file.
  5. Remote worker: `stamp/run-002` (`--node gpupc`) from the published `61517e4`; the attempt ran on
     gpupc and its result reached main (`bff90ae`, `stamps/egret.txt`).
- **Deployed UI** (`http://<coordinator>:8093/wfe/`, headless Chrome): wfrun-trial shows Published
  `@61517e4` with `stamp` "Not in this workspace", "7 commits behind" with Update workspace, Detail's
  table, the trial wording; no page errors. One 390 px run reported horizontal overflow; two repeats
  and an element scan did not reproduce it (recorded as not reproduced). The run view of stamp/run-001
  shows "Source: published — definition and code from developer/pj-wfrun-trial@66b4262272 (main), the
  revision named at creation".

## Recovery

Code rollback alone is not enough. `47a83cc` replays `delegation.dispatch` without `target.source`,
so a parent that dispatched after the release (ponderer conduct/run-069) would read as inconsistent;
a copy reserved from a published source and not finished cannot be finished by it. The local notes
list the procedure (stop the executor, account for those runs, activate, re-pin agpc).

## Not verified

- Update workspace / `wfe workspace sync` on a live workspace (fixtures only; no live workspace was moved).
- A trial of editing content on the deployed service (fixtures and the request form only).
- Revision race and delayed delivery on the live service (fixtures only; the live child was created
  at the dispatched commit, but the default branch did not move in between).
