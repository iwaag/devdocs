# run topic rework — plan

Source: `braindump.md`. Change the `run-` topic from a channel-agnostic
button into a project-bound, per-task conversation served by a new
`supercoder` role.

## Current shape (read 2026-08-18)

- `mission-*` serving: `zulip_listener.serve` → superdirector →
  `handle_superdirector_response` upserts the Work (`upsert_work`) and
  registers `task[N].md` as Sub-Works (`register_task_files`). This is the
  insertion point for the new preparation step.
- `run-*` serving: `handle_run` ignores the chatlog, picks `next_work()`
  across every `[AUTO]` project, runs the `coding` role in `main/`, reads
  `report.md` + `success.flag`, reports to Plane.
- **Already broken:** `work_prompt` loads `guide("run_coding", "guide.md")`
  but the guide directory on disk is now `run_supercoder/` — the current
  `handle_run` would crash at prompt build. The rework subsumes this.
- pyagag client already has what we need: `create_channel(..., folder_id=)`,
  `channel_folders()`, `channels()`, `channel_subscribers()`, `topic_write`.
  The realm has zero channel folders as of 2026-08-18, so "the folder the
  parent pj- channel sits in" may legitimately be "none".

## Step 1 — add the `supercoder` role

`agents.toml`:

```toml
# Serves one run- topic as a conversation with the developer, in the
# project folder. Same profile as coding today; separate role so they can
# diverge.
[roles.supercoder]
profile = "sonnet"
requires = []
```

(braindump says "agent profile" but structurally this is a role on the
existing `sonnet` profile, like `superdirector`.)

`coding` and `director` roles and their guides stay, unused — future scope.

## Step 2 — mission serving prepares the run surfaces

In `handle_superdirector_response`, immediately after
`register_task_files` (only on the `plan.md` branch):

1. Recover the parent Work issue and its label (`PA-12` form). Make
   `upsert_work` return the issue (or re-look it up with
   `find_issue_by_external`) — today it returns only a report line.
2. Find the folder of the parent `pj-<slug>` channel: read `channels()`,
   take that channel's `folder_id` (may be absent → create the channel
   without a folder; do NOT invent folders here).
3. Create channel `#work-<work label lowercased>` (e.g. `work-pa-12`) via
   `create_channel(name, description, principals, folder_id=...)`.
   - `principals`: the parent `pj-` channel's subscribers
     (`channel_subscribers`), so the developer and autolab are both in.
   - `description`: carry the binding the run handler will need back, e.g.
     `[AUTO] project: <slug>; mission: <channel>/<topic>` — parsing the
     channel name alone cannot recover the slug.
   - `create_channel` is subscribe-based and idempotent on re-planning.
4. For each registered Sub-Work, post the task content
   (`compose_document(title, description)`) into topic
   `run-task<N>-<work label lowercased>` in that channel, as the autolab
   bot. The bot being the last poster keeps the sweep quiet until a human
   posts — the topic waits, by design.
5. Re-planning (decided 2026-08-18): **update in place, not
   cancel+recreate** — on both sides.
   Plane: replace `cancel_sub_works` + `register_task_files` with a
   reconcile by serial. Match the parent Work's live children to the new
   `task[N].md` set via `sub_work_serial` (the external-id `#N` tail, which
   also matches old `@rev`-keyed issues, so no migration):
   - serial exists → update title/description, **state untouched** (Done
     stays Done — this is what keeps the Step-3 gate correct);
   - serial is new → create the Sub-Work (external key drops `@rev`:
     `<channel>/<topic>#<N>`);
   - serial disappeared → cancel that Sub-Work.
   Zulip, one-to-one:
   - updated → post "Updated by planner" + the new task content into the
     existing `run-task<N>-…` topic (bot post: sweep stays quiet);
   - new → create the topic with the task content, as before;
   - cancelled → post "Cancelled by planner" and resolve the topic (the
     post's message id feeds `resolve_topic`);
   - a **done** task whose content changed → update the issue, leave the
     resolved topic alone except a note that it changed after completion;
     whether to redo it is the mission conversation's call. Per braindump,
     this case likely forces a re-plan of the following tasks and needs a
     careful workflow — its behavior test is deferred; implement the
     minimal note-only handling and do not exercise it this episode.
   Mission-level cancel (`cancel.flag`) is the only remaining
   cancel-everything path: cancel live non-completed Sub-Works and archive
   the whole `work-…` channel. No re-creation ever follows it, so the
   archived channel's retained name cannot collide.
   The generation directory `<G>/` stays as the workspace double-act
   guard; it just no longer appears in Plane keys.
   Report lines for all of this go into `sections`.

## Step 3 — rewrite `handle_run`

New contract: a `run-` topic only works inside a `work-*` channel and is
bound to one Sub-Work.

1. Parse `run-task(?P<n>\d+)-(?P<work>.+)` from the topic; read the project
   slug and mission key from the channel description. A `run-` topic that
   does not parse, or sits in a non-`work-` channel, gets one explanatory
   reply (this replaces the old any-channel button; `dispatch` keeps routing
   all `run-` topics here).
2. **Gate:** find the parent Work in Plane, list its non-cancelled
   Sub-Works (`sub_works` + `sub_work_serial`), locate serial N and serial
   N-1. If N-1 exists and is not in a `completed` state group → reply
   `Please complete previous work` and stop (handler-side, no agent run).
   N=1 has no gate.
3. Serve through the shared `serve_topic` skeleton (ack, generation
   workspace under `.local/topics/<channel>/<topic>/<G>/supercoder/`,
   `chatlog.md`), so every later human post re-serves the topic — "leave it
   to supercoder and the user".
4. Prompt = placement lines (chatlog path, workspace path, the task
   notification text i.e. the Sub-Work title+description read from Plane)
   + `guide("run_supercoder", "guide.md")`, via `prompt_with_guide` — the
   same shape as `superdirector_prompt`.
5. Run role `supercoder` with `run_role(...)` in the **project folder**
   (`.local/projects/<slug>/`, where `main/`, `direction/`, `devlog/` are
   real directories — matching the rewritten guide), record under
   `RECORDS_ROOT / "supercoder"`, `WORK_TIMEOUT_SECONDS`, and keep
   `RunProgress` streaming into the topic.
6. **Closing the loop (needed for the Step-3 gate to ever open):** after a
   serving, if the serving's generation workspace holds `report.md`,
   comment it on the Sub-Work, move it to `completed` (`report_work`), and
   mark the run topic resolved (`TopicResult(..., resolve_after=True)` —
   the skeleton already supports it, same as mission cancel). The guide
   already says "if the developer agreed the task was done, create
   report.md" — the file is the agreement signal, and the generation
   directory guards it from being acted on twice.
7. **Devlog record (braindump 2026-08-18 addition):** on the same
   done branch, write the task content handed to the supercoder as
   `devlog/<mission dir>/task-<N>/work.md`, copy `report.md` beside it,
   and `commit_all_and_push` the devlog clone (`[AUTO] task <N> report
   for <work label>`) — the `serve_bmining` pattern, deterministic
   handler code, never the agent. `<mission dir>` (decided): 
   `<work label lowercased>-<slug of the Work title at first write>`,
   e.g. `pa-12-fix-title-screen/`, frozen at first write — later
   re-plans may rewrite the Work title (`upsert_work`), so the handler
   looks the directory up by its `<work label>-` prefix and only mints
   the name when no such directory exists yet. The current title lives
   inside `work.md` anyway.

Asset-labelled Sub-Works (decided 2026-08-18): after the previous-work
gate and before the supercoder launch, run `asset_gate` — order absent →
place the order and stop; in progress → report and stop; done → resign the
S3 key and append the asset note to the prompt. Since topics are now
per-task, a waiting asset blocks only its own topic, not the whole queue.

Kept as-is, unused: `next_work`/`eligible_works`, the `coding` and
`director` roles and guides. How a finished `work-…` channel gets closed
is next phase's decision — after the last task is done the channel simply
stays open this episode.

## Step 4 — guide check (braindump line 12)

`agent/guides/run_supercoder/guide.md` review:

- Typo: "crete" → "create".
- It never says **where** to write `report.md`. Add: write it into the
  serving workspace (the handler passes the absolute path), matching the
  superdirector guide's shape — otherwise the handler cannot find it.
- It doesn't mention the chatlog; the placement lines cover that, so the
  guide itself is otherwise fine for a conversational run.
- Delete nothing else; `success.flag` is dropped from the flow (report.md
  presence is the done signal — one signal, not two).

## Step 5 — tests

- `test_mission.py`: `upsert_work` returning the issue; the serial
  reconcile — update keeps state, disappeared serial cancels, `@rev`-keyed
  legacy children still match by serial.
- `test_zulip_listener.py` (fake client, as existing tests do):
  - mission serving creates `work-…` channel with the pj- channel's
    subscribers/folder and posts one `run-task<N>-…` topic per task;
  - topic-name parsing, wrong-channel reply;
  - gate: N-1 not done → "Please complete previous work", no agent run;
  - report.md → Plane comment + completed; no report.md → Sub-Work stays.

## Step 6 — rollout

Per localrule: commit, push to GitHub, then reflect —
`launchctl kickstart -k gui/$(id -u)/com.agdev.agautolab-zulip` on agstudio,
and the `setup_autolab_node.yml` playbook `--limit agautolab1` (Plane
credential source = `.local/plane/autolab.env`). Smoke: one mission on a
test project → work channel + topics appear → fire task1 → gate blocks
task2 until task1 is Done.
