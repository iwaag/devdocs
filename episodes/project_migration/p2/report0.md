# project_migration p2 step 0 — preflight (2026-10-13)

Read-only. Nothing was changed in the operating area, in Gitea or in the project.

## System state

| Item | State |
| --- | --- |
| Installed editor release | agdevworld `92de1da` (service, `bin/wfe` and contract docs agree) |
| Registered projects | `rts-vs-bot`, `ppt`, `agsfx` (all `available`) |
| Zulip intake | on; since 2026-10-11; projects **all registered** (automatic), per-workspace overrides only for `ppt` and `agsfx` |
| Notifier | enabled; channel and folder per project provisioned on first use |
| Cluster services (`nctl`) | not consulted: this phase touches no cluster service, and `nctl` is not on this shell's PATH |

Two things differ from p1's picture and change the plan:

- **Every project has a `freework` workflow from the start** (`docs/contract.md`). jevlab's workflows therefore are the three of the plan plus `freework`, and step 2 must write a `freework.yaml` bound to the project's areas, as agsfx and ppt have.
- **Intake is automatic.** `zulip.json` needs no `intake.projects` entry for a new project; registration is enough. Step 2's "add to intake" is replaced by "check that `wfe intake status` lists the channels".

## What the project assumes of its host

- **It was developed on this host.** The original working copy sits in the home folder (outside the operating area), with an ignored `.local/localrule.md`, `localenv.md` and credentials, and ignored `.work/` folders (checkouts, venvs, weights) under `lab/`.
- The local rules are short: commit after a change; tools and weights may be installed freely, or asked for from the person if they cannot be installed automatically; record discovered tool information in `.local/localenv.md`; generate files in English.
- Models: the experiments use the local Ollama models and `qwen3.8:27b-mxfp8` is installed here (it appears about 7,000 times in the repository). Small checkpoint servers (the `start` and `game_test` episodes) need their `.work` venv and weights.
- Python 3.14 with numpy is present. `ffmpeg` and `say` are present; **`whisper` is not** (the video builders use it as a listening proxy).
- Large unrelated runtimes also run on this host (port conflicts were recorded once in `localenv.md`).

## What this means for execution

- A fresh workspace clone has **no `.work/`**. Anything needing weights or venvs either reuses the original copy's folders by a documented path (no symlinks into another project's ignored area without the person's say) or is out of the phase.
- Model-free experiments are reproducible from a clone: `set6-statistically_correct/p2` validation is "python3 tests/test_sim.py && python3 validate.py && python3 diagnose.py", deterministic and needs no model. **That is step 3b's cheap recomputation candidate.**

## Findings that need the person

1. **The original working copy holds unpushed work**: 15 untracked paths, none committed — a whole `set6-statistically_correct/p3` episode folder, two braindumps and a report of `start`/`game_test`, a `video_presentation/p1/ex1` folder, and ten server logs of `set3-weakstrong/p2`. Gitea's `main` is the one cloned for the migration. If the migration commit is made now, the workspace and the original copy diverge. Decision needed: commit and push that work first (the person's), or accept that p3 stays only in the original copy for now.
2. After the migration, **two checkouts of one project** exist on this host (the original and the workspace). Which one the person keeps working in is theirs to say; the plan assumes the workspace becomes the place for runs and the original copy is left untouched.
