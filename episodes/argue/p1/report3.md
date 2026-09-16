# Step 3 — study planning or concrete project setup

Date: 2026-09-16 JST. Plan: [plan.md](plan.md) step 3. Previous:
[report2.md](report2.md). Code only; the live run of the whole chain is
step 4.

## What an argue ends in

| Outcome | `outcome:` | Artifacts | Complete when |
|---|---|---|---|
| A concrete project | `project` | `#pj-<slug>` (folder of its own; the realm's humans, autolab and Front in it), its `goal` topic holding the final goal and how to proceed, and the workspace autolab prepared with `main/GOAL.md` and a `README_PROJECT.md` naming the channel and the argue | the channel and `goal` exist and autolab has answered `workplan-setup-<slug>` |
| A new study | `study` | the same on autolab's `study` pattern, with `researchplan-<slug>` in place of `goal` and `main/RESEARCHPLAN.md` in the workspace | as above, with the plan topic |
| A research plan in an existing study | `plan` | one `researchplan-<stem>` topic in that study's `pj-` channel, naming the argue | the topic exists |

Every artifact links back: the channel description says `opened from argue
<channel>/<topic>`, the document ends with the same line, and the workspace
request asks for `README_PROJECT.md` to name the channel, the topic and the
argue. The argue records `[selfnote][outcome] <kind> <channel>` and tells
its origin conversation where things ended, then is resolved. **Resolving
ends discussion dispatch only**: the project or study stays open, and
"complete" means planning and setup are done, not that the desire is
achieved.

## How Front gets there (`agent/guides/argue/guide.md`)

1. **The council, once.** With the desire on record Front invites
   `@**archsage**` for what knowledge exists, what would have to be
   researched and what domain nobody covers; it reuses that analysis and
   calls archsage again only for a new direction, a missing domain or
   conflicting evidence. Follow-ups inside a domain go to a sage directly
   (`@**archsage** sage:<name>`); cluster, project and media questions to
   the agents that answer for those.
2. **The judgement is Front's**, from the conversation and the advice:
   study first when exploring would widen, surface or firm up the idea
   (not only to fill a fact), else a project. No round count, no score. It
   says which and why before setting up; the conversation is the
   authorization, and it asks only when the human has not said which way.
3. **Setup is one command.** Front writes `GOAL.md` or `RESEARCHPLAN.md`
   in its workspace, then `agproject open <slug> --kind project|study
   --doc …` or `agproject plan <study> --doc …`. Nothing is started:
   no `workrun-`, no routine run; the guide says so and forbids posting
   into a `workrun-` topic.
4. **The outcome** is Front's reply — the desire by message id and what it
   became, why study or project, the artifacts by channel and topic, the
   next work with its owner and place, the open questions — ending in the
   block `outcome: … / target: pj-… / complete: true`.

## What the code does

### `agfront.project` — the `agproject` tool (`dbe0d6b`)

- `open`: refuses a slug in use or malformed; creates the channel with the
  provisioner credential this node already holds for autolab's filing
  (`AGFRONT_PROVISIONER_ENV`, default `pj-agdev/.local/zulip/provisioner.env`),
  members = realm owners + Front + the agent whose roster answers
  `workplan-` (read off the board, so autolab's id is never written down),
  folder `pj-<slug>` minted or reused; posts the document as Front under
  `goal` / `researchplan-<slug>`; writes `[selfnote][rootchat] <argue>`
  and the setup request into `workplan-setup-<slug>`, so autolab's reply
  names Front and the mention route brings it **back to the argue**.
- `plan`: the channel must exist; one post under a new `researchplan-<stem>`.
- Both print what they did, the channel link, and that nothing was started.

### `agfront.argue` completion

- `verify_outcome` reads the realm, not the reply: the channel exists; a
  `goal` post (project) or a `researchplan-…` topic (study/plan); and for
  `project`/`study` a post in `workplan-setup-<slug>` by somebody other
  than Front that is speech and not an ack. A missing piece is named in
  the visible reply (`(not complete: …)`) and the argue stays open.
  Completion also needs a recorded desire.
- On success: `[selfnote][outcome]`, a line into the origin conversation
  (under its live name), and `TopicResult(resolve_after=True)`.
- A serving keeps `threads/` now (`remotes_for_home` + the topic that
  called), so Front reads autolab's setup answer beside the chatlog.
- `handle_mention` serves an **argue home** with the argue role and no
  hand-off mention — a callback from `workplan-setup-` no longer runs the
  ordinary front role over the argue.
- The argue role's grant: `Read,Write,Glob,Grep,Bash(agentchat:*),Bash(agproject:*)`
  (`Write` for the document file in its own workspace).

## The plan's verification list

| Case | Test |
|---|---|
| An exploratory desire leads to a new study | `test_project.py::test_open_as_a_study_uses_the_study_pattern_and_the_plan_topic`; `test_argue.py::test_a_new_study_needs_its_plan_topic_and_the_workspace_answer` |
| A suitable existing study receives a concrete research plan | `test_plan_posts_into_an_existing_study_and_nothing_else`; `test_a_plan_in_an_existing_study_completes_without_a_workspace_answer` |
| A project-ready desire produces folder and channel with approach and goal | `test_open_creates_the_channel_posts_the_goal_and_asks_for_the_workspace` (channel, members, folder, `goal`, the anchored setup request, no `workrun-`); `test_a_project_completes_only_when_channel_goal_and_workspace_answer_exist` |
| Each outcome links back to argue | the same tests check the description, the document's last line and the root note |
| Completes without starting downstream execution | no `workrun-` topic in any post; `plan` posts one message; the guide forbids more |
| Waits for human input without background spending | nothing polls: a `(not complete: …)` reply leaves the topic with Front as last speaker until somebody posts |
| A claimed completion that is not there | `test_a_project_without_the_workspace_answer_is_not_complete`, `test_a_missing_channel_or_document_is_named`, `test_completion_needs_a_recorded_desire` |
| autolab's setup answer reaches the argue | `test_a_callback_into_an_argue_home_is_served_by_the_argue_role` |

agfront: **177 passed** (`uv run pytest -q`).

## Decisions worth knowing

- **The workspace is autolab's, so autolab makes it.** Front does not
  touch `.local/projects/`; it asks in `workplan-setup-<slug>`, the same
  shape `study_industry` p1 used, and autolab has answered such a request
  with one setup serving and no mission before. That serving is setup, not
  execution, and is the only paid run the setup costs beyond Front's own.
- **A new study gets its channel, its plan and its workspace, not its
  routine.** A `#routine-study-<x>` guide is a document a human has written
  by hand every time so far; the outcome names it as the next work and
  its owner rather than pretending Front wrote one.
- **The relay's `/ops` board reads a `pj-` channel by its description**
  (`[AUTO] project: <slug>; …`), which `agproject` writes in that shape.
- Nothing was posted to the realm in this step, so no handoff note is owed.
