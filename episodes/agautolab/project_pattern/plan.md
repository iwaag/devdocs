# agautolab project_pattern — Plan

Braindump: `braindump.md`. Free the project workspace from the fixed
`main/direction/devlog` layout: the guides now point at a per-project
`README_PROJECT.md` (written by the agent) and at `autolab doc patterns`;
the pattern document `agent/project_pattern.md` (commit `31b1eb1`) names
the standard repo layout per pattern. This episode makes those references
real and watches autolab build a "study" pattern project on request.

Decisions taken in the braindump discussion (2026-08-26), not re-opened:

- README_PROJECT.md is **agent-written** ("if it doesn't exist, create
  it"); nothing generates it deterministically.
- Repository creation goes through a subcommand (`autolab project
  init-repo`) so the Gitea token never reaches the agent's context.
- `publish/` is never pushed by an agent; the developer reviews and pushes
  by hand. GitHub credentials for agents are out of this episode.
- Naming follows `project_init.py`: `autodev/<project>` for `main/`,
  `autodev/<project>-<folder>` for others.

The guide edits described in the braindump are already made but
**uncommitted** in `agautolab` (workplan_superdirector, workrun_supercoder);
they are committed in Step 2 after the typo fixes.

Experimental, non-public environment. Only **MUST NOT** lines are
prohibitions.

## Background the implementer should know

- There is **no `autolab` command today**: no `[project.scripts]` in
  `agautolab/pyproject.toml`, nothing in the venv. Every role's grant
  already includes `Bash(autolab:*)` and the workplan guide already says
  "The command `autolab doc patterns`" — the guide is ahead of reality
  until Step 1 lands.
- The serving env gets its PATH from the listener; check how `agentchat`
  reaches Front's runs (`agfront` pattern: the venv bin directory) and do
  the same for autolab's venv.
- Gitea plumbing to reuse: `project_init.py` — `load_gitea_config`
  (`.local/gitea/autolab-agent.token`), `ensure_gitea_repo`,
  `ensure_clone`, askpass via `AUTOLAB_GITEA_TOKEN_VALUE`. The subcommand
  wraps these; the token value must never be printed, echoed in errors, or
  passed as an argv the agent composes.
- **The listener runs `init_project(project)` on every serving**
  (`zulip_listener.py:284`, also 928, 1074). On a fresh
  `#pj-studyarxiv` this fires *before* the agent reads the request and
  would create Plane project + `autodev/studyarxiv` + the full three-repo
  scaffold. Step 3 must neutralise this for pattern-managed projects;
  the braindump's cut-off sentence ("なお、main/, devlog/,
  direction/フォルダの決定論的作成は") is resolved here as: **deterministic
  creation stays for existing projects; a workspace containing
  `README_PROJECT.md` is pattern-managed and `init_project` leaves it
  entirely alone** (no Plane call, no repo ensure, no clone checks).
- `is_main_only` reads the disk; `init_project` is idempotent per its
  docstring — extend, don't fork.
- Test rig: `uv run pytest` in `agautolab/`; restart listener after code
  change: `launchctl kickstart -k gui/$(id -u)/com.agdev.agautolab-zulip`.
  agautolab is pinned by pj-agdev — bump the pin after pushing
  (`localrule.md`: commit → push → reflect onto consumers).
- The live test reuses `autodev/papers` (scheduled_routine p4's digest
  repo) as `main/`. The papers GUI/dispatcher clone under
  `pj-agdev/.local/rtschedule` is unrelated; the `papers` routine keeps
  running against `.local/projects/papers/` — the studyarxiv clone is a
  second, independent checkout and must not disturb it.
- **MUST NOT**: expose the Gitea token (or its file path as readable
  content) to a serving; push `publish/` from any agent; let
  `init_project` modify a workspace that has `README_PROJECT.md`; delete
  or rewrite `autodev/papers` history.

## Step 1 — the `autolab` CLI

`[project.scripts] autolab = "agautolab.cli:main"`. Subcommands:

- `autolab doc patterns` — print `agent/project_pattern.md` verbatim.
  Unknown doc name: list the known ones, exit non-zero.
- `autolab project init-repo <folder> [--project <slug>]` — resolve the
  project from cwd (the serving works inside the workspace; reuse
  `project_settings.project_name_from_direction`'s approach or add the
  equivalent for any workspace folder), derive the standard repo name,
  `ensure_gitea_repo` + `ensure_clone` into `<workspace>/<folder>`,
  print the resulting clone path and remote URL. Refuse (non-zero, one
  clear line) when the folder already exists as a clone of something else.
- `autolab --help` / bare `autolab` — self-describing usage (Tool Giving:
  the help is the documentation the agent gets).

Token handling: read inside the process via `load_gitea_config`; never in
argv, never echoed. Tests: help text, doc output, init-repo on a fixture
workspace (mock the Gitea API as the existing project_init tests do),
the wrong-remote refusal, and one test asserting the token value appears
in no output.

Report `report1.md`: the CLI surface (`--help` quoted), test run, and the
command working from inside a real workspace.

## Step 2 — guides and pattern doc honest

- Fix in the uncommitted guide edits: `dosn't` → `doesn't`, sentence
  case; make "You edit those files" refer clearly to `README_PROJECT.md`.
- Re-read both guides start to finish against the new reality (no fixed
  folder description anywhere; `autolab doc patterns` exists now).
- Commit the guide edits together with these fixes; push; bump the
  pj-agdev pin; restart the listener.

Report `report2.md`: the final guide texts (diff from the pre-episode
state), pin bump commit.

## Step 3 — init_project respects pattern-managed workspaces

- `init_project` returns early (a distinct "pattern-managed, untouched"
  result string) when `<workspace>/README_PROJECT.md` exists. Existing
  projects (none has the file) keep the exact current behaviour; the
  step-order tests gain the new case.
- Decide and state what a **brand-new** pattern project needs on disk
  before its first serving: expected answer — nothing but the Zulip
  channel; the first serving still runs `init_project`, which (no
  README_PROJECT.md, empty workspace) would scaffold. To prevent that for
  studyarxiv, pre-create `.local/projects/studyarxiv/README_PROJECT.md`
  with one line ("pattern-managed; folders are created by the workplan on
  the developer's request") **by hand as the Developer** — creating the
  marker is the human act that declares the project pattern-managed this
  episode; automating that declaration is future work, noted not built.
- Plane: a pattern-managed project gets no Plane project in this episode;
  note it as an open decision.

Report `report3.md`: diff, tests, proof that a serving in a marked empty
workspace runs no Gitea/Plane call (log excerpt).

## Step 4 — the live test

Setup by hand: `#pj-studyarxiv` channel with the autolab bot subscribed
(the p2 report3 ritual); the marker file from Step 3. Confirm the intended
GitHub repo name with the Developer first — the braindump says
`study-arxiv-torend`, presumably `study-arxiv-trend`.

Then, as the Developer in `#pj-studyarxiv`, the braindump's request
verbatim (with the confirmed URL): a study-pattern project, `publish` →
the GitHub URL, `main` → `http://agstudio:3000/autodev/papers.git`.

Success checklist (decided now, judged in the report):
- both folders exist as clones of exactly the given URLs (`git remote -v`);
- no new Gitea repo was created (papers URL was given; nothing else asked);
- `README_PROJECT.md` describes both folders, the study pattern, and the
  publish/no-push rule — and reflects `main/`'s **existing** content (the
  p4 INDEX/papers structure), i.e. the agent read before writing;
- nothing pushed anywhere; `publish/` clone clean;
- the reply reports what was made and stops (no mission invented).

Negative tests, one post each:
- a nonexistent pattern ("make a `zine` pattern project") → asks back,
  creates nothing;
- a study request **without** repo URLs → either asks back or uses
  `init-repo` with standard names; both acceptable, record which and
  whether the choice matched the pattern doc's "if not specified" line.

The GitHub `publish` remote will not accept an agent push (no
credentials); that is by design — the checklist's "nothing pushed" covers
it. If the clone itself fails because the GitHub repo does not exist yet,
create it empty by hand (Developer account) and note it.

Report `report4.md`: full topic transcript references, the checklist with
verdicts, README_PROJECT.md quoted, workspace tree.

## Step 5 — phase report

`report.md`: did the nondeterministic build match the pattern doc without
extra standing text; every question the agent asked back and whether the
answer belongs in `project_pattern.md` (grow the doc, not the guides);
what the marker-file declaration should become (auto on first pattern
request? an `autolab project declare` subcommand?); the Plane decision;
whether `game` pattern (the current fixed layout) can now be expressed as
"just another pattern" and what still hard-codes it (`init_project`'s
scaffold, `record_task_in_devlog`, `serve_bmining`) — list, do not fix.

## Out of scope

GitHub push credentials for agents; automating the pattern-managed
declaration; migrating existing projects to patterns; Plane integration
for pattern projects; the review workflow that moves summaries into
`publish/`; touching Front, forge, rtschedule.
