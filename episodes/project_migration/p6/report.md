# project_migration p6 — final report: shared data and evolving workflows through playprey

Status: **complete** (2026-10-17), with a reduced Step 7 scope set by the Developer. Two of the six trials
stopped by the Developer's decision: different anatomy, because the supplied model was too distorted; and
coexisting approaches, stopped after its plan was agreed. The Developer judged that the migration of
playprey itself was sufficiently shown. Step reports: [report1](report1.md) to [report7](report7.md), plus
[ex1](ex1/report.md) (cagent develops macOS NFS support and installs the share) and [ex2](ex2/report.md)
(cagent on Codex gpt-6-luna). Raw evidence is in the ignored `pj-agdev/.local/project-migration-p6/`.

## Delivered behavior

| Area | What now exists | Where |
| --- | --- | --- |
| Shared storage | One NFS-backed shared area, `agshare`, served by the always-on Mac and mounted by the Linux autolab VM. Installed, repaired twice and verified by managed cagent through its agreed `change` workflow; recorded in cagent and Nautobot | cagent `change/run-005` (install), `run-006`, `run-007` (repairs); [report3](report3.md), [ex1](ex1/report.md) |
| Deployment tools | Rerunnable NFS server (Debian Linux, macOS) and client (Linux, macOS) roles, nctl checks that tell configured, mounted and usable apart, `verify_nfs_mount.yml`; cagent can develop missing platform support (`develop` workflow) | pj-clusterintent `fff9a4c`; [report2](report2.md) |
| Portable references | `shared://<share>/<path>/` in a request; input sets with per-file hashes, staged or verified; outputs published as `o<n>` of a named candidate, never partially visible; confirmation bound to the published manifest; LAN download; `wfe share resolve/ls/verify` on any host | WFE `e19fa04` (`workflow_editor/docs/shares.md`); [report4](report4.md) |
| playprey | A managed WFE project with its history rewritten without media, inputs and checkpoints on the share, `assets.py` fetching what a fresh copy needs, a portable delivery (`delivery.py build/verify`) | `developer/pj-playprey` `0b1509f6`; [report5](report5.md) |
| Workflows | `onboard` (scope decides the work) and `compare` (separate candidates, choice, lesson fed back), version rules, adjustment and successor-run use; run summaries written before the confirmation | playprey `bdd3eb7a`; [report6](report6.md) |
| Run-size guards | Run branches never carry files over 50 MiB; unfinished copies are swept | WFE `a1b217f`, `e19fa04` |

## Asset and history disposition

- The original repository `trustedagent/pj-playprey` (539 commits) and the original working tree are
  **untouched**.
- The managed copy keeps 538 of the 539 commits:
  - 16,871 evidence files (2.45 GiB) were removed from every commit and archived on the share at their
    original paths. Each archived copy was checked against its blob id.
  - The tree went from 2.79 GB to 158 MB, and the pack from 1.5 GiB to 115 MiB.
- On the share: original inputs (463 MiB), retired inputs (391 MiB), horse checkpoints (1.71 GiB), the
  baseline runtime assets (647 MiB) and the archive.
- What to keep of the original working tree (about 56 GB of ignored files, mostly regenerable) is the
  Developer's decision.

## Trial results

| Trial | Result |
| --- | --- |
| Existing case | **Proven.** `onboard/run-003` rebuilt `eagle_beakfit` from a folder reference in a clean copy. The asset is byte-identical to the recorded case, all checks pass and the delivery verifies on the share. Accepted and integrated. The first attempt (run-001) was rejected, because Godot embedded 2 GB of textures that a tracked sidecar had blocked; fixed in the project |
| Different anatomy | **Stopped by the Developer** (snake model too distorted). Before that it exposed a real hidden assumption: every runtime check required a tongue. It added a declared tongue-absent capability with no regression in the 9 existing profiles |
| Coexisting approaches | **Plan only.** Two candidate profiles, with a visible go/no-go node added at the plan boundary. Nothing was built |
| Cross-host delivery | **Proven.** The Linux VM verified the published delivery through its own mount and ran the profile check from a copy, with no producer checkout. The LAN download matched byte for byte |
| Resume and publication | **Proven** with a fixture: a killed publish shows nothing, the retry publishes once, and a changed input is recorded on resume |
| Knowledge reuse | **Shown in plans and records**: earlier case lessons cited as applicable method rather than copied values. No finished later case |

Two-host **storage** access is proven. Distributed **execution** stays deferred. Only one WFE executor ran,
on the Mac, and the pipeline is macOS-only.

## Costs

- playprey runs: 21 attempts, **$40.76**, about 230 minutes of agent time ([report7](report7.md)).
- cagent ran on Codex gpt-6-luna after ex2, and that harness reports no dollar cost. Its attempt counts and
  times are in [report3](report3.md) and [report7](report7.md).
- Omni's own session time is not counted.

## Interventions (deus ex machina)

Every intervention is attributed in the step reports. The main ones:

- **Tool defects:** diagnosed the 2 GB material embedding and wrote the WFE size and partial-copy guards.
- **Infrastructure:** ran cagent's person steps through Ansible with privilege escalation, and gave the
  NFS idle-close mechanism to cagent run-007.
- **Agent guidance:** corrected the false "do not commit" rule in the shared agent guide.
- **Workflows:** fixed playprey's post-acceptance report placement.
- **Run history:** rewrote one run branch to drop a pre-guard 63 MB blob, with the Developer's approval.
- **Review:** verified deliveries independently and built the review launcher.

The Developer delegated all answers and acceptance in Step 7 to the Omni Agent, and made the Omni Agent a
permanent approver in every process. Every answer is recorded under the Omni Agent's name.

## What stalled the work

The Developer asked for this, to remove meaningless rules and guardrails. The table with costs is in
[report7](report7.md#what-stalled-the-work-asked-for-by-the-developer). In short:

- **Removed:** the shared guide's ban on committing.
- **Candidates to narrow or remove:**
  - the share's "no dot entries at all" rule;
  - WFE allowing a generated report after the result confirmation, which integration then refuses (it hit
    forge in p3 and playprey here);
  - cagent's re-agreement on every `plan.md` edit (22 questions in run-007).
- **Outside the project:** the Omni Agent harness's auto-mode classifier. It refused delegated and even
  read-only actions, and left the executor idle for about 100 minutes in total.

## Operations

Documented in `devdocs/README_DEV.md` "Shared areas" (person-facing use, unavailable share, retention),
WFE `docs/shares.md` (contract), playprey `knowledge/delivery.md` (using a delivery) and the ignored host
notes. The share's lifecycle and recovery are cagent's change records `run-005` to `run-007`:

- **Mechanism:** the client closes an idle connection, macOS nfsd leaves it half-closed, and the client
  hangs.
- **Fix:** a 300 s client automount idle timeout, so the client unmounts before the connection is left
  half-closed.
- **Repair path:** a cagent `change` request.

## Named follow-ups

1. The serving Mac keeps 11 CLOSE_WAIT NFS sockets that an nfsd restart does not clear, and their growth is
   untested. The idle-timeout fix also works only while the share is truly idle. This is a cagent `change`
   when it matters.
2. nctl emits no reconcile action when only a placement's config changes (seen in run-007).
3. Narrow the share's dot-entry rule, and make WFE refuse or reorder a generated report after
   `confirms: result`.
4. Relax cagent's re-agreement rule.
5. The different-anatomy case needs a usable model. `knowledge/cases/anaconda.md` and the tongue-absent
   capability exist only on the held run-002 branch, not on main.
6. protoprey integration of a delivery is a separate integration (renderer and scene ownership). No consumer
   integration is claimed.
7. Retention decisions, kept rather than deleted:
   - rejected or superseded outputs on the share (run-001 `o1` 2.06 GiB, run-003 `o1`, run-002 `o1`);
   - the Gitea branch of run-001 with its 2 GB blobs;
   - the two held runs (cancel them if they will not resume);
   - the original playprey working tree.

## Deployed revisions at close

| Component | Revision |
| --- | --- |
| WFE live release | agdevworld `e19fa04` |
| agautolab (wfrun guide, read live) | `500be25` (pj-agdev `fc70763`) |
| pj-clusterintent (live) | `fff9a4c` |
| pj-cagent main | `f7f49aa` |
| playprey main | `0b1509f6` |
