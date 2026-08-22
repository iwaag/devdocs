# agag_builder p2 — report

Goal: agautolab and agfront get their five common modules from `agag.agent`
/ `agag.entrance`, as agforge did in p1, with nothing about what they do
changed. **Done**; all four success criteria met. Steps:
[report1](report1.md) Front's name, [report2](report2.md) autolab,
[report3](report3.md) agfront, [report4](report4.md) pins,
[report5](report5.md) live check.

## Commits

| repo | commits |
|---|---|
| pyagag | `5da31c9` `AgentSpec.extra_prefixes`, `run_role(extra_meta=)`; `a8fa481` `run_role(agent=)` |
| agautolab | `96f09ff` on the skeleton, `dbdbced` README |
| agfront | `fbab21a` instance.example.toml, `ef70baf` on the skeleton |
| agforge | `6a5b931` pin only |
| pj-agdev | `5a8313d` three submodule pointers, one commit |

All three locks: `pyagag…#a8fa481529d16c56ddf8716e50c3cf6d479a9fad`.

## Line counts — the five concerns per agent

| agent | before | after | of which the agent's own logic |
|---|---|---|---|
| agforge (p1) | 521 | 297 | `tool_environment` + wrappers kept for old tests (128) |
| agautolab | 1554 (45+34+25+196+1254) | 1332 (47+14+118+38+1115) | `zulip_listener.py` mission code 1115, `role_run` budget/bypass/workspace pin |
| agfront | 443 (133+310) | 305 (26+8+29+242) | `serve`/`handle_mention`/`front_prompt` 242 |

Diff stats: agautolab 18 files, +442/−682; agfront 13 files, +159/−365.

## Tests

autolab 179 → 168, agfront 24 → 20, forge 197 (unchanged), pyagag 401
(+2). Every deleted test tested a removed copy of skeleton code
(`instance_name` wrapper, `tool_environment`, `ROLE_ALLOWED_TOOLS` tables,
`topic_filter`, `entrance_prompt`/`serve_entrance`, `main`'s dispatch);
pyagag tests the originals. Rewritten ones pin what is now each agent's own:
the routes handed to `listener_main`, the handler-side guards, the v2 grants.
Details per agent in report2/report3.

## What the listener swap said (skeleton vs mission)

1. **Own channel vs prefix precedence.** Skeleton: prefix route first
   (forge's requests live in its own channel). autolab: everything in its
   own channel is a question. Resolved on autolab's side
   (`at_the_entrance` in its handlers), not by a skeleton flag. If a third
   agent wants autolab's rule, that is the moment for an `AgentSpec` field.
2. Entrance timeout 900 → 600 s for autolab; the entrance chatlog now drops
   acks (the skeleton's `drop=is_ack`); a passive DM thread autolab never
   had. None affected the live check.
3. `write_run_record` never wrote autolab's `project` key into the file;
   only the returned dict carried it. Preserved as-is via `extra_meta`.

## Live check (report5)

`pj-runsmoke1/workplan-p2-skeleton-file` → R-6/R-7 → `work-r-6/workrun-task1-r-6`
→ `main/skeleton.md` committed `e04a930`, R-7 Done, topic resolved, devlog
pushed. Entrance `p2-list-plans` answered in autolab's vocabulary. Front →
agecho → mention → report at home, callback marked served. No topic served
twice.

## What still differs between the three listener entries

`agautolab/listener.py` and `agfront/listener.py` differ only in the
docstring, the imported prefixes/handlers, the `dispatch` dict, and
`on_mention`. agforge's `zulip_listener.py` (p1) is the same call plus a
`dm_handler` and two wrappers (`topic_filter`, `dispatch`) kept for its old
tests — the pattern this phase avoided; deleting those is a 20-line forge
cleanup. So: **the skeleton is done** — `agag init`'s `listener.py` is the
actual shape of every agent, and what an agent is is its `SPEC`, its
`agents.toml` grants, its guides, and its handlers.

## Human-side notes / left open

- Front's `params/intro.md` exists but was not posted to `#agents`
  (report1): whether the Developer's agent should be introduced to other
  agents is a decision, not a default.
- Front's env vars are now `FRONT_INSTANCE_NAME` / `FRONT_ZULIP_LOG_ONLY`.
- autolab's Plane lookup is still `ROOT.parent/.local/plane-credentials.env`
  (report2): switching to `credentials_path(root)` needs a symlink in each
  autolab checkout's `.local/`, the agautolab1 VM's included.
- `R-6` in Plane and `#work-r-6` are live-check residue, finished and
  resolved; delete at leisure. agecho's nohup listener (PID 65851) is still
  the only thing keeping agecho reachable.
- standardize p10 TODOs seen on the way, not pruned: forge's test-only
  wrappers above; `agforge` F2-22 unrun plan.

## Next phase seed

autolab invoking `agag init` on request needs only the `--yes` path and the
printed checklist from p1; the generated project now matches the three
live agents exactly, so there is no second shape to reconcile.
