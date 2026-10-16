# project_migration p6 — Step 1 report: inventory, requirements and seams

Status: complete (2026-10-16). Read-only investigation; no repository,
service or cluster state was changed. Raw evidence (host facts, sizes, paths,
command output) is in the ignored phase evidence directory
`pj-agdev/.local/project-migration-p6/step1/`.

## 1. Repository state and applicable instructions

| Component | Revision at inventory |
| --- | --- |
| devdocs | `e290f60` (this plan) |
| pj-agdev | `2d74765` |
| agdevworld (WFE source) | `e5488f6`; installed WFE release `38ebdff`, no `workflow_editor` diff between them |
| agautolab (wfrun guide) | `3fbafad` |
| pj-clusterintent | `bb64e6a`; nctl `8bb4d3b`, ansible_agdev `e74195d`, nintent `2ff73f8`, nodeutils `d68b963` |
| managed cagent (`pj-cagent`) | `fe86534` |
| playprey source | `trustedagent/pj-playprey` `c5d7c836` (539 commits); the original working tree is at the same commit |
| protoprey (consumer) | `3bf3e90` |

Instructions applied: `devpolicy/styles.md`, `devpolicy/terms.md`,
`devdocs/README_DEV.md`, the ignored host notes and `localrule.md`.

Cluster facts came from nctl: the status check is healthy, and drift is
**45 converged / 1 drifting**, which is the pre-existing Observer
`service_missing` condition. The latest automatic observation was `partial`
because one laptop node was asleep. Every other host had been observed about
4.4 h before this read.

## 2. playprey asset classification

The project is a macOS Godot + Blender + browser-editor lab. It has model
profiles (`data/model_profiles.json`), and each model has an authored
`config.json` and `presentation.json`. GLB and Blender files are ignored. The
launcher checks each required file against the profile's `asset_id` hash and
refuses to run a different model silently.

### Git tree and history (measured separately)

| Set | Size | Content |
| --- | --- | --- |
| HEAD tree contents | 2.93 GB (≈2.79 GiB), 23,001 files | `godot_lab/mouth_start/evidence` alone is 2.49 GiB: images ≈1.06 GB, JSON/gz/bin ≈1.0 GB (compresses to ~0.13 GB), mp4 ≈0.43 GB. Blender shots ≈0.13 GB; runtime textures ≈0.13 GB; code 5.5 MB; docs 5.5 MB; authored model configuration 3.8 MB |
| Full history pack | ≈1.5 GiB | Almost entirely current HEAD evidence media; history-only blobs are only ≈15 MB packed |
| Model binaries in history | none | No `.glb`/`.blend` was ever committed; only Godot `.import` sidecars |

Two surprises: ≈128 MB of byte-identical copies of ignored textures are
committed under `assets/reduction/`, which the ignore rule misses. HEAD holds
≈298 MB of duplicate blobs in total. **Removing evidence from HEAD frees almost
nothing.** A smaller repository needs either a fresh managed import or a
history rewrite (decided in Step 5).

### Ignored files in the original working tree (≈56 GB)

| Class | Size | Examples / disposition |
| --- | --- | --- |
| Original inputs (Developer-supplied) | 0.45 GB current + 0.38 GB retired | `source/<model>/*.glb`. These are Meshy outputs; their prompts and settings are recorded nowhere |
| Necessary intermediate checkpoints | 1.82 GB | Horse Blender checkpoints that the default model's rebuild needs and that `source/` alone cannot reproduce |
| Runtime assets required by profiles | 0.49 GB | Profile GLBs, masks, tissue GLBs |
| Analysis environment | 0.49 GB | A Python virtual environment inside an ignored `work/` folder. It has no requirements file, and 39 tracked files point at it |
| Regenerable | ≈17 GB | Case-chain blends 1.82, case `work/` 0.67, exported GLBs 0.71, extracted textures 0.24, generator-input probe dumps 0.14, Godot import cache 1.94, exported `.app` bundles 11.6 |
| Large evidence / leftovers | ≈20 GB | Raw AVI captures 12.6, other dumps 0.69, `.blend1` backups 6.6 |
| Obsolete cases | ≈15 GB | Old-episode labs (e.g. `assess_separated_model` 8.6) |
| Stale runtime assets | 0.32 GB | No longer referenced |

What must actually leave Git: **≈2.8 GB** (original inputs, horse checkpoints,
runtime assets) **plus a declared analysis environment**. Nothing a
registered profile needs is missing from the original tree, and every
present file matches its `asset_id`. In a clean clone,
`tools/model_profiles.py check` fails for every profile with a "required file
… is missing" error.

**Portability defects to fix in Step 5:** the original working-tree path is
hard-coded in 25 tracked text files, including runnable chain scripts
(eagle ex2 `chain.sh`/`env.sh`/`godot_sync.sh`, combined_jaw p2 scripts),
and in 1,332 evidence JSON files.

### Per-profile inputs

| Profile | Required inputs | Production route |
| --- | --- | --- |
| `horse_combined` (default) + r1 variants | profile GLB, tooth mask, tissue GLB | Rebuild depends on four checkpoint blends, back to the retired separated model; source GLB is recorded only in README/export notes, not `inputs.assembly` |
| `wolf`, `cheetah` | two source GLBs each | Case chain → texture classes → VFX materials → asset sync; tissue generation also needs probe dumps checked by fingerprint |
| `eagle`, `eagle_beakfit` | two source GLBs (shared) | `eagle_beakfit` has the only fully scripted chain: `chain.sh src` → `chain.sh build` → `godot_sync.sh` |

All routes need Blender 5.2.2, Godot 4.7.2, Node ≥ 23.6 (editor) and the
numpy/scipy/scikit-image environment.

### Onboarding knowledge and project distinctions

`devdocs/episodes/casestudy_and_toolify/guide.md` describes onboarding in five
stages: inspection, construction, registration, fitting and handover. It
draws on the wolf, cheetah and eagle cases and explicitly permits judgement
and iteration. The project keeps three phases of work distinct:

- **Mechanism correctness:** the mechanism works and can be evaluated, and
  that is what completes a phase.
- **Human tuning:** people tune the experience, and once a person tunes,
  generators stop being rerun.
- **Optimization:** done only on request and logged under `optimize/`.

The migration keeps these distinctions and is not a profiling exercise.

### Proposed trial cases

1. **Existing case — `eagle_beakfit`** (fallback `eagle`).
   - Inputs: the two eagle source GLBs and the tracked config.
   - Route: scripted chain → `model_profiles.py check`.
   - Evaluable means `tools/casestudy/eagle_p2/probe_check.gd` passes
     (eval/identity/carry/root/tongue/idle), plus the persistence/state
     scripts, closure evaluation and same-camera stills for the human.
   - A byte-identical rebuild is not established. Expect a new `asset_id`
     and the documented restamp step; that is a recorded new revision, not
     a failure.
2. **Structurally different case.** Preferred: a new anatomy supplied by the
   Developer as a shared folder, which exercises the real intake. **No such
   model exists yet** (missing; owner: Developer). Available now:
   - `eagle_beak2` (eagle P1 "Alt1": its source is present, the plan is
     written, it was never run). It is the natural candidate for the
     coexisting-approaches trial against `eagle_beakfit`.
   - `cheetah`, the mammal path with crowns, tongue straightening, generated
     tissue and interactions. It is the different-anatomy fallback.

   Not recommended: `horse_combined`, which depends on the checkpoint chain,
   and the retired separated horse.

### Runtime platform

The pipeline is macOS-only. The launcher uses osascript/plutil, the Godot
wrapper uses caffeinate, Blender is called by its application path, the only
export preset is macOS, and 42 scripts call macOS-only binaries. Only Macs
with these tools can run the pipeline. Other cluster hosts can store and
verify shared data, and a Linux host cannot reproduce a build without new
work, which is not in scope.

## 3. Cluster capability for the shared area (current behavior)

| Node role | OS | Capacity (root fs total/free) | Availability |
| --- | --- | --- | --- |
| Execution host (WFE executor, Nautobot, MinIO) | macOS arm64 | ≈1.9 TB / ≈1.0 TB | Always on |
| GPU workstation | Ubuntu 24.04 x86_64 | ≈3.7 TB / ≈1.9 TB (two disks) | Often powered off at night |
| Hypervisor | Proxmox (Debian 13) | 94 GB / 74 GB root; local-lvm free space is not collected | Always on |
| Autolab VM | Ubuntu 24.04 | 30 GB / 16 GB | Always on with the hypervisor |
| DNS guest | Ubuntu 22.04 unprivileged LXC | 7.8 GB / 6 GB | Always on |
| Laptop | macOS arm64 | ≈0.9 TB / ≈0.7 TB | Usually asleep |

**Existing storage**
- No NFS code exists in any backend repository, and Nautobot declares no
  share service. MinIO is the only storage service.
- The NFS server on the execution host is enabled but not running, and it
  has no exports.
- The execution host also runs SMB with three guest-writable shares that are
  recorded neither in Nautobot nor in cagent. This is an observation gap,
  noted here and not changed.

**Server candidates** (cagent decides in Step 3):
- **Execution host:** capacity and availability; macOS nfsd, privacy (TCC)
  and uid-mapping risks; large blast radius; macOS observation needs new
  code.
- **GPU workstation:** the conventional Linux server with the most capacity,
  but often off.
- **Hypervisor or a guest on it:** always on, but capacity is small or
  unknown, and the kernel NFS server needs a VM or privileged container.

**Client candidates**
- The execution host is mandatory: it is where the executor identity runs,
  under launchd, using the macOS automounter.
- The always-on autolab VM is the easiest second client to verify; the GPU
  workstation is the likely processing host but power-cycles.

**Facts still unknown before Step 3**
- Linux uids/gids.
- Capacity of non-root volumes.
- Whether the vault become password works on every target.
- Whether macOS restricts launchd processes from network volumes.

## 4. Capability / gap matrix

### Storage installation (cagent and backend)

| Capability | Current behavior | Gap → owner |
| --- | --- | --- |
| Agent-run playbook | `cagentctl playbook` exists. Its allow-list is `playbooks = []` in the workspace's ignored `.local/cagent.toml` (also in all 11 run copies); the plan block lists `{name, limit}`; it dispatches `ansible-playbook <name> --limit` against live `ansible_agdev` | No entries configured → pj-cagent workspace config |
| Content binding | The agreement names the playbook file only. No revision, digest or inventory generation is checked | Bind and revalidate source revisions in the plan block → pj-cagent `cagentctl` |
| Empty target | Exit 0 with zero matched hosts counts as success; `limit: []` dispatches `--limit ""` | Parse the recap and refuse empty selection → `cagentctl` |
| Preview | No `--check`/`--diff` subcommand | Add a preview → `cagentctl` / ansible_agdev usage |
| Recovery | The playbook ledger has no backend operation id, so an interrupted entry stays unknown | Prefer the reconcile path, which has an operation id |
| Desired-state route | Services and placements in Nautobot, deployment profiles (`ansible_agdev/vars/deployment_profiles.yml`), `nctl reconcile` with hash-bound settings, SSH preflight, regenerated inventory and operation ids | Add `nfs_server`/`nfs_client` deployment profiles, roles and checks → ansible_agdev; no nintent schema change needed (server service + mount service, one placement per client) |
| Observation | nodeutils sees the root filesystem and running Linux systemd units only: no mounts, no exports, no macOS system daemons | Export/mount observation kinds and drift gap codes → nodeutils collector + nctl checks (superproject gitlink bump) |
| Usability evidence | None | Read/write/rename, large-file hash, cross-host visibility, remount, service-context automount — cagent's change-run verification, not a reconciler |

### Large-file exchange (WFE)

| Capability | Current behavior | Gap |
| --- | --- | --- |
| Run copy | Full `git clone --no-checkout`; `.local/` excluded; only top-level regular files of workspace `.local/` ≤ 1 MiB are carried | Inputs must arrive by reference + staging |
| Local areas | Git-tracked folders declared in `project.yaml` | Suit knowledge, not payloads |
| Request intake | Every entrance (`wfe request`, editor, HTTP, Zulip intake, delegation) → `createRunInCopy` → `inputOf` (verbatim text) | `inputOf` is the single place to detect `shared://` references |
| Artifacts / file API | Project-relative only; text reads ≤ 1 MiB; project file API refuses ignored files, caps at 4 MiB | Keep unchanged; add a distinct external-artifact record |
| Presentations | Copied into committed `devdocs/runs/.../presentations/`; 200 MiB / 2000 files limit; whole files held in memory by the present server | Unsuitable for multi-hundred-MB payloads; keep for small previews |
| Confirmation | Subject = `{path, sha256, bytes}` list (≤ 200 MiB, whole-file read); question carries a presentation *or* loose files; acceptance pins the accepted tree; integration stops on unexpected changes | Put the small output manifest into the subject so existing `subject-changed`/integration checks bind it; allow preview + manifest together; add `output-superseded` |
| LAN review | Presentation ports + editor answer URL from ignored `present.json` | Static byte-range download of the share, URL from ignored host configuration |

## 5. Concrete seams (proposed behavior, to be implemented in Steps 2–4)

**Storage (Step 2).** Install through the desired-state route rather than the
raw playbook path:

1. Add NFS server/client deployment profiles and roles for the actual OSes
   chosen (Linux server and/or macOS server, Linux and macOS clients),
   preserving existing exports and mounts.
2. Add observation for exports and mounts that distinguishes configured,
   running and mounted. "Usable" remains cagent verification.
3. Bind the deployment source revisions in `cagentctl` plan agreements and
   refuse empty target selection.

The raw playbook path stays available for one-off operations once
`cagentctl` refuses empty selection and binds revisions.

**Reference contract (Step 4).**

| Seam | Proposal |
| --- | --- |
| Reference syntax | `shared://<share>/<relative path>`, parsed by a new pure `shared/shares.ts` |
| Request entry | `inputOf` extracts references from the request text into `RunInput.references`, covering every entrance |
| Explicit forms | `--ref` and an HTTP field are optional |
| Editor | Only shows detected references and their availability (bounded stat) |
| Resolution | Ignored `<area>/shares.json` beside `present.json`: per-share root, mount marker (to detect "unmounted"), download base. Own containment check, so the project file guard stays workspace-only. Added to the protected-path and backup lists |
| CLI | `wfe share list\|resolve\|ls\|verify` |
| Input set | Run-recorded manifest (relative path, size, sha256), streamed hashing, staged under the copy's `.local/inputs/`; on resume a changed or missing input is recorded, not silently replaced |
| External output | Written on the share at a run/candidate destination, finalized only after verification; a small manifest copy is committed in the run record and is a first-class external artifact |
| Confirmation | Binds the manifest (in the subject and accepted tree) and verifies payload integrity when asked |
| Readers to update | Run view, `wfe run` CLI, the agent brief, the wfrun guide, notifications and project access |

**Tests and deploy:**
- Gates: `npm run check` in `workflow_editor` (+ new shares tests), and the
  role, nctl and nodeutils gates from the pj-clusterintent test matrix.
- WFE deploys via `scripts/wfe-release.sh deploy` (refused while an attempt
  is open). Guide edits wait for the release.

**Open choices carried to Step 4:**
- Frozen published input set versus verified local staging. The default
  proposal is verified staging, with the manifest as identity.
- How much re-hashing to do at ask and at decide time for multi-GB sets.
- Who serves the share over HTTP. A share-side static server is likely.

## 6. Ownership

| Repository | Owns in p6 |
| --- | --- |
| pj-clusterintent / ansible_agdev, nodeutils, nctl | NFS profiles/roles, export/mount observation and drift classification |
| pj-cagent (managed) | `cagentctl` binding/refusal fixes; placement investigation, plan, installation, verification, records |
| agdevworld (WFE) + agautolab guide | Share references, input/output manifests, confirmation binding, LAN download, readers |
| pj-playprey (managed, Step 5) | Tools, authored configuration, workflows, knowledge, delivery manifest |
| Developer | Supplied source models and any new anatomy input; plan agreement for infrastructure effects |

## Acceptance check

- **Inputs located:** every required input is located in the original tree,
  with hashes matching. Missing: generation prompts for supplied models and
  a new anatomy input (both Developer-owned).
- **Current versus proposed:** current behavior (sections 2–4) is
  distinguished from proposed behavior (section 5).
- **Concrete seams:** the storage seams (deployment profiles, observation,
  `cagentctl` binding) and the review seams (`inputOf`, `shares.json`,
  manifests in the confirmation subject, share-side download) are concrete,
  with owners.
