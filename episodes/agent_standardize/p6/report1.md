# agent_standardize p6 — Step 1 report: the harvest becomes shared machinery

AI-generated (Omni Agent, 2026-08-21).

## What moved

`agfront/src/agfront/agents_md.py` is gone. `harvest_intros`,
`render_agents_md`, `write_agents_md`, `agents_file_path` and the four
constants around them now live in `agag.intro`, beside `post_intro` — pyagag
`d3bd27a`.

They landed in `intro.py` rather than a new module on purpose. `post_intro`
writes the latest post of an `intro-<instance>` topic; the harvest reads the
latest post of every `intro-*` topic. Those are the two ends of one contract,
they already shared `AGENTS_CHANNEL` and `INTRO_TOPIC_PREFIX` as duplicated
literals in two repositories, and after the move each constant is declared
once. The module docstring now says so, so the next reader does not have to
rediscover why a "posting" module also reads.

The shape is unchanged, deliberately — this step was a lift, not a redesign:

- the latest post per live `intro-*` topic in `#agents`, bodies verbatim;
- resolved (`✔ `) and non-`intro-` topics skipped;
- entries sorted by agent name, so two harvests of one board are the same file;
- a `Generated: <iso8601>` line;
- an empty board renders the honest "no agents known" sentence instead of
  failing the run.

One rename: `write_agents_md(client, front_dir)` → `write_agents_md(client,
workspace)`. The parameter is now any run's workspace, because in Step 3 it
will be a `workplan-` or `workrun-` workspace and not Front's.

## Tests moved with it

`agfront/tests/test_agents_md.py` → `pyagag/tests/test_intro_harvest.py`,
assertions unchanged except the import and one docstring sentence that said
"agfront's own" and now says "the consuming agent's own". Twelve tests: the
five harvest facts, the three rendering facts, the two placement facts, and
the guard client that raises if the harvest ever tries to change a
subscription. pyagag is green at 285.

## agfront switched in the same change

No parallel copy and no compatibility shim, per the plan. `zulip_listener.py`
imports `write_agents_md` from `agag.intro`; `tests/test_zulip_listener.py`
imports `agag.intro` where it used `agfront.agents_md` for `AGENTS_CHANNEL`
and `NO_AGENTS`. A grep for `agents_md` in `agfront/src` now returns only the
import line and the call site. agfront is green at 21.

Front's behavior is byte-identical: same file, same place, same content.

## Commits and locks

| repo | commit | what |
|---|---|---|
| pyagag | `d3bd27a` | Lift the intro harvest into `agag.intro` |
| agfront | `2d29a3e` | Read the intro board through pyagag (lock → `d3bd27a`) |
| agautolab | `0f0df83` | Bump pyagag: the intro board harvest is shared now |

All three pushed to GitHub. agautolab's lock is bumped now, ahead of Step 3
needing it, so the harvest is already importable when the run-setup change
lands; its 178 tests pass on the new pin unchanged.

agforge is **not** re-locked. Criterion 4 freezes its code, and it has no use
for the harvest — it is asked, it does not ask. It stays on its current pin
until something it needs moves.

## Deployment note

Nothing is deployed by this step. The agfront and agautolab listeners run
from working trees on this Mac against their `.venv`s, which `uv sync` has
already updated, so a `launchctl kickstart` is enough whenever the next
behavior change makes one worth doing — Step 3 will do it. agautolab1 runs
the gateway only and holds no listener, so it is unaffected until the Step 3
push.
