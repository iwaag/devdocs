# sage p2 — step 4 report: teaching the agents the workflow

## What changed

- **archsage's guide** (archsage `5f454b4`) now says it establishes
  studies as well as analysing knowledge. It covers:
  - What each tool makes: `agproject open` (plan, channel, workspace
    request), `agroutine create` (routine and guide), and `archsage sage
    add/attach`.
  - That the workspace is **pending** until autolab answers. The run ends
    with an `intent=progress` reply, which names nobody, and autolab's
    answer brings the conversation back.
  - That it should check before creating (`agproject status`, `agroutine
    show`, `archsage sage list`) and reuse existing studies.
  - The three entry shapes, with examples: a new study, a study that
    exists but is not connected, and a sage without a study.
  - That reports must keep setup complete, research complete and knowledge
    refreshed apart.
  - What a **study routine guide** contains. The runner asks autolab for
    one bounded mission in `#pj-<slug>`, with the goal as a quote and the
    operational context (read the marker and the existing
    methods/reports, stated scope, honesty about sources, `main/`
    publish-ready, one task, index rows, never touch `publish/`). Current
    acceptance behaviour applies: tasks close on agreement and the mission
    is done only when the acceptance is recorded. After integration, the
    runner asks archsage to refresh the sage. The guide stays under
    10,000 characters.
  - Refresh and queue handling (`sage sync`, `queue list/resolve`, and
    carrying queued questions into plans).
  - That the listener names the requester, so the reply should not repeat
    the mention (step 2 saw a doubled mention).
  - That in an argue archsage does not establish a study itself. It says
    what it would establish, and the facilitator asks in its own channel
    once the human decides.
  - That a developer's seed reference (such as `study_aisvgs/`) can seed
    the research plan and is cited by revision.

  No mandatory procedure was added. The tools' `--help` carries the usage.
- **archsage's introduction** (`params/intro.md`), posted live, has a new
  "Establishing a study" section. It says to ask in archsage's channel, in
  a topic of its own, with the decision, any reference, and whether
  initial research is wanted. It says what archsage makes and that it
  answers once, when setup is complete. It also says that setup is not
  research, that registering a routine runs nothing, that the routine
  should be run for research, and that archsage should be asked to refresh
  the sage afterwards. The sage list now shows each sage's study or "no
  study attached yet".
- **Grants and PATH.** `[roles.archsage]` holds `Bash(agproject:*)`,
  `Bash(agroutine:*)` and `Bash(agentchat:*)` beside `archsage`,
  `sagetree` and `agrefs` (granted in step 2). The console scripts are in
  archsage's `.venv/bin`, which `chat_environment` puts on every run's
  PATH. `archsage intro`, `sage update/attach/remove` and `queue` are under
  the existing `Bash(archsage:*)`.
- **Front** (agfront `1419002`):
  - The `front` and `desk` guides route study establishment, connection
    and sage attachment to archsage, "at the entrance its introduction
    names". No channel name is compiled into Front's guide.
  - Both keep routine execution with Front. When research was asked for,
    Front runs the study's routine once archsage reports; after acceptance
    it asks archsage to refresh the sage and reports the revision.
  - Both say Front opens a **project** itself with `agproject open --kind
    project` and that **no developer action is needed** for the channel.
    That fixes the desk's #11488 answer. The desk guide previously did not
    mention `agproject` at all.
  - The `argue` guide sends a new study to archsage (outside the argue)
    and keeps `agproject` for projects and for a plan in an existing study.
    Its finishing rule waits for archsage's report for a new study, and
    the `ag-argue` verification requires the study to be `ready` (step 1).
- **Deployed:** Front and archsage were kickstarted at 11:51Z with no run
  in flight, and both startup recoveries queued 0. The introduction was
  posted with `archsage intro`; `agentchat intro archsage-agstudio1` shows
  the new text and the sage list.

## Verification

- agfront 179 and archsage 37 tests pass.
- The introduction reads back from the board as posted.
- Whether a normal request progresses on the installed tools without the
  Omni Agent filling gaps is the subject of step 5.
