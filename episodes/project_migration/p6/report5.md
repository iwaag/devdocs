# project_migration p6 — Step 5 report: playprey as a knowledge-centered managed project

Status: complete (2026-10-17). playprey is the managed WFE project `playprey` (root `developer/pj-playprey`).
Its large files are on the shared area with verified copies. Its history was copied and rewritten without
media, and the original repository is untouched. A fresh clone obtains the declared files and reproduces
the eagle_beakfit checks. Raw evidence is in the ignored `pj-agdev/.local/project-migration-p6/step5/`.

## Registration

| Item | State |
| --- | --- |
| Root repository | Gitea `developer/pj-playprey` (id 143), created through WFE's Gitea client and pushed with its credential |
| Project | `wfe project register developer/pj-playprey`, `wfe project obtain playprey` → workspace `playprey` in the operating area |
| Definition | `project.yaml` (ag.project.v2): intent, goals, areas `knowledge`, `godot_lab`, `blender_lab`, `mouth_editor`; `devdocs: directory` |
| Workflow | `devdocs/workflows/freework.yaml` (talk first; agent `playprey-agent`, guide `agent/guide.md`). Not approved: as in the other managed projects, approval is the Developer's |
| Zulip | `wf-playprey` folder and `wf-playprey.freework` provisioned automatically by the transport |
| Revision | playprey `b19f00c6` |

Step 6 adds the onboarding workflows. This step registers the project and makes it reproducible.

## History disposition (plan item 3)

Decided before any push and recorded in the repository (`devdocs/migration/README.md`):

- **Rewritten copy into a new repository; the original is untouched.** `trustedagent/pj-playprey` and the
  original working tree were neither changed nor force-pushed. They remain the archive.
- The copy keeps the history of code and documents (538 of 539 commits; one commit touched only removed
  paths and became empty). `devdocs/migration/commit-map.txt` maps every original commit to its new one
  (`c5d7c836` → `191f6d8a`).
- Removed from every commit: 16,871 evidence files at HEAD (2.5 GB of screenshots, videos and JSON over
  64 KiB that no document links to) and 18 textures committed by mistake (byte-identical to what Godot's
  import extracts).
- Kept: all code, scenes, data, authored model configuration and documents, the evidence that documents
  link to (≤ 2 MiB, no video), and small text and JSON evidence (3,180 files, 116 MB).
- Result: the tree went from 2.79 GB to 158 MB, and the pack from 1.5 GiB to 115 MiB.
- The original working tree's absolute path was replaced by `<repo>` in every commit. No absolute home
  path remains in any blob.

## Data on the shared area (plan item 2)

Each file was copied from the original tree, hashed as read and again as landed, and never overwrote a
different file:

| Reference | Content | Files | Size |
| --- | --- | ---: | ---: |
| `shared://agshare/playprey/inputs/` | Original Meshy models supplied by the Developer (eagle, eagle beak2, wolf, cheetah, horse jaw and throat) | 8 | 463 MiB |
| `…/inputs/retired/` | Inputs of the retired horse_separated profile and the first models | 6 | 391 MiB |
| `…/checkpoints/` | Horse lineage Blender checkpoints that a horse rebuild needs and `source/` cannot reproduce | 14 | 1.71 GiB |
| `…/baselines/c5d7c836/` | Runtime assets of all eight profiles, plus the probe dumps the tissue and pocket generators read | 207 | 647 MiB |
| `…/archive/c5d7c836/` | Every removed evidence file at its original path; each copy checked against its Git blob id | 16,871 | 2.45 GiB |

- **The manifest.** `godot_lab/mouth_start/data/shared_assets.json` declares the 235 non-archive files:
  local path, reference, size, sha256 and class. `devdocs/migration/archive-c5d7c836.tsv` maps the
  archive. A re-read of all 235 share copies matched their hashes.
- **Coverage.** Every profile's launch files and every original input named in a config are declared or
  tracked. None are undeclared.
- **Not copied.** These stay in the original tree only:
  - raw AVI captures (12.6 GB; their encoded videos are archived);
  - exported app bundles;
  - `.blend1` backups;
  - the Godot import cache;
  - regenerable case-chain checkpoints;
  - the labs of retired episodes.

  Their retention is the Developer's decision.

## Code changes (plan items 4–6)

| Change | Where |
| --- | --- |
| `tools/assets.py list\|check\|fetch\|add`: fetch a profile's launch files (`--profile`) or its original inputs (`--inputs`) from the share, verified by sha256. A local file with other content is never overwritten without `--force`. The share is located with `wfe share resolve`, so no host path is in the repository | `godot_lab/mouth_start/tools/` |
| The launch checks (Python and the Godot twin) name the fetch command when a declared file is missing | `model_profiles.py`, `scripts/moc/model_profiles.gd` |
| **Tuning guard.** `write_paths` records what each generator wrote (`generated/written.json`). A value changed since then (a person's tuning) is not overwritten unless `PLAYPREY_OVERWRITE_TUNED=1` makes the replacement explicit. Tested: write, rewrite, a simulated edit refused, the explicit overwrite allowed | `model_profiles.py`, README_DEV |
| Portability: chain scripts find the repository root themselves. The analysis environment is declared (`requirements-analysis.txt`, pinned from the environment the cases used; `tools/setup_env.sh` → `.local/venv`) and 17 files point at it. The import log is no longer written to `/tmp` | `blender_lab/…`, `tools/` |
| The onboarding guide moved to `knowledge/onboarding.md`, links rewritten, with a pointer left at the old place. The cases stay where its links point | `knowledge/` |
| Delivery contract and `tools/delivery.py build\|verify`: a Godot project of one profile (runtime at the commit, without evidence, only that profile registered) plus `delivery.json` (profile identity, producing commit, Godot version, every file by sha256, how to use it). A dirty tree is refused | `knowledge/delivery.md` |
| Agent guide head; README "A fresh checkout"; README_DEV "Files outside Git"; `evidence/ARCHIVED.md`; `devdocs/README.md` | — |

Retired and dormant features (`obsolete/`, the globs code) were imported as they were and are not
maintained, as the project's README_DEV already says.

## Acceptance evidence

All on this host, against the pushed repository:

| Check | Result |
| --- | --- |
| Fresh clone from Gitea, before fetching | `model_profiles.py check eagle_beakfit` fails and names `assets.py fetch --profile eagle_beakfit` |
| `assets.py fetch --profile eagle_beakfit` | 2 files fetched from the share and verified; `check` reports both `ok` |
| `godot --import` (twice: the first pass reports textures the GLB is still extracting) | 0 errors on the second pass |
| `model_profiles.py check eagle_beakfit` | `OK eagle_beakfit: config fa5431e37ede36a4 asset 632c3b971e13ef7d` |
| eagle P2 suite (`probe_check.gd`, headless) | All 13 checks PASS, `fails=[]`. Carry end drift 0.93 mm and the beak cycle to 56°, as in the original case's report |
| `delivery.py build --profile eagle_beakfit` | 1,606 files, 55.2 MiB, from commit `7d4acb88` |
| The delivery alone, in another folder, after deleting the producer clone | import with 0 errors; profile check OK; the probe suite all PASS; `delivery.py verify` passes |

This shows that a clean checkout reproduces the selected baseline's mechanism checks from declared inputs.
It does **not** show a rebuild of the GLB from the original models through the Blender chain; that is
Step 7's existing-case trial, and a byte-identical rebuild was never established (report1).

## Interventions and limits

- The whole migration was done by the Omni Agent, as this plan assigns. No in-system agent was involved.
- `git-filter-repo` was run through `uvx`; nothing was installed system-wide.
- In my interactive shell, the first `wfe` on PATH is an old build without `share`. Attempts get the
  operating area's launcher first, and the Step 4 trial agent used `wfe share`. `assets.py` names this case
  when it happens.
- Still open from Step 4: the Linux client's NFS mount hangs after the server's idle drop. Fetching works on
  the server host through its local backing path, which is where the macOS-only pipeline runs.
- Pending decisions for the Developer: whether to mark `trustedagent/pj-playprey` read-only or archived,
  and what to keep of the original working tree (≈ 56 GB of ignored files, most regenerable).

## Acceptance check

- **A clean run copy can obtain its declared inputs and reproduce the selected baseline:** yes for the
  mechanism checks of eagle_beakfit, from a fresh clone and from a standalone delivery.
- **Required data and knowledge survive the migration:** yes. Code, configuration, documents and linked
  evidence are in Git with their history. Inputs, checkpoints and runtime assets are on the share and
  declared. Every removed evidence file is archived and checked against its Git blob.
- **Explicit history disposition:** recorded in the repository and above. The original was not rewritten.
