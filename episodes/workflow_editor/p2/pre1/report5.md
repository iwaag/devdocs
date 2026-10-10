# workflow_editor p2/pre1 step 5 — readiness rehearsal

## Setup

- **Rehearsal area:** `pj-agdev/.local/workflow-editor-p2-rehearsal/`, created
  with `wfe setup … --port 8098`. It is separate from the person's area
  (`workflow-editor-p2`, port 8097) and from the p1 fixture.
- **Service:** `./wfe serve` from that area.
- **IDE agent:** a fresh subagent stood in for the IDE agent. It was told only
  that its working directory is the area and that the area's
  `CLAUDE.md`/`AGENTS.md` is its guide (as the IDE would load it). It got the
  suggested first prompt with the purpose "maintaining the release notes of a
  small command-line tool". Rehearsal rule: no reading of implementation code
  or episode documents. No one answered its questions.
- **The person:** played by `checks/rehearsal.ts` in a real browser (new
  script, modes `adjust`, `watch`, `parity`, `recovery`), plus Git commands
  for the Git route.

## Results by criterion

**1. Discovery from the documented directory: pass.**
- The agent read `AGENTS.md`.
- It found `./wfe help` and each `--help`, and the contract through the
  absolute path in the guide.
- It needed no implementation code and no further help.
- Its 13 commands are listed in its report. All of them were documented
  `wfe` commands, Git, or file edits.

**2. Create, register, write one workflow: pass.**
- `wfe create pj-relnotes …`: root repository, `project.yaml` with intent and
  four goals, devdocs as a real submodule with URL
  `../sources/relnotes-devdocs.git`, two commits with the person's identity,
  and registration.
- `wfe add-repo wedo/tool …`: a second submodule.
- `wfe workflow new release-notes`, then the YAML written by hand:
  five nodes (study → study → do → study → talk), bindings to devdocs and
  `wedo/tool`.
- `wfe validate`: 0 errors, 0 warnings. `wfe arrange`.
- No approval was recorded.

**3. Browser adjustments: pass.**
- `rehearsal.ts adjust`, in the workflow editor:
  - the `collect` description changed;
  - edge `collect → confirm` added through the inspector;
  - `confirm` dragged from (1240, 40) to (1240, 228);
  - Save.
- All three changes were in the YAML.

**4. Agent reads the person's changes and continues; the browser follows
without a reload: pass.**
- The agent re-read the guide.
- It found all three changes by comparing the file with its own earlier
  version (see finding F6).
- It rewrote the placeholder description, removed the redundant edge with a
  stated reason, and added a `publish` node next to the moved card. It did
  not run arrange, so the person's position was kept.
- The page stayed open throughout (`rehearsal.ts watch`). The change appeared
  919 ms after the file changed, with one navigation in total (no reload).

**5. Validation, approval and auto-layout through both routes: pass (7/7).**
- The browser's validation summary equals `wfe validate`.
- Intent and definition approvals given in the browser by "Rehearsal Person"
  are read as approved by `wfe status`, with the approver recorded.
- Browser Auto-arrange + Save and `wfe arrange` on the same saved file
  produced identical positions. Approvals stayed approved.
- The open view followed the CLI's write without a draft conflict.
- Git route: as the person, I committed the workflow in devdocs and then the
  gitlink in the root with `git`. `wfe status` showed a clean tree, the
  recorded commit and the workflow's editor link.
- VS Code's Source Control runs these same Git operations. The tool adds no
  Git layer of its own.

**6. Recovery: pass (8/8)**, on a scratch copy of the workflow, removed
afterwards. Covered:
- a file created while the project view was open;
- malformed content, then valid content;
- the service stopped (outage shown), the file edited while it was down, the
  service restarted (edit shown without a reload);
- rename (banner with a link to the new file);
- deletion (nothing recreated).

The Git state was checked separately: an untracked root file appeared in the
project view without a reload. The 21 bench probes from step 4 also pass on
the final code.

**7. Nothing overwritten: pass.**
- **The person's p2 area:** the checksums of every file, taken before the
  rehearsal and after the final refresh, differ only in the generated
  `AGENTS.md`. The guide template changed; the file still carries the
  generated stamp. The registry is unchanged and empty, and there is no
  project.
- **The p1 fixture registry:** mtime unchanged.
- **Your p1 processes:** the service on 8095 and the Vite server on 5175 are
  still running, untouched.
- **Tests and checks:** all run on temporary directories or the rehearsal
  area.

## Findings from the rehearsal agent, and what was done

| # | Finding | Action |
| --- | --- | --- |
| F1 | No route to create a new repository for a non-devdocs submodule; the agent made a bare repo and seed commit by hand. The person had no route either. | Fixed: `wfe add-repo <path> --new` and a "Create a new local repository" option in the project view's Add form, both `addNewRepository()`. Guide fact added. |
| F2 | Whether to commit was unclear. | Guide fact: the editor and `wfe` never commit except the initial commits; when to commit is ordinary Git work; a commit before handing over gives a diff baseline. |
| F3 | Whether the agent should approve was unclear. | Guide fact: an approval is the named person's to give, in the editor or by asking the agent to record it with their name. |
| F4 | Only the root URL was documented. | `wfe status` prints each workflow's editor link. The guide says so. |
| F5 | Node types and `study/`/`wedo/` had no meaning in the contract. | Contract section "Node types and repository conventions", taken from `design/braindumps/braindump2.md`. Bindings that no node uses are explicitly allowed. |
| F6 | The person's browser edits to a never-committed file could only be found from memory. | Covered by the F2 fact. No `wfe diff` was added; Git is the diff tool, given a commit. |
| F7 | `wfe arrange` would overwrite hand-placed cards. | `wfe arrange --help` now says so, and how to place one card. |
| — | "Approved notes" was ambiguous next to definition approvals; redundant (transitively implied) edges get no warning; a meaningless placeholder description cannot be "kept". | Recorded only. These are about wording and modelling, not missing capability. |

Found while fixing F1: a relative submodule URL built between a symlinked and
a real path (macOS `/var` and `/private/var`) climbed to the filesystem root.
URLs are now computed between real paths, and a test checks it. The
rehearsal project was not affected; its paths had no symlinked prefix.

Every guide addition keeps its "pre1 rehearsal" reference.

## Verification on the final code

- `npm run check`: tsc, 51 tests, build.
- Browser checks:
  - `checks/setup.ts` 15/15;
  - `checks/measure.ts` probes 21/21;
  - p1 step2, step3, step4 and e2e pass on a private fixture and service;
  - `checks/rehearsal.ts`: adjust 4/4, watch 2/2, parity 7/7, recovery 9/9.

The rehearsal area stays in place as evidence. Its service was stopped.
