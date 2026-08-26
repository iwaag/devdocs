# agautolab `project_pattern` — phase report

The project workspace is no longer the fixed `main/`+`direction/`+`devlog/`
triple. It is whatever the developer asks for, built by the agent from a
pattern document, and described in the project's own `README_PROJECT.md`.

- `agautolab` `3d46aa1` (Step 1, the `autolab` CLI), `bc77f4d` (Step 2, the
  guides), `0e0841c` (Step 3, pattern-managed workspaces).
- pj-agdev pins `9eb5487`, `864f275`, `d91d843`. Listener restarted after each.
- 201 tests pass (180 before this episode).
- Live: `#pj-studyarxiv` built to the study pattern on request, plus two
  negative tests and a clean re-run of one. Steps 1–4 have their own reports.

## Did the nondeterministic build match the pattern doc?

**Yes, and with no extra standing text.** Two guide paragraphs and one
document produced, in 28 seconds and 11 turns, a workspace that was
structurally correct on every count: both folders cloned from exactly the
URLs given (`agstudio:3000` host preserved, not normalised), no repository
created that nobody asked for, nothing pushed, no mission invented, and a
reply that said what it did and stopped. The negative tests were as clean: a
nonexistent pattern got a question back and an untouched workspace; a request
with no URLs got `autolab project init-repo` with the standard names, which is
the pattern document's "if not specified" line working as written.

Nothing in the guides describes a folder any more. The whole layout vocabulary
now lives in `agent/project_pattern.md`, reachable as `autolab doc patterns`,
and the agent found it unprompted — in Step 3's probe serving it cited the
command in answer to a question that never mentioned patterns.

**The one thing it did not get right was the README's content.** It wrote
`README_PROJECT.md` from the pattern document's sentences ("workspace where
summaries of knowledge are stored") when `main/` on disk was the p4 digest
repository with a `README.md` about two routines and a `papers/INDEX.md` over
four paper directories. It had read the clone — an hour later, asked something
else, it described `main/` as "already populated with paper summaries" — so
this is the pattern text winning over the observation, not a blind write.

That is this episode's one real finding, and it is a *guidance* finding, not a
code one: when a document supplies a ready-made sentence per folder, "create
it to explain how each folder works" gets answered by copying. See the
question below.

## Questions the agent asked back, and where the answers belong

Only one serving asked anything: the `zine` request.

> - Add a folder to studyarxiv's existing "study" pattern (e.g. a place for
>   zine-style output)?
> - Define a brand-new "zine" pattern — if so, what folders should it have and
>   what's each for?
> - Start an entirely separate project using this new pattern?

All three are good questions, and **two of them have answers that belong in
`project_pattern.md`**:

1. *Can a project add a folder outside its pattern?* — Yes; that is what
   `init-repo` and the "plain local folder" bullet already allow. The document
   does not say whether doing so keeps the project "a study project". One
   sentence: a pattern is a starting layout, not a constraint; added folders
   are recorded in `README_PROJECT.md` like any other.
2. *How is a new pattern defined?* — Not by the agent. The document should say
   that patterns are the developer's to add and that an unknown pattern is
   always a question back, which is what the guide already achieves but the
   document never states.
3. *Can one channel's workplan start a different project?* — **No**, and this
   one belongs nowhere near the pattern document: the channel is what says
   which project the work is for (`agent_standardize` p4). It is already true
   in the code and already stated in `README_DEV.md`. The agent asked because
   nothing it can read says it. A line in the workplan guide would be the
   place — the guide is where "what this serving is" lives.

**The README-content finding gives a fourth, and it is the important one.**
`project_pattern.md` should say that the per-folder text in a pattern is what
the folder is *for*, and that `README_PROJECT.md` says what the folder
*currently holds* — read the folder before describing it. Growing the
document, not the guides, is the right move: the guides now say "the pattern
document explains it", and every layout answer added there is one the guides
do not have to carry.

## What the marker-file declaration should become

Today a project is declared pattern-managed by a human creating
`.local/projects/<slug>/README_PROJECT.md`. It works, and it is honest about
who decides, but it has two flaws the live test exposed:

- **It is invisible.** Nothing but this report says the file is a declaration.
  A developer who deletes it silently re-arms the three-repo scaffold on the
  next serving.
- **The declaration text becomes documentation.** The studyarxiv run replaced
  the marker line; the studynourl run *appended* to it, so
  "pattern-managed; folders are created by the workplan on the developer's
  request" is now the first line of that project's README. A sentence written
  for `init_project` ended up addressed to the reader.

Three candidates, in order of preference:

1. **`autolab project declare [--pattern <name>]`** — the Developer runs it
   (or the agent does, on an explicit request), it writes a
   `README_PROJECT.md` stub whose text is documentation rather than a
   declaration, and `--help` explains what being pattern-managed means. It
   fits the CLI that now exists, it is the only candidate that makes the
   declaration *self-describing*, and it keeps the human act a human act.
2. **Automatic on the first pattern request** — the serving notices "make me a
   `<pattern>` project", writes the file, proceeds. Removes the
   set-up-by-hand step entirely, which is the direction of travel; but the
   scaffold has already run by then, so it needs `init_project` to be lazy
   rather than eager, which is a bigger change than it looks.
3. **A separate marker** (`.pattern-managed`, or a key in `agents.toml`) with
   `README_PROJECT.md` left purely as documentation. Cleanest separation,
   costs a second file, and loses the property that makes the current design
   pleasant: the thing that declares the project pattern-managed is the thing
   the agent has to write anyway.

Whichever wins, `init_project`'s gate should keep reading
`README_PROJECT.md` — an existing pattern project must not need migrating.

## The Plane decision

A pattern-managed project gets **no Plane project**. Consequences, all live:

- `workplan-` servings work; a `plan.md` is reported as a file and registered
  nowhere.
- **A pattern project cannot execute a mission.** `serve_run` binds a
  `workrun-` topic to a Plane Sub-Work; with no project there is nothing to
  bind. Preparation is the whole of what a pattern project can do today.
- `serve_bmining` answers with the no-direction reply, since a pattern project
  need not have `direction/`.

That was right for this episode — Step 4 only prepares workspaces — and it is
now the largest open question in the design. Three ways out:

1. **Give every pattern project a Plane project too.** Cheapest; one ledger
   for every project; costs the claim that a pattern project needs nothing but
   a channel.
2. **Let the pattern name its ledger folder** (a `devlog/`-shaped folder the
   pattern declares). Fits patterns properly, works offline, loses the board
   and the previous-work gate that `run_target` reads.
3. **Preparation-only until the ledger question is answered on its own
   terms** — which is where we are now, made explicit.

I lean to (1) as a stopgap and (2) as the answer, but this is a decision, not
a finding: nothing in this episode's evidence settles it.

## Can `game` now be "just another pattern"?

**Almost, on paper — not yet in the code.** `project_pattern.md` already
describes `game` exactly as it describes `study`, and the guides no longer
mention its folders. What still hard-codes it, listed and **not fixed**:

- **`project_init.init_project`** (`project_init.py:421-432`) — the scaffold
  itself. `repos = [(project, "main")]` plus `-direction` and `-devlog`, and
  the plain `devlog/` folder for main-only. This *is* the game pattern,
  executed deterministically. Every non-pattern project depends on it.
- **`project_init.is_main_only`** (`:378-382`) — reads the layout by looking
  for `devlog/` without `.git` and no `direction/`. A layout predicate with
  two folder names baked in.
- **`zulip_listener.record_task_in_devlog`** (`:872-897`) via
  `devlog_directory` (`:842`) — every completed task is filed in
  `<slug>/devlog/`. A pattern with no `devlog/` gets its report written into a
  folder the pattern never declared.
- **`zulip_listener.publish_main`** (`:905-919`) — the close-out pushes
  `<slug>/main`. Reads "main is not a repository; nothing pushed" when it is
  absent, so it degrades quietly, but the folder name is the contract.
- **`zulip_listener.serve_bmining`** (`:1064-`) via `direction_directory`
  (`:1036`) — brain-mining requires a `direction/` clone and declines
  otherwise. The whole feature is game-shaped.
- **`project_settings.project_name_from_direction`** (`:34-43`) — infers the
  project for the `director` role from `<slug>/direction/…`. Step 1 added the
  general `project_name_from_workspace` beside it; the direction-specific one
  is still what `role_run` calls for that role.
- **`project_archive.REPO_SUFFIXES`** (`:49`) — `("", "-direction",
  "-devlog")`. Archiving a project archives exactly three repositories; a
  pattern project's extra repositories would be left behind.
- **`cli.MAIN_FOLDER`** (`cli.py:37`) — new this episode, and the mildest of
  the list: it only decides that `main/`'s repository is `<project>` rather
  than `<project>-main`, which is the naming convention itself.

The shape of the eventual fix is visible in that list: the three folder names
appear as *the pattern the code was written for*, and each site needs to ask
the project which folder plays that role instead. `README_PROJECT.md` is
agent-written prose, so it cannot be that source of truth — something
machine-readable would have to sit beside it. That is a whole episode, and it
should not start before the Plane decision, because `record_task_in_devlog`
and `serve_run` are the same question asked twice.

## What this episode leaves working

- `autolab doc patterns` and `autolab project init-repo <folder>`, reachable by
  name from any role run, with the Gitea token never in an argv, an output or
  an error.
- Both guides free of folder descriptions; `README_PROJECT.md` and the pattern
  document carry it.
- `init_project` leaves a workspace holding `README_PROJECT.md` entirely alone
  — no Plane, no Gitea, no folder — and every existing project behaves exactly
  as before.
- Two pattern-managed projects, `studyarxiv` and `studynourl`, built by the
  agent on request.

## Out of scope, unchanged

GitHub push credentials for agents; automating the pattern-managed
declaration; migrating existing projects to patterns; Plane integration for
pattern projects; the review workflow that moves summaries into `publish/`;
Front, forge, rtschedule.

Two things noticed in passing, neither acted on:

- `init-repo` creates an **empty** repository, so its clone sits on an unborn
  branch (`No commits yet on main…origin/main [gone]`). `init_project` avoids
  this by seeding `.gitignore`. Nothing broke; it will first matter when
  something is committed into such a clone.
- A pattern-managed workspace lives under the gitignored `.local/` of the
  `agautolab` checkout, and a run can read that checkout's git history — one
  serving quoted a Step 3 commit message back at the developer. Harmless here;
  the workspace is less isolated than it looks.
