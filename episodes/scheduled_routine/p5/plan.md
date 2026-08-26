# scheduled_routine p5 — Plan

Braindump: `braindump.md`. A second study-pattern project: analyse GitHub
trending, summarise trending repositories. **Two summaries** are executed
in this phase — one through a hand-fired mission, one through a scheduled
fire — so p5 is both the second live test of the pattern flow
(`agautolab/project_pattern` episode) and the first time a pattern-managed
project runs an actual mission on the schedule.

Decisions taken up front, not re-opened:

- **Plane, stopgap option 1** from the project_pattern phase report,
  approved by the Developer (2026-08-26): **every pattern-managed project
  gets a Plane project automatically**. The pattern-managed gate in
  `init_project` changes from "do nothing" to "ensure the Plane project,
  touch nothing else" — folders, Gitea, clones stay entirely the agent's.
  The deeper "pattern projects and their ledger" design (a
  pattern-declared ledger folder, option 2) stays open for its own
  episode.
- Repo names are **not specified** in the request, so the agent should use
  `autolab project init-repo` standard names — the second live proof of
  the pattern doc's "if not specified" line.
- Trend evidence follows p4's finding 1: star counts and rank quoted from
  a machine-readable source (the GitHub API; the trending page has no
  API — scraping `github.com/trending` for the *pick* is fine, but the
  figures in the summary come from `api.github.com/repos/<owner>/<repo>`).
- `publish/` is committed by the agent on the Developer's approval and
  **never pushed by an agent** (pattern doc); the Developer pushes by
  hand. GitHub-side publication and credentials stay out of scope.

Experimental, non-public environment. Only **MUST NOT** lines are
prohibitions.

## Background the implementer should know

- Pattern-managed ritual (project_pattern Steps 3–4): create the
  `#pj-<slug>` channel with the autolab bot subscribed, then create
  `.local/projects/<slug>/README_PROJECT.md` by hand — the marker that
  keeps `init_project` away from the folders. The workplan request then
  builds the workspace (`autolab doc patterns`, init-repo, README).
- The gate to change is project_pattern Step 3's early return in
  `init_project` (keyed on `README_PROJECT.md`): it must now call
  `ensure_plane_project` (idempotent, host-side token — no serving ever
  sees it) before returning, and the returned string should say so. The
  step-order test for the pattern-managed case asserts: Plane ensured,
  no Gitea call, no folder created.
- **Known frictions when a pattern project executes a mission** — expected,
  recorded, not pre-fixed (they are the project_pattern phase report's
  hard-code list, now hit live for the first time):
  - `record_task_in_devlog` writes `<slug>/devlog/…` (`mkdir -p`), a
    folder the pattern never declared. Let it happen; note it.
  - `publish_main` pushes `<slug>/main` — wanted here.
  - `serve_bmining` would decline (no `direction/`) — not used.
  If the workplan→`start.flag`→workrun chain breaks anywhere else on a
  Plane-but-no-`direction/` project, that is a **finding**, and the fix is
  the smallest change that lets the mission run (say what it was).
- Slug suggestion: `ghtrends` (any valid name; keep it a lowercase slug).
  Standard repos then: `autodev/ghtrends` (main), `autodev/ghtrends-publish`.
- `init-repo` clones start on an **unborn branch** (project_pattern phase
  report, noticed-in-passing): the first commit into `main/` and
  `publish/` is also what makes `origin/main` exist. Watch whether
  `publish_main`'s push handles the first-ever push.
- Routine plumbing (p1–p3): standing text in `#front › routine-<name>`;
  fires go through `rtschedule`/`front-schedule` as one-shot events; the
  production dispatcher (`com.agdev.routine-dispatch`, 5 min) fires them.
  No `--now` acceleration is needed — p5's schedule test is a single
  one-shot fire minutes ahead, on the real clock.
- autolab roles already have `WebFetch`/`WebSearch`/`curl` — reaching
  github.com and api.github.com needs no new grant. Unauthenticated API
  rate limits (60/h/IP) are ample for two summaries.
- `WORK_TIMEOUT_SECONDS` = 1200 per task; one repo summary fits easily.
- **MUST NOT**: push `publish/` from any agent; give agents GitHub
  credentials; change the pattern-managed gate beyond the one
  `ensure_plane_project` call; expose the Gitea or Plane tokens to a serving; let a fix for a
  mission-path friction grow beyond that friction.

## Step 1 — project creation through the pattern flow

- Code first: the gate change above, its test, commit, push, pin bump,
  listener restart (the p2 ritual). This is the episode's only code
  change; keep it to the one call.
- Ritual: `#pj-ghtrends` channel (bot subscribed), marker
  `README_PROJECT.md`. No hand-made Plane: the first serving's
  `init_project` creates it — verify in the Plane UI after the workplan
  run and record that it appeared without a human call.
- As the Developer in `#pj-ghtrends › workplan-create`: "studyパターンの
  プロジェクトを作って。リポジトリは標準の名前でいい。mainにはGitHubの
  トレンド解析で選んだリポジトリの概要まとめを蓄え、publishにはレビュー
  済みだけを置く。"
- Check against the project_pattern Step 4 checklist: standard-name repos
  created via init-repo, README_PROJECT.md written, nothing pushed, no
  mission invented. Since `main/` is **fresh** this time, the README-copy
  question from the phase report does not arise — note whether the agent
  nevertheless described the folders in its own words.

Report `report1.md`: transcript references, workspace tree, `git remote
-v` both folders, README quoted.

## Step 2 — first summary, hand-fired mission

- In `workplan-…`: the mission — pick **one** repository trending on
  GitHub today (name the trending list and date in the summary), write
  `<layout the agent chooses>/…summary.md`: what it is, why it trends
  (figures from the GitHub API, endpoint named), license, and a short
  index so the next run can see what is already covered; commit. The
  layout inside `main/` is the agent's to choose and to record in
  README_PROJECT.md — do not prescribe INDEX.md; whether it converges on
  a p4-papers-like shape unprompted is worth one line in the report.
- Approve the plan, `start.flag`, watch the workrun end to end:
  Plane binding, task Done, `record_task_in_devlog` (the undeclared
  `devlog/` appears — record), `publish_main` first-ever push on the
  unborn branch.

Report `report2.md`: mission text, timeline, the summary quoted, Gitea
state of `main`, every friction hit and any minimal fix applied.

## Step 3 — second summary, through the schedule

- Standing text `#front › routine-ghtrends`, modelled on p2/p4's: ask
  autolab in `#pj-ghtrends` for a mission — one trending repository **not
  already covered in `main/`**, same summary shape; approving means
  acting; report home with the repo name and the trend figures.
- Via `#front › front-schedule`, one **one-shot fire** of `ghtrends` a few
  minutes ahead (real clock, production dispatcher — no `--now`).
- Watch: Front's whole chain on a pattern project; the duplicate check
  (does the second run read the first's index); whether the first-run
  permission ask recurs (p4 finding 6 said it is per-routine — this is a
  new routine, so **expect one ask**; answering it once is part of the
  ritual, and confirming that finding is a result).

Report `report3.md`: standing text, fire event ids and commits, run
timeline, Front runs, the second summary quoted.

## Step 4 — review and publish

- As the Developer, review both summaries in the workplan topic; ask the
  agent to move the approved ones into `publish/` per the pattern
  (commit, **no push**). Verify `publish/` is committed locally and its
  remote untouched; then the Developer pushes `publish/` by hand and
  records the command — the first full pass of the study pattern's
  review flow.

Report `report4.md`: the review exchange, `publish/` log local vs remote
before and after the hand push.

## Step 5 — phase report

`report.md`:
- Both summaries delivered? Trend figures verifiable in the named API
  responses (p4 finding 1 — this time the instruction was present from
  the start; did it hold in 2 of 2)?
- The mission path on a pattern project: every friction from the known
  list that fired, anything new, and the minimal fixes applied — this is
  the live evidence the "game as just another pattern" episode needs.
- The schedule leg: Front runs, the expected first-run ask, duplicate
  avoidance across two runs.
- Whether `ghtrends` should become a standing routine (daily?) and what
  the standing text would need; do not schedule it beyond the one fire.
- The Plane stopgap: what auto-ensured Plane cost or saved on a pattern
  project, as input to the still-open ledger design (option 2). Do not
  decide it here.

## Out of scope

Recurring schedule for ghtrends; GitHub push credentials; the pattern
ledger design (Plane vs pattern-declared folder); fixing
`record_task_in_devlog`'s undeclared `devlog/`; changes to Front, forge,
rtschedule, dispatcher; automating the pattern-managed declaration.
