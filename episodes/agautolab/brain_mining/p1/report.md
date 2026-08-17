# brain_mining p1 — report

Reconciled: autolab serves `bmining-` topics. Commit `1bc25fd` on
`iwaag/agautolab` `main`, pushed and live on the agstudio listener
(startup log shows `prefixes ('mission-', 'run-', 'create-', 'bmining-')`).

## What was built

A `bmining-` topic in a `pj-*` channel is a brain-mining conversation.
Each serving, through the shared `serve_topic` skeleton (ack first,
always answer, re-serve when a human posted during the run):

1. `init_project` guarantees the three clones exist.
2. The full topic history is written as
   `direction/.local/work/chatlog.md` — re-placed every serving,
   never accumulated.
3. One `director` run happens in the `direction/` clone, prompted with
   the chatlog placement line plus
   `agent/guides/bmining_director/guide.md` (now committed).
4. If the direction clone is dirty afterwards, the handler — not the
   agent — commits everything and pushes
   (`commit_all_and_push`: `git status --porcelain` gate, `add -A`,
   authored commit, `push origin HEAD:main`). The reply then carries
   "recorded notes committed and pushed".
5. The director's reply is posted verbatim.
6. `.local/work/` is removed after the reply (also on failure). The
   clone's `.gitignore` (`.local/`), not the cleanup, is what keeps the
   chatlog out of the commit.

## Decisions taken (from the discussion on the braindump)

- **The director role can now write.** The guide asks it to record
  important information into the workspace, but the role carried the
  read-only grant (`Read,Glob,Grep`) and sat in `READONLY_ROLES`. It
  had no live call site — bmining is its first — so it moved to
  `WORKING_ALLOWED_TOOLS` with zero blast radius.
- **Commit/push is deterministic handler code**, never delegated to the
  agent: the recording persists even when the run forgets git exists.
- **Chatlog is disposable**: replaced each serving, deleted after, so a
  serving never reads a stale conversation and nothing transient can
  leak into the direction repository.
- The director run gets the work timeout (1200 s): it reads the whole
  direction clone before answering.

## Changes

| File | Change |
|---|---|
| `src/agautolab/zulip_listener.py` | `BMINING_TOPIC_PREFIX`, `handle_bmining`/`serve_bmining`, `run_director`, `bmining_prompt`, dispatch route (inside the `pj-*` gate), sweep prefix |
| `src/agautolab/role_run.py` | `director` → working tool set, out of `READONLY_ROLES` |
| `src/agautolab/project_init.py` | `commit_all_and_push` |
| `agent/guides/bmining_director/guide.md` | committed (was untracked) |
| tests | 9 new cases; 140 pass |

## Verification

- `uv run pytest tests/` — 140 passed.
- `com.agdev.agautolab-zulip` and `com.agdev.agautolab-gateway`
  kick-started on agstudio; the listener's startup line confirms the
  new prefix tuple. No live `bmining-` conversation has been run yet —
  the first real topic is the natural next check.

## Notes

- agautolab1 runs no Zulip listener (only the gateway, which does not
  use the director role), so no node deploy was needed; the next
  regular playbook run will carry the commit there anyway.
- `agent/guides/autolab-front/` was also untracked but belongs to
  another episode; left alone.
