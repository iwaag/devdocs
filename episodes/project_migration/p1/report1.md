# project_migration p1 — step 1: local areas in the workflow editor

**Released**: agdevworld `15b75cb` (feature) and `8178431` (wording), installed in the operating area. Previous release: `5f7a198`.

## What changed

A project can now declare **local areas** in `project.yaml`. A local area is a folder of the root repository that the project keeps as a knowledge area of its own, such as a study kept locally instead of as a shared repository.

```yaml
areas:
  study/study-sfx: {description: "Literature surveys …, kept in this repository"}
  lab: {description: "Experiments and small reusable tools"}
```

- **Binding.** A workflow binds an area exactly like a submodule. Access stays a path scope, so nothing in `shared/access.ts` changed.
- **Publication.** An area is part of the root repository. The existing publication already commits the root's changed paths, so nothing changed there.
- **Display.** The Project Editor's card is now "Repositories and local areas". An area sits in the group of its category (`study/…`, `wedo/…`, else Other) with a *local area* chip and its description. `wfe status` prints "local area (folder of the project root)". The dashboard still lists Gitea repositories only.
- **Validation.**
  - `area-missing` (error): a binding to a declared area whose folder is absent.
  - `project-area-path-invalid`, `-duplicate`, `-nested` (project warnings): the root, devdocs, `.local`, `..`, a repeated path, or an area inside another.
  - An area declared over a submodule path is reported on the submodule. An area that overlaps a submodule is reported on the area.
- **Contract.** `docs/contract.md` has a *Local areas* section, and the binding and validation tables name areas.

## Why a declaration and not "any folder"

The plan proposed accepting any existing folder of the root. Declaring areas in `project.yaml` was chosen instead, for three reasons:

- The project states which folders are its knowledge areas, so the editor can show them with a purpose.
- A typo in a binding still fails as `repository-missing` instead of silently binding a stray folder.
- The later "make this area its own repository" operation, which is out of scope here, has a list to start from.

The editor reads `areas` and never rewrites it. It is outside the editor's write shape for `project.yaml`, so a settings save keeps it as written.

## Verification

| Check | Result |
| --- | --- |
| `npm run check` (tsc, `node --test`, vite build) | 215 tests pass; `test/areas.test.ts` adds 4 |
| `checks/displayMode.ts` (browser) | all checks pass after its title expectation followed the new card title |
| Release deploy | service and `bin/wfe` report the new commit |
| `wfe validate` on ppt | both workflows valid |
| `wfe validate` on rts-vs-bot | `improve-game` valid. `build-game` keeps its two errors from before this change: it is still `ag.workflow.v1`, unread since workflow_editor p5 |
| Project Editor, Detail, 1440 × 1000 | the agsfx project shows `study-sfx` under Study and `lab` under Other as local areas with descriptions, and `wedo-sfx` under Wedo as a submodule |
