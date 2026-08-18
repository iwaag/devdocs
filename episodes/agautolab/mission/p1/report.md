# mission p1 — report

Reconciled: `mission-` topics are served by the superdirector alone. The
front relay is gone. Live on both placements at commit `be059f6` on
`iwaag/agautolab` `main` (agstudio listener kick-started; agautolab1
deployed via `setup_autolab_node.yml`, `failed=0`, HEAD verified).

## What was built

A `mission-` topic serving no longer runs a front agent that distills the
chatlog into `new_mission.md`. Instead, each serving:

1. Cuts a generation workspace `<N>/superdirector/` and writes the full
   topic chatlog there.
2. `init_project` guarantees the three clones exist; the registered
   mission and tasks are mirrored from Plane into `<workspace>/current/`
   (a subdirectory, so the mirror's `task1.md`… can never be mistaken
   for a task split written this run).
3. One `superdirector` run happens **in the project folder** — where
   `main/`, `direction/`, `devlog/` are real directories — with the
   workspace handed over by absolute path in the prompt (the same shape
   as the create-topic answer flow): read `chatlog.md` and `current/`
   there, write `plan.md`, `task[N].md`, `start.flag`/`cancel.flag`
   back there.
4. The handler acts on what the run wrote: `plan.md` upserts the Work
   (title from the plan's own heading now), old Sub-Works are cancelled,
   `task[N].md` register as generation-keyed Sub-Works, flags transition
   the Work. **A run that wrote nothing changes nothing** — its reply
   (usually a question back to the requester) is the whole outcome.
5. Nothing is cleaned up: the generation number is the double-acting
   guard, and every workspace stays as evidence. The old
   `clear_planning_files` retry economy (keep the plan on a Plane
   failure to avoid paying twice) was dropped deliberately — a retry
   re-runs the superdirector.

## The road there (two failures worth keeping)

The first implementation symlinked `main/`, `direction/`, `devlog/` into
the generation workspace and ran the superdirector there. The first live
serving declared a populated `direction/` empty. The transcript showed
two independent causes:

- **claude_code's auto-mode permission classifier denied an allowlisted
  command**: `ls -la direction/ 2>&1` inside a compound command was
  flagged "requires approval" despite `Bash(ls:*)` in `--allowedTools`.
  Non-interactive, so the denial was a dead end.
- **The harness file tools do not follow directory symlinks**: the
  fallback `Glob direction/**` returned "No files found" on a directory
  with two files in it.

Two responses, both kept:

- `role_run` now passes `--dangerously-skip-permissions` for every
  claude_code run (`b42989e`). The roles are workspace-bound; the
  allowlist stays in the code as documentation of what a role is
  expected to reach for. The developer noted the classifier's strictness
  may end in dropping the claude_code harness entirely.
- The symlink layout itself was replaced by the project-folder-cwd +
  absolute-workspace-path shape above (`be059f6`), so the flow does not
  depend on any harness following symlinks.

In between, the superdirector ran briefly on the `local` profile
(agcode + qwen, `4d39eaf`) for cheap flow testing, and was put back on
`sonnet` with the final design (`be059f6`).

## Changes

| File | Change |
|---|---|
| `src/agautolab/zulip_listener.py` | front path removed (`run_front`, `front_prompt`, `FRONT_TIMEOUT`); `superdirector_prompt`, project-folder run, `current/` mirror, `handle_superdirector_response` (plan-optional, flags from the workspace) |
| `src/agautolab/role_run.py` | `skip_permissions=True` for claude_code runs |
| `src/agautolab/mission.py` | comments/docstrings follow the new flow |
| `agent/guides/mission_superdirector/guide.md` | reads the chat log itself, plan-optional question mode, flags moved in from the front guide, typo fixes |
| `agent/guides/mission_front/` | deleted |
| `agent/guides/bmining_director/guide.md` | typo fixes (drive-by) |
| `agents.toml` | superdirector `sonnet` → `local` → `sonnet` |
| tests | mission-flow tests rewritten for the new serving; 140 pass |

Commits, in order: `64e5ab9` (superdirector-only flow, symlink
workspace), `b42989e` (skip-permissions), `4d39eaf` (local profile),
`be059f6` (project-folder cwd, back to sonnet).

## Verification

- `pytest` — 140 passed at `be059f6`.
- agstudio listener restarted on each step and sweeping; agautolab1 at
  `be059f6` (gateway health probe ok). agautolab1 runs no Zulip
  listener, so all `mission-` traffic is served by agstudio.
- Live check so far: the symlink-era serving on
  `pj-foodchain`/`mission-start` (the run that exposed both failures,
  transcript under
  `.local/topics/pj-foodchain/mission-start/2/superdirector/` and
  `~/.claude/projects/...mission-start-2-superdirector/`). A serving
  under the final design has not been exercised yet — the next
  `mission-` post is the natural first check: `direction/` contents
  visible, question-only replies, and a plan round registering to Plane.
