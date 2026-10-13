# project_migration p2 step 1 — gap check (2026-10-13)

A scratch clone of `trustedagent/pj-jevlab` `main` (outside the operating area, throwaway registry) got a trial `project.yaml` and one workflow, and was registered and validated by the installed release (`92de1da`).

```yaml
areas:
  study: {description: …}
  lab:   {description: …}
```

| Check | Result |
| --- | --- |
| `wfe register` | accepted |
| Structure | complete: root, `devdocs` as a folder, `study` and `lab` as local areas |
| `freework.yaml` bound to `lab` and `study` (editable) | valid, 0 errors, 0 warnings |
| Devdocs as a directory, with the existing `episodes/` inside | no issue |
| Top-level area `study` | accepted; shown with category **other**, because the category follows `study/<name>` only |
| Editor service | not involved; the validation reads files only |

**Gaps: none that block the migration.** The only visible difference from agsfx is the category label of `study`. A purely cosmetic remedy would be to move the folder to `study/study-jev`, which the plan declined (it would break the cross-references between 200+ commits of reports). Nothing in the system is changed in this phase.

Nothing was written to the operating area's registry, Gitea or the project.
