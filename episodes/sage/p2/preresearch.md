# sage p2 — preresearch

Facts gathered on 2026-09-26 for the phase in `braindump.md`: archsage now
leads the sages, so it should be able to create, when a desire needs one, the
whole set of things a study needs. This file records what exists, what is
missing and what bit earlier episodes. It is not a plan.

## 1. Where things stand

archsage (`archsage` repo) runs the sages. Its role grant
(`agents.toml`, `[roles.archsage]`) is
`Read,Write,Edit,Glob,Grep,Bash(archsage:*),Bash(sagetree:*),Bash(agrefs:*)`.
Of the study kit it can create **only the sage**, with
`archsage sage add <name> --about … [--study <git url>] --guide-file …`.
Its guide asks it to name the research questions and "which existing study
each belongs to", but it has no way to create a study.

The four sages:

| sage | `study` in `sage.toml` | tree | queue (`tostudy/`) | in git |
|---|---|---|---|---|
| arxiv | GitHub `study-arxiv-trend` (the publish repo) | synced, last commit 2026-09-12 | empty | yes |
| growbox | `""` | empty | 1 note (project start, from argue-desk-garden) | yes |
| protoprey | internal Gitea `autodev/protoprey-research` (**main**, not publish) | synced | empty | **no, untracked** |
| worldtrend | `""` | empty | 1 note (round-1 discipline) | **no, untracked** |

The gaps this table shows:

- **worldtrend already has a study that was never attached.** The
  `pj-worldtrend` channel and its workspace (main at `autodev/worldtrend`,
  holding only the setup commit of 2026-09-18) exist. The sage still says
  `study = ""`, and nothing has run the study.
- **growbox has no study at all.** `pj-desk-garden` is a project, not a
  study.
- **Nothing consumes `tostudy/`.** This was the open handoff of sage p1
  (`../p1/report.md`, "Open handoff") and is still open. Two notes are
  waiting.
- **Sage definitions archsage writes at runtime are never committed.**
  `sages/protoprey/` and `sages/worldtrend/` show as `??` in the archsage
  repo. `sages/*/mainstudy/` and `sages/*/tostudy/` are ignored on purpose;
  `sage.toml` and `guide.md` are not.
- **An existing sage cannot be given a study by command.**
  `add_sage` refuses a sage that already exists, and `sage sync` only clones
  what `sage.toml` names. The README says "attach it with `archsage sage
  sync`", but attaching means editing `sage.toml` by hand.
- **Trees are refreshed only by hand** (`service/sync_knowledge.sh` /
  `archsage sage sync`, run by an operator).

A live case for this phase: on 2026-09-26 the Developer shared
`general@b3a9870…:study_aisvgs/` through agrefs. It holds `next_todo.md`
(collect everything about AI-made SVG images, publish to
`iwaag/study-aisvgs` later) and asked the Front Desk to register it as a new
study routine (`#front › front-desk-20260926-194416`, msg 11484). The desk
answered that it cannot (msg 11488). That is correct for the routine, but
wrong about the project channel (see §4).

## 2. The study kit: what each existing study needed

The `study` pattern is in `agautolab/agent/project_pattern.md` (lines
143–169, served by `autolab doc patterns`):

- `main/` holds the knowledge: a `README.md` plus a `main/<kind>/INDEX.md`
  per kind of item, always publish-ready.
- `publish/` is a clone of a public GitHub repository. It is filled by the
  `publish` routine, and **only the developer pushes it**.
- `methods/`, `reports/` and `RESEARCHPLAN.md` are not in the pattern doc.
  They come from the setup requests and from `agproject`.

The full kit, and who makes each piece today:

| piece | made by today | tool |
|---|---|---|
| `#pj-<slug>` channel (public, own folder, owners + Front + autolab, plus Opsroom Observer for the boards) | Omni Agent as Developer (4 studies); Front via `agproject` (worldtrend, protoprey-research) | one-off scripts; `agproject open --kind study` |
| `README_PROJECT.md` pattern marker in the workspace | Omni Agent by hand | none |
| workspace + `main/` repo on the internal Gitea | autolab, answering a `workplan-` setup post | `autolab project init-repo` |
| public GitHub repo (`iwaag/study-*`, CC0 LICENSE) | the developer, beforehand | none; agents hold no GitHub credential |
| `publish/` clone | autolab, at setup, only when the GitHub repo already exists | — |
| `#routine-study-<name>` channel + `guide` post | Omni Agent as Developer | one-off script `study_routine/study_industry/p1/.local/step3-create-routine.py` |
| sage with `--study` | archsage | `archsage sage add` |
| sage tree refresh | operator | `archsage sage sync` |

How each study was set up: studyarxiv (`agautolab/project_pattern/report3-4`),
studyrealworld (`study_routine/study_realworld/p1/report1`), studyuspolitics
(`routine_tests/p2/report1`), studyindustry
(`study_routine/study_industry/p1/report2-3`). All four were set up by hand,
none with `agproject`. The study_industry report lists the channel and the
marker as Deus Ex Machina work and as handoff candidates.

## 3. Routines

- A routine is **any public channel whose name starts with `routine-`**.
  The channels are filed in the `routine` folder (look it up by name), and
  the **newest post in the `guide` topic is the whole guide**. The
  agentroom board discovers routines by the name prefix
  (`agdevworld/agentroom/src/agentroom/routines.py`). A ✔ on `guide` retires
  the routine. A `display: <emoji> Title` line sets the board title.
- Channel description, exact text: "Routine `<name>`. `guide` = the process
  guide (newest post is the whole guide; posting there starts nothing).
  Each execution is its own `routinerun-<id>` topic here."
- Subscribers: Developer, Front and Opsroom Observer (the relay's mirror
  reads on that credential). Four older channels have only Developer and
  Front.
- Study guides share one shape (industry v1 msg 7015, realworld v1 5497,
  uspolitics v2 6393):
  - "In `#pj-<project>`, ask autolab for **one** mission and see it through
    to Done", followed by the goal as a block quote.
  - Operational bullets: `README_PROJECT.md` wins, read `methods/`/`reports/`
    first, bounded scope, honesty about data, keep `main/` publish-ready,
    never touch `publish/`, "one task, not one per source".
  - What the report names at the end.
  - Every post is signed "Developer" and headed "guide vN (date)"; a new
    version is a new full post.
- **Runs:** Front opens `routinerun-<id>` (its guide,
  `agfront/agent/guides/front/guide.md:39-78`), and the `routine_run` role
  drives it (`agfront/src/agfront/routine.py`). A run is started by asking
  Front, by the board's "Ask Front" button, or by opening the topic by hand.
  **There is no scheduler**; `refine_routine` p1 removed it on purpose.
- There is no CLI for creating a routine; `agag provision` covers agents
  only. The pyagag helpers exist: `channel_folder_by_name`,
  `create_channel(…, folder_id=…)` and `set_channel_folder` in
  `agag/zulip.py`.

## 4. Existing tools to reuse

- **`agproject open <slug> --kind study --doc <file>`**
  (`agfront/src/agfront/project.py`) already does most of the Zulip and
  autolab half:
  - What it does:
    - Refuses an existing or archived channel.
    - Creates the `pj-<slug>` folder and channel **with the provisioner
      credential**, and subscribes owners, Front and autolab.
    - Posts the document to `researchplan-<slug>`.
    - Asks autolab for setup only in `workplan-setup-<slug>`: `main/` with
      `methods/` and `reports/`, the plan in `main/RESEARCHPLAN.md`, no
      `publish/` yet.
  - What it does **not** do:
    - Write the `README_PROJECT.md` marker, so autolab scaffolds
      `direction/` and `devlog/` first (seen in both worldtrend and
      protoprey-research).
    - Create a routine channel.
    - Attach a sage.
    - Add Opsroom Observer.
  - Every post is made as Front.
  - It is granted to Front's `front`, `desk` and `argue` roles, but **only
    the `front` guide explains it**. The desk guide never mentions it, which
    is why the desk said the project channel was Developer-side work.
- **`archsage sage add`**: defines the sage. It cannot update one.
- **`agrefs`**: already granted to archsage. It is how a developer-supplied
  seed such as `study_aisvgs/` reaches it.
- **The study_industry `.local/step2-*.py` and `step3-*.py` scripts**:
  working reference code for the pj channel and the routine channel plus
  guide, including a read-back check of the guide after posting.

## 5. Pitfalls recorded elsewhere

- **The developer pushes `publish/` by hand.** v6 of the publish routine
  let an agent push (`study_realworld/p1/report4`), and v8 (msg 5482)
  withdrew that. Creating the GitHub repo is also the developer's job: agents
  have no GitHub credential, and auto mode's classifier blocked
  `gh repo create` (`sage/p1/report1.md:90`). So an agent-created study
  **cannot own a public tree on its own**. protoprey's sage reads Gitea
  `main` directly, which is the precedent for a study without a publish
  repo.
- **A new subscription does not reach an event queue that is already
  registered.** After creating a channel Front must read, restart Front's
  listener with nothing in flight (`study_industry/p1/report3`,
  `routine_tests/p2/report2`). Mirrors and `agag.listen` may have changed
  this; check before relying on either answer.
- **Zulip silently truncates posts over about 10,000 characters.** Read a
  guide back after posting it.
- **A missing marker triggers an unwanted scaffold** (`argue/p1/report.md`
  finding 5). `README_PROJECT.md` is in no repository, so it has no backup.
- **A topic has one home.** Do not reuse a `workplan-` or delegation topic
  name (`agentchat anchor` exists for moves).
- **autolab resolves setup and task topics before the requester agrees,
  and runs do not close their missions** (`study_industry/p1/report.md`).
- **Auto mode's classifier** denies compound state changes and account or
  secret work. Run one state change per command. Pushes issued in parallel
  have "succeeded" without landing.

## 6. Advice to the implementer

1. **Decide the split before writing code.** A reasonable reading of the
   braindump:
   - **archsage decides the substance**: the study's scope and research
     plan, the routine guide text, and the sage that reads the result.
   - **Mechanical creation is a tool archsage is given**, not a new agent.
   - **Front keeps running routines.**

   Front already delegates "run a routine". "Register a study" becomes one
   more thing Front hands to archsage, which keeps a single entrance per
   agent.
2. **Reuse `agproject` rather than writing a second channel creator.**
   Either extend it (`--routine`, marker, Observer subscriber) and grant it
   to archsage, or move the shared part into pyagag next to `provision`.
   Granting it means archsage's process needs the provisioner credential.
   That is one more holder of a realm-creating credential; record the choice
   in the plan.
3. **Give a routine command** (for example `agroutine open <name>
   --guide-file …`) built from the step3 script:
   - Resolve the folder by name.
   - Use the exact description text from §3.
   - Subscribe Developer, Front and Opsroom Observer.
   - Post the guide as its author, then read it back.

   Whether the guide should be signed by archsage instead of "Developer" is
   a real choice: every existing guide is Developer-signed, and nothing
   checks the sender, but humans read it as authority.
4. **Close the sage side as well:**
   - A way to set `study` on an existing sage.
   - Committing sage definitions (or deciding they are local state and
     ignoring them).
   - Stating whether a sage reads `publish` (public, reviewed, lags behind
     a human push) or `main` (fresh, internal). protoprey reads `main` and
     arxiv reads `publish`; the rule is unwritten.
5. **Update archsage's intro and guide** to say it can now open a study.
   Tell it when to use the ability (a desire the Developer decided on, as
   Front's guide says for `agproject`) and what it produces. Without the
   when, the tool is an Unexplained Chainsaw; with a list of prohibitions
   instead, it becomes Anxiety-Driven Guidance.
6. **Leave `tostudy/` consumption and tree refresh as an explicit scope
   decision.** Either one can quietly turn this phase into a scheduler
   project, and `refine_routine` removed scheduling on purpose.
7. **Backfill targets for acceptance:**
   - `aisvgs`: brand new, seeded from `general@b3a9870…:study_aisvgs/`.
   - `worldtrend`: the study exists but the sage is not attached and no
     routine exists.
   - `growbox`: the sage exists but there is no study.

   These are three different entry points into the same kit.
8. **Also fix the desk guide.** Add the `agproject` paragraph from the
   front guide, or route study creation to archsage explicitly. Otherwise
   the Front Desk keeps telling the Developer that work is theirs when a tool
   for it exists.

## Open questions for the developer

- Should archsage hold the provisioner credential itself, or ask Front
  (which already holds `agproject`) to create channels on its behalf?
- Does an agent-created study get a public GitHub repo (made by the
  developer on request), or does it stay Gitea-only like
  protoprey-research?
- Who signs a routine guide archsage writes?
- Are `tostudy/` consumption and tree refresh in or out of p2?
