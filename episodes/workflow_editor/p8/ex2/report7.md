# p8/ex2 step 7 — release and supplement of the existing projects

Step 7 of [plan.md](plan.md). **Deployed and supplemented on 2026-10-13.** The order was the one the plan asks for:
the release that can read the new format first, then the basic workflow.

## Before

- Production was idle: no queued or claimed job, no open attempt.
- Backup of the stores and host files: `.local/agdev/.local/backups/p8ex2-20261013-183941/`.
  - exec, notify, intake and present, via SQLite `.backup`;
  - `zulip.json`, `present.json`, `node-catalog.json`.
  - No store schema changes in this episode.
- One fix before the release (agdevworld `ce3387d`): `wfe workflow basics --push` now pushes with the host's
  Gitea credentials, as run publication does, and fetches afterwards. Before, it relied on whatever the
  keychain held.

## Release

| What | Value |
| --- | --- |
| Editor | agdevworld **`ce3387d`** (previous `6bdcd3b`), `wfe-release.sh deploy .local/agdev` |
| Release check | the release's own `npm ci && npm run check`: 255/255 tests, build OK |
| Health | `com.agdev.wfe-service` restarted; the service and `bin/wfe` name `ce3387d`; `:8098` 200; `/wfe/api/version` through the front proxy names `ce3387d` |
| Production runs | `bin/wfe run check` per workspace: all **11 usable** (rts-vs-bot 5, ppt 4, agsfx 2) |
| Execution guide | agautolab **`2e74777`** (wfexec reads it at each claim; no restart) |
| pj-agdev | pointer commit `4da29ae` |

## Supplement

`bin/wfe workflow basics --all --dry-run` first, then `--all --commit --push`:

| Project | Result | Commit (pushed to Gitea) | Bindings | Execution |
| --- | --- | --- | --- | --- |
| rts-vs-bot | added | `1eb3b7a` | `game .` editable; `rts study/study-rts` readonly | the host's agent; guide `agent/review.md` |
| ppt | added | `8c972d0` | `project .` readonly; `study` editable; `wedo` editable | `ppt-agent`, `sonnet`, guide `agent/guide.md` |
| agsfx | added | `ce6da58` | `project .` readonly; `lab`, `study`, `wedo` editable | `sfx-agent`, guide `agent/guide.md` |

- Each commit holds only `devdocs/workflows/freework.yaml`. The workspaces' untracked `.DS_Store` files are
  untouched.
- `git status -sb` shows `main...origin/main` in all three.
- **A second run** of `wfe workflow basics --all` reports `present` for all three and changes nothing.
- `wfe status` lists `freework.yaml` with 0 errors and 0 warnings. Both approvals are unapproved, as for every
  workflow of these projects.
- **Zulip:** the notify thread provisioned the per-workflow channels it makes for every workflow:
  `wf-agsfx.freework` (#280), `wf-ppt.freework` (#281) and `wf-rts-vs-bot.freework` (#282). freework declares
  no intake, so request topics there do not create runs.
- **A read-only look at the live editor** (WebKit, `localhost:8093/wfe/#/ws/ppt`): Simple, **Create workflow**
  visible, the list and the request form show *Create a presentation · Free work · Study*, no page errors.
  [production-ppt-simple.png](images/production-ppt-simple.png).

## Rollback (checked, not performed)

- **The editor:** `agdevworld/scripts/wfe-release.sh rollback .local/agdev` returns to `6bdcd3b`. Revert
  agautolab's `agent/guides/wfrun/guide.md` to `f1c466d` with it, so agents are not pointed at `--completing`.
- **What the old release does with the new records.** I checked this with `6bdcd3b`'s CLI on the scratch
  trial area:
  - it shows `freework.yaml` with one error, because it does not know `custom`, so no new freework run can
    be made;
  - it reads any run that used `--completing` as **unusable** (`mid-node`). The run is not damaged, and the
    new release reads it again.
  - Production has no freework run yet.
- **To remove freework:** `git revert` `1eb3b7a`, `8c972d0` and `ce6da58` in the three projects, and push.
  The three Zulip channels would then stay as empty channels. Archive them by hand if wanted.

The host notes are in `pj-agdev/.local/devenv.md`, "workflow_editor p8/ex2 on this host".
