# p11 Step 1 — restriction inventory

Work started on 2026-10-17 (JST). The implementation coverage is tracked in
[inventory.md](inventory.md); implementation and regression results will update
its open entries in subsequent steps.

## Baseline and ownership

| Repository | Starting revision |
| --- | --- |
| devdocs | `540501d0fa7f` |
| devpolicy | `0eb9fb8a012f` |
| pj-agdev | `fc70763cefc4` |
| agdevworld | `e19fa04d525b` |
| agautolab | `500be251aa3a` |
| pyagag | `6a492f6e4bf1` |
| pj-clusterintent | `fff9a4cb5a8a` |
| nctl | `c95310696cf1` |
| nintent | `2ff73f8e95cd` |
| ansible_agdev | `f68a68fd5a91` |
| settings | `47d5d713fab1` |
| managed rts-vs-bot / ppt / agsfx | `1062e00e4927` / `37831a63cfbe` / `e8d82d5835ca` |
| managed jevlab / agops / forge | `b3599aed0f7b` / `8c005cead6a3` / `e9ea3735ebe3` |
| managed archsage / cagent / playprey | `ddd8c1edbc5d` / `f7f49aaa72f6` / `0b1509f64f4b` |

Source repositories were clean. Existing untracked p6/p11 braindumps and Finder
metadata in two managed workspaces were recorded and left intact. The projects
folder itself is not a Git repository; reports belong to the independent devdocs
repository. A machine-local baseline, instruction-file index and raw restriction
matches are in ignored `pj-agdev/.local/workflow-editor-p11/`.

The audit includes the nine current registered projects, nested repositories,
workspace-local settings, installed releases, run copies, shared guides and their
generators. The initial instruction search indexed 108 files, including ignored
configuration; some are historical records rather than active guidance. Continued
caller tracing is recorded against inventory entries during implementation.

## Runtime evidence

`uv run nctl status --json` from the nctl package returned exit 0, `ok: true`,
Nautobot reachable/authenticated, intent REST/GraphQL present, one registered
worker and zero pending jobs. Observation ages vary; this proves connectivity,
not fresh convergence of every host. Running from the projects directory first
returned a concrete missing-config error; using the package directory resolved it.
Raw addresses and host observations are intentionally excluded from tracked reports.

## p6 evidence and next work

The p6 Step 7 record establishes S1–S6, including the earlier commit-guide fix,
dotfile omission, report-after-acceptance failure, external classifier refusals,
50 MiB checkpoint veto and repeated cagent plan agreement. p11 explicitly removes
those process prerequisites, including those p6 called meaningful. Dirty-tree
delivery is also in scope. Existing Developer-stopped playprey runs stay stopped.

The root local rules contain procedural mandates about commits, language, plan
numbering, compatibility and deployment preparation. Their dispositions are L01;
LAN exposure describes the requested environment behavior. Shared and project-local
rules are audited together so deleting one guide cannot leave its generator or a
run copy enforcing the same restriction.
