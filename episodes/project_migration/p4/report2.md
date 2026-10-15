# project_migration p4 — step 2: common project access

Written 2026-10-15. agdevworld `f94f07a`, not deployed yet; the installed release is still `a642e27`. Public contract: `workflow_editor/docs/access.md`.

## What was added

| Capability | Where | Surface |
| --- | --- | --- |
| **Project access** | `server/access.ts`, `cli/projects.ts` | `wfe projects list\|show\|tree\|read\|search\|runs\|run`; `GET api/projects[/<project>[/tree\|/file\|/search\|/runs[/<wf>/<run>[/file]]]]` |
| **Guides of another project** | `server/exec.ts` `execution()`, `shared/model.ts` `guideRef` | `execution.guides: ["<project>:<path>"]` |
| **Guarded bindings** | `server/integrate.ts` `guardsFor` / `integrateOne`, `shared/approval.ts` | binding field `integrate: if-unchanged`; step and integration state `target-changed` |

### Project access

- **Discovery** lists every registered project, with or without a workspace on this host. Each entry has:
  - name, intent and devdocs mode;
  - the root's current commit;
  - its submodules by Gitea name;
  - its workspaces and whether each is at the committed tip.

  Shared repositories are listed with their users.
- **Structure** covers:
  - `project.yaml`;
  - devdocs as declared and as observed;
  - declared local areas, and whether each is present at the revision;
  - submodules, with the **pinned** commit against the repository's **upstream** tip (`same`/`behind`/`ahead`/`diverged`, with counts);
  - workflows read wherever devdocs lives, with bindings, execution, intake and nodes.

  Knowledge areas are what the project declares. Folder names infer nothing.
- **Content** is tree, read and search at:
  - the root's default branch on Gitea (the default);
  - `--rev`;
  - the commit a submodule is pinned at;
  - `--upstream`;
  - `--require <commit>`.

  A workspace's working tree is read only with `--source worktree`, and is labelled. Every read is bounded, and a truncation is said. Binary content gives metadata only, or is saved with `--out`.
- **Execution** shows each run's **live** record (this host's resolver: the copy or the project tree) beside its **published** snapshot (the committed run folder, checked by the same reader). Run files can be read from either.
- **Provenance and states.** Every result names:
  - project, repository (Gitea name, registry key, mount path, category);
  - revision (selector, commit, and the commit that pins it);
  - observation time and fetch state (`fresh`/`recent`/`stale`).

  Failures are `absent`, `inaccessible`, `uninitialized` or `fetch-failed`, never an empty answer.
- **Isolation.** Committed content is fetched into `<area>/.local/access-cache/` (bare, one per Gitea repository, under a lock, with the existing `gitEnv()` credentials). No workspace is fetched into, checked out or written. `.local/` and ignored files are never content.

### The write-path prerequisites from step 1

1. **Guides of another project.** `<project>:<path>` is read at claim from that project's committed default branch, fetched then, and recorded as `<project>:<path>` with its sha256. A project's own run can therefore follow a guide archsage maintains, without a copy. An unreadable guide refuses the job.
2. **Integration guard.** An editable binding with `integrate: if-unchanged` makes its target a precondition.
   - If the target changed under the binding's path since the run's merge base with it, the integration stops as `target-changed` with the files, before anything is combined or pushed.
   - A change elsewhere in the repository still merges.
   - A run that brought the current target in passes.
   - The run view says "Not integrated: the target changed since this run prepared its result — reassess it in a successor run".
   - The field is in the definition digest only when declared.

Nothing else in ownership, acceptance, non-forced push, the journal or recovery changed.

## Checks

| Check | Result |
| --- | --- |
| `npm run check` (type check, all tests, Vite build) | **357/357** (13 new), build ok |
| `test/access.test.ts` (9) | Gitea stand-in. Covers: discovery with and without a workspace; devdocs as a directory and as a **submodule** (project `b` has no workspace); pinned against upstream reads, `behind 1`, `--require`; an earlier root revision; `repo:` addressing; `absent`, `inaccessible` (outside Gitea, `.local/`, no Gitea), `uninitialized`, `fetch-failed`; a **stale cache** labelled stale; explicit working tree (labelled, dirty, ignored refused); live run beside published (the snapshot lacks `run.integrate`), a run only on this host, published runs without a workspace; CLI `--json` failure state; cross-project guide at claim (read, missing, unknown project). The workspace's refs, status and submodule refs are unchanged after the reads. |
| `test/guard.test.ts` (4) | Validation, digest and resolution of `integrate`. A shared repository changed in another file stops as `target-changed`, nothing pushed, and the standing says so. In a root-local area, a change outside the area merges, and an **ownership change** inside it, in a file the run did not touch, stops. A run that fast-forwarded to the current target passes. |
| Live CLI, no archsage configured (source checkout against the operating registry) | `list` shows all 6 projects with committed tips. agops and forge workspaces are flagged *not at the committed tip* (their workspaces are behind after integrations). `show rts-vs-bot`: `study-rts` pinned `d2734db` = main. `read ppt wedo/wedo-ppt/README.md` is pinned by `pj-ppt@37831a6` and truncated as asked. `search agsfx loudness study` finds 37 matches in the root-local study. `runs forge provide` shows `run-007` published on `main` although the forge workspace is behind. `read forge .local/forge.env` is refused. |
| Live HTTP, a plain client (a private handler on a scratch loopback port, the real registry, no executor) | `GET api/projects` gives the 6 projects. `…/rts-vs-bot/file?path=study/study-rts/README.md` is pinned `d2734db`, 447 bytes. `…/forge/runs?workflow=provide` is live from workspace forge, 7 runs. `.local/` gives 403. An unknown project gives 404 `{"state": "absent"}`. |

## Not changed, noted

- Access bindings remain declarations (self-checks), as the contract has always said. Publication commits whatever a run changed in its copy. The guard protects integration targets. It does not sandbox a run.
- The browser does not yet show `integrate` on a binding; the file and `wfe projects show` do. Adding a binding in the browser does not set it.
- `--upstream` applies to the first submodule on a path. Deeper submodules follow its pins.

## Next

Step 3 builds `pj-archsage` against this release candidate: its guides, its index seeded from the old sages with provenance, its workflows, the trial targets' maintenance records, and the rts-vs-bot knowledge workflow that uses archsage's guide.
