# Plan: asset_pipeline1 p1

Realize `braindump.md`: superdirector planning, asset-labelled works ordered from
agforge via `create-` topics, question-answer loop, aesthetics stopgap.

Ground rules (deliberately minimal):

- This is an experimental, non-public environment. No backward compatibility
  needed — this phase is destructive; delete/replace old flow freely.
- Do NOT add any cost-saving logic for agent runs. Sonnet cost is accepted.
- Keep the shared `agag` library conventions (Plane keying, guide loading,
  sweep pattern) — both agents depend on them.
- Implementation details beyond what each step states are implementer's
  discretion.
- Write `report[step].md` per step in this folder.

Repos touched: `pj-agdev/agautolab`, `pj-agdev/agforge`.
Deploy at the end: push agautolab to GitHub, then
`ansible-playbook … setup_autolab_node.yml --limit agautolab1` (and
`--limit agstudio`); agforge runs natively on agstudio under launchd
(`launchctl kickstart -k gui/$(id -u)/com.agdev.agforge-zulip`).

## Step 1 — Repair stale guide references (agautolab)

Guides were renamed (commits 7889c4a→a468621) but code still loads old paths;
every mission-/run- serving currently dies with `GuideError`.

- `src/agautolab/zulip_listener.py:144` → `guide("mission_front", "guide.md")`
- `zulip_listener.py:203` → superdirector guide (rewired in Step 2 anyway)
- `zulip_listener.py:276` → `guide("run_coding", "guide.md")`
- `tests/test_zulip_listener.py:385` uses the old front path.
- Guides say `devlogs/`; `project_init.py` creates `devlog/`. Align on `devlog/`.

Verify: run the listener test suite.

## Step 2 — superdirector role and mission-flow rework (agautolab)

Replace the disposable per-topic coding workspace with a superdirector run in
the persistent project folder.

- `agents.toml`: add role `superdirector` on the existing `sonnet` profile.
  No new profile/model.
- `src/agautolab/role_run.py`: add `superdirector` to `ROLE_ALLOWED_TOOLS`
  with the writable set (`WORKING_ALLOWED_TOOLS`, line 16). It writes files.
- `zulip_listener.py` `handle_front_response()` (line 213): after front writes
  `new_mission.md`, copy it into `.local/projects/<slug>/` (the folder
  containing `main/`, `direction/`, `devlog/`) and run role `superdirector`
  there with `agent/guides/mission_superdirector/guide.md`. Drop the
  `<N>/coding/` sibling-dir mechanism.
- Parent work: `mission.py` `upsert_work()` (line 382) currently uses
  `new_mission.md`. Change to: title from the mission, description from the
  superdirector's `plan.md`.
- Sub-works: keep `register_task_files()` (mission.py:435) but point it at the
  project folder. Add `[Asset]` handling: if a `task[N].md` ends with
  `[Asset]`, strip the marker and register with labels `["AUTO", "asset"]`
  (`ensure_issue` already takes labels; `ensure_label` at mission.py:199 caches).
- Cleanup: after successful Plane registration delete `plan.md`, `task*.md`,
  `new_mission.md` from the project folder. On failure leave them for retry.
  Plane holds the canonical copies from here on (Step 6 relies on that).

Hints: `TASK_FILE` regex at mission.py:57 matches only `task[N].md`; `plan.md`
is invisible to it — fine. Timeout for the old coding split was 600 s
(zulip_listener.py:79); superdirector reads more context, give it ≥ the work
timeout (1200 s).

Verify: extend listener tests with the stub/fake harness: front →
superdirector → parent work from plan.md → asset-labelled sub-work → files
cleaned up.

## Step 3 — aesthetics stopgap (agautolab)

In `src/agautolab/project_init.py` `init_project()` (line 316): when ensuring
the `<project>-direction` clone, create `aesthetics.md` containing
`2D retro digital game art style` if absent, commit and push to gitea.
Idempotent so existing projects gain it on next init.

## Step 4 — agforge: question.flag mention, S3 key in reports, /api/resign

- **question.flag**: in `src/agforge/create_topic.py`, after the generator run,
  if `question.flag` exists in the workspace, prefix the reply post with a
  Zulip mention (`@**Full Name**`) of the last non-agforge sender in the topic
  (sender info is already in the fetched message list used for `chatlog.md`,
  lines 93–95). Add one line to the create-generator guide: "if you must ask
  the requester a question instead of planning, write the question and touch
  `question.flag`."
- **S3 key in reports**: `runcreate_topic.py` delivery (lines 152–159) and the
  Plane report comment currently carry only the presigned URL (dies in 60 min,
  `generate.py:15`). Include the S3 object key (e.g.
  `files/<date>/<uuid>.zip`) in both.
- **/api/resign**: add to `src/agforge/request_service.py` a
  `POST /api/resign {"key": "<s3 key>"}` that calls the existing presigner
  (`generate.py` `upload_and_presign` — refactor so signing an existing key
  needs no upload) and returns a fresh URL. Pure script, no agent run. This is
  a correctness device: agautolab re-signs right before launching coding so
  the URL cannot expire mid-run (coding timeout 1200 s vs unknown remaining
  TTL).

Hint: the bucket is `agforge` at `http://agstudio.local:9100` (MinIO, boto3,
`generate.py:116-144`). Reload the launchd services after changes.

## Step 5 — agautolab: asset state machine on run- topics

In `zulip_listener.py` `handle_run()` (line 289), after `next_work()` picks a
work, check its labels for `asset` (issue rows carry label ids; see
`eligible_works` mission.py:292 for the lookup pattern). Non-asset works run
unchanged. For asset works, decide by Plane — the ledger is agforge's Work
keyed `external_id = "pj-<slug>/create-asset_<work_id>"`, source `agforge`:

1. **No agforge work** → post the order into topic `create-asset_<work_id>`
   in the project channel, then report "asset ordered" to the run- topic and
   finish the serving. Order text = task title + description + the content of
   `direction/aesthetics.md`. Write it self-contained: agforge's front reads
   the whole topic chatlog and its generator plans from it.
2. **agforge work exists, not completed** → report "asset in progress", do
   nothing else, finish the serving. Do NOT fire `runcreate-`; the Omni Agent
   triggers it manually this episode.
3. **agforge work completed** → recover the zip URL: prefer the S3 key from
   the delivery post / Plane comment (Step 4), call `/api/resign` on
   `http://localhost:8092` for a fresh URL. Then run coding with the composed
   prompt: run guide + `\n\nNote: the asset required by this work can be
   downloaded from the URL below. If the asset does not match the spec, try
   to compromise; only if truly unacceptable, treat the work as failed:\n<url>`
   (director check deliberately skipped this episode).

The autolab work stays `unstarted` through states 1–2, so `next_work()` keeps
picking it and other works are blocked until the asset lands — accepted for
now.

Naming: underscore before the id (`create-asset_<id>`) — agforge's listener
matches only the `create-` prefix (`zulip_listener.py:29-32`), the rest is
free; underscore avoids collision with agforge's own
`create-YYYYMMDD-HHMMSS-<id>` hyphenation. One topic per asset work is load-
bearing: agforge keys one Plane Work per `<channel>/<topic>` — reuse would
overwrite the ledger entry and mix specs in one chatlog.

Verify: stub-harness tests for the three states; check the run- topic report
text and that no runcreate- post is ever emitted.

## Step 6 — agautolab: answer agforge questions on create- topics

- Add `create-` to the sweep `topic_filter` (zulip_listener.py:428) and a
  dispatch branch: react ONLY when the topic's last message mentions the
  autolab bot. This mention gate is the loop breaker between the two bots —
  without it agforge (reacts to any non-forge post) and autolab would ping-pong
  forever.
- On reaction: parse `<work_id>` from the topic name `create-asset_<work_id>`;
  fetch from Plane the asset sub-work (description = task) and its parent
  (description = plan.md). Write `chatlog.md`, `plan.md`, `task.md` into a
  generation dir `.local/topics/<channel>/<topic>/<N>/answer/` (reuse
  `generation_dir`, zulip_listener.py:123).
- Run role `superdirector`, cwd `.local/projects/<slug>/`, guide
  `agent/guides/create_answer_superdirector/guide.md` (create it: read the
  three referenced files plus `main/` and `direction/` as needed, write
  `answer.md`). Prompt via the existing `prompt_with_guide` placement-line
  pattern pointing at the three files.
- Post `answer.md` to the create- topic. Autolab is then the last non-forge
  poster, so agforge's sweep resumes the conversation.

Verify: stub-harness test — mention-gated reaction, no reaction without
mention, answer posted.

## Step 7 — End-to-end check and deploy

- Local E2E on agstudio with real sonnet runs: mission- topic → plan +
  `[Asset]` task → run- fires order → (Omni Agent posts `runcreate-`) →
  delivery → run- launches coding with the asset URL. Record evidence in
  `report7.md`.
- Deploy agautolab (GitHub push → ansible, both limits), kickstart agforge
  launchd jobs.
- Leave the Deus Ex Machina one-liner for the manual runcreate- trigger in
  this episode doc.
