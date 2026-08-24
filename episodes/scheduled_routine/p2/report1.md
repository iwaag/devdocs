# scheduled_routine p2 — Step 1 report: main-only project initialisation

## The flag

`init_project.py <slug> --main-only` (agautolab commit "init_project
--main-only: one repo, devlog/ as a plain local folder"). Full stays the
default; main-only is the option. Reasons:

- Every runtime caller (`serve_run`, the superdirector serving,
  `serve_bmining`) calls `init_project(slug)` with no layout. Making
  main-only the default would silently change what a `workplan-` in a
  brand-new `pj-` channel creates; the four existing full projects were made
  that way by exactly that path.
- The routine project is the new, lighter case; the flag is where the
  Developer says so, once, by hand.

`init_project(project, *, main_only=None)`:

- `None` (all runtime callers) **reads the layout off the disk**:
  `is_main_only(root)` = `devlog/` exists without `.git` and no `direction/`.
  A re-run from a serving therefore never clones two repos into a main-only
  project. A project not on disk yet is full, as before.
- `True`: Plane project, Gitea repo + clone + `.gitignore` for `main`,
  then `devlog/` as an empty plain folder. On an existing full project it
  touches `main/` only — `direction/` and the `devlog/` clone are left as
  they are (test).
- `False` on an existing main-only project is refused
  (`ProjectInitError … refusing to turn it into a clone`), so nothing ever
  turns the local record into a repository. The CLI without the flag passes
  `None`, so a plain `init_project.py rtnotes` re-check also works.

Backward compatibility: the four three-repo projects keep working — the
inference sees `devlog/.git` and takes the full branch, and the step order
test for full projects is unchanged. `project_archive` already returns
`absent` for a Gitea repo that does not exist, so archiving a main-only
project needs no change.

Tests (`tests/test_project_init.py`, 16 passed): the existing step-order
test unchanged; three new — main-only step order + plain-folder devlog +
runtime re-run infers main-only; main-only on a full project leaves it
alone; full on a main-only project is refused.

## Fresh project tree

`uv run python init_project.py rtnotes --main-only` → `success`
(Plane project "Rtnotes", Gitea `autodev/rtnotes`):

```
.local/projects/rtnotes
.local/projects/rtnotes/devlog          (plain folder, empty)
.local/projects/rtnotes/main
.local/projects/rtnotes/main/.git
.local/projects/rtnotes/main/.gitignore  (".local/")
```

`main` log: `2c3b1e6 Ignore .local/`. A second run without the flag
(`init_project.py rtnotes`) printed `success` and created nothing else — no
`direction/`, `devlog/` still a plain folder.

Not yet done in this step: `record_task_in_devlog` still calls
`commit_all_and_push` on `devlog/` unconditionally (Step 2), and the
`workplan-`/`workrun-` guides carry an uncommitted "if exists" edit in the
working tree that Step 2 commits alongside.
