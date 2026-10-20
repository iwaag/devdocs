# p9 Step 1 — capability boundary

Machine facts and raw observations are in pj-agdev's ignored
`.local/project-migration-p9/` (step1/). Observed 2026-10-20.

## Starting point

| Item | State |
| --- | --- |
| WFE | installed `0251983` (= agdevworld HEAD); clean working trees in agdevworld and pj-agdev |
| Workers | `studio` (coordinator host, default, `conversation` lane) and `gpupc`, both available |
| Nautobot/nctl | `nctl status` ok; fresh `nctl observe agstudio` (observed); drift 46 converged, 1 drifting = the pre-existing `agobserver service_missing` |
| Unrelated working changes | untracked braindumps in devdocs (p6, workflow_editor p11) — left alone |
| Registry | 11 projects; no `lowerpoly`, no registered repository `trustedagent/pj-lowerpoly` |

## The repository

- `trustedagent/pj-lowerpoly`, public, `main` @ `a31efb8`, 21 commits,
  152 tracked files, 1.8 MiB of history. Root: `devdocs/`, `lab/`, `study/`,
  `wedo/`, `.gitignore`; no `project.yaml`, no workflows, no submodules.
- The original working copy is on the coordinator host (outside the
  operating area), at the same commit, clean, `origin` = the Gitea
  repository. Its ignored `.local/` holds what the experiments used:
  - the toolchain (~0.4 GB): a Python 3.12 venv (trimesh, pymeshlab, xatlas,
    scikit-image …), gltfpack/meshoptimizer 1.3, own wrappers `msimplify` and
    `mdt_lod`, KTX-Software 4.4.2 (extracted, not installed), gltf-transform;
  - experiment outputs (33 GB in `out/`, ~600 evaluated LODs);
  - environment notes (`localenv.md`) and a local rule file.
  Nothing of `.local/` is tracked. Early commits contained a few `.pyc`
  files (tens of KB, removed from the tree by `7ba1254`); not rewritten.
- Inputs: the playprey Meshy GLBs on the shared area
  (`shared://agshare/playprey/inputs/`, 16 files, 9 active).
- Blender 5.2.2 is installed on the coordinator host. The experiment ran
  there (its LAN viewer address is the coordinator host's).
- Knowledge: `study/` (literature), `wedo/` (12 experiment lessons),
  `lab/first_experiment/` (scripts, viewer), `devdocs/episodes/first_study`,
  `first_experiment/p1`, `p2`. Pending human evaluation: the flip test was
  tried and stopped by the Developer ("neither clearly better"); download
  size vs GPU memory, the level set and the 1024 px target view remain open.

## Front's execution identity

| Fact | Value |
| --- | --- |
| Executor | agautolab `wfexec` on worker `studio`, the coordinator host's user |
| Harness | Claude Code with permission prompts skipped (`skip_permissions` for `claude_code`) |
| WFE tool | `wfe` of the installed release; registry from the area launcher / `WFE_REGISTRY` |
| Gitea for `wfe` operations | `gitea.json` beside the registry: principal `developer` (or `--gitea <file>` / `WFE_GITEA`) |
| Git | available; no Gitea credential in the user's Git configuration (p6 ex1) |
| `trustedagent` credential | an existing credential file in pj-agdev's ignored Gitea folder (and a copy in the original working copy). Omni's session classifier refused Omni's attempts to inspect it ("credential exploration"); its format was not read by Omni |

## Capability matrix

| Operation | Supported by | State |
| --- | --- | --- |
| Clone/read the repository | public repository; `wfe` access caches once registered | works (anonymous clone done) |
| Prepare `project.yaml` + guide | file editing; `wfe validate` in the checkout | supported |
| Commit/push the preparation checkout **as trustedagent** | — | **G1: missing.** `wfe` authenticates as one principal per setting file, and only inside its own operations; there is no operation that runs Git with a chosen Gitea principal. p6/p8 Omni used a scratch helper importing `gitea.ts`. Front would have to rebuild that, or paste a token |
| Register | `wfe project register trustedagent/pj-lowerpoly` (needs a published `ag.project.v2`; the root may be owned by another user) | supported |
| Obtain a workspace | `wfe project obtain lowerpoly` (clone as the service principal) | supported |
| Place freework | `wfe workflow basics --commit --push` (new project: root and every local area editable) | supported |
| Run, delegate, decide, integrate | WFE runs; integration pushes as the service principal | supported if `developer` can push to `trustedagent/pj-lowerpoly` — **to verify** (p1: `developer` had push rights on other trustedagent repositories) |
| Local areas `study`/`wedo`/`lab` | `ag.project.v2` `areas` (directory areas) | supported |
| Large inputs/outputs | `wfe run input`, `wfe run publish` on `agshare` | supported |
| Discovery by Front | guide lists `wfe create` but not `wfe project register|obtain` | **G2: documentation.** Help covers it; the guide's tool list omits it |
| Toolchain on the execution node | the original working copy's ignored `.local/` | **not a WFE gap.** A run copy has an empty `.local/`. lowerpoly has to say where its toolchain lives on a node (ignored env notes + a portable reference), or rebuild it. Front must learn this from the request or the Developer, since nothing tracked records it |

Not gaps: run adjustment cannot add bindings later (the basic workflow
binds every area at placement); no branch protection on the target.

## Identity decisions

| Write path | Principal | Commit author |
| --- | --- | --- |
| Front's preparation push (project.yaml, guide) | `trustedagent` via the new operation (G1) | Front's run sets `user.name`/`user.email` for its commit; recorded in report3 |
| `wfe project register` / `obtain` | `developer` (service principal; reads only, registry-local) | — |
| `wfe workflow basics --commit --push` | `developer` | the coordinator host's Git identity |
| Run checkpoints, integration, published run records | `developer` (the WFE service; unchanged) | the executor's identity |

The service keeps `developer` globally. Switching integration to a
target-scoped `trustedagent` is not needed: the repository grants
`developer` access (to confirm in Step 2), and a second identity for one
project would split the run record's provenance. Reports state the
principal per write instead of claiming trustedagent for everything.

## Selected trial

Explain the existing LOD pipeline and reproduce **one** representative case
with the Phase 2 defaults: `lab/first_experiment/step7/lod_pipeline.py` on
`eagle_beak` (the smallest dense-matrix input, 0.58 M triangles; 69–313 s per
input in Phase 2), on the coordinator host (Blender, Metal baking, the
toolchain). Deliver the LOD GLBs, `lods.json`, evaluation sheet and a
comparison with the Phase 2 recorded numbers, on the share, plus the
execution recipe and limitations. Human visual judgment stays pending.

## Step 2 scope

- **G1**: a small common operation: `wfe gitea` with named principals in the
  existing Gitea setting (`whoami`, `repo` access, `git` with the principal's
  authenticated environment, refusing force pushes), token never printed.
- **G2**: name `wfe project register|obtain` and `wfe gitea` in Front's guide
  tool list.
- Verify `developer`'s push permission on the target with the same operation.
