# p3 step 1 — agforge vocabulary

`create-` → `assetplan-`, `runcreate-` → `assetrun-` across agforge. Code
only; the live realm still speaks the old words until the Step 3 cutover, so
the intro is **not** re-posted yet.

Commit: `agforge 2469d69`, pushed to GitHub. Nothing reloaded.

## What moved

Renamed on disk (`git mv`, so history follows):

| old | new |
|---|---|
| `src/agforge/create_topic.py` | `src/agforge/assetplan_topic.py` |
| `src/agforge/runcreate_topic.py` | `src/agforge/assetrun_topic.py` |
| `tests/test_create_topic.py` | `tests/test_assetplan_topic.py` |
| `tests/test_runcreate_topic.py` | `tests/test_assetrun_topic.py` |
| `agent/guides/create_front/` | `agent/guides/assetplan_front/` |
| `agent/guides/create_generator/` | `agent/guides/assetplan_generator/` |
| `agent/guides/runcreate_generator/` | `agent/guides/assetrun_generator/` |

The plan left the guide-folder rename to the implementer. Taken: leaving
`runcreate_generator` beside a module called `assetrun_topic` would have kept
the retired word alive as the name of the thing the new word describes. The
guides' *contents* needed no edit — they say "create" only as a verb, which
confirms the plan's survey that guides carry no prefix vocabulary.

Renamed in place:

- `zulip_listener.py`: `REQUEST_TOPIC_PREFIX` → `ASSETPLAN_TOPIC_PREFIX`
  (`"assetplan-"`), `RUNCREATE_TOPIC_PREFIX` → `ASSETRUN_TOPIC_PREFIX`
  (`"assetrun-"`), the `SWEEP_PREFIXES` tuple, `entrance_reply()`'s
  directions, and the module docstring.
- `assetrun_topic.py`: `handle_runcreate` → `handle_assetrun`,
  `RUNCREATE_TIMEOUT_SECONDS` → `ASSETRUN_TIMEOUT_SECONDS`, the record
  directory `.local/agent/runcreate` → `.local/agent/assetrun`.
- Docstrings and comments in `works.py`, `plane.py`, `toolsets.py`,
  `generate.py`, `comfy_video.py`, `assetplan_topic.py`.
- `params/intro.md` — the cross-agent contract text now says
  "open an `assetplan-…` topic". Committed, not posted.
- `README_DEV.md` — prefix mentions and "the whole create flow".
- ~50 test references.

## The comment that died

`dispatch()` carried a note explaining that `runcreate-` is matched before
`create-` on purpose, because one prefix was a superstring of the other.
`assetrun-` and `assetplan-` share no stem, so the ordering is now free and
the note is gone — replaced by one line saying exactly that. This was
predicted in the plan; it is the one behavioural comment the rename retires.

## Evidence

- `uv run pytest -q` → **189 passed** (deterministic suite, unchanged count).
- Criterion-3 grep over agforge's `src params agent tests README_DEV.md
  service agents.toml`:

  ```
  $ grep -rn 'create-\|runcreate-\|mission-\|"run-' …
  tests/test_role_run.py:41:    record = tmp_path / "records" / "run-0001.json"
  tests/test_role_run.py:51:    assert written["request_id"] == "run-0001"
  ```

  Both are the `ag.agent-run.v1` record filename, not a topic prefix — the
  same false positive the plan already excused in pyagag.

- pyagag re-verified cheaply, as the plan asked: its only `run-` hits are
  `topics.py`'s `run-{number:04d}.json` record numbering, and its only topic
  prefix constant is `RESOLVED_TOPIC_PREFIX = "✔ "`. **No pyagag change.**

## State after this step

agforge's code speaks the new vocabulary; the running listener does not — it
is still on `c135bc7` and will stay there until Step 3. agautolab still opens
`create-asset_<issue>` topics, which the new sweep no longer matches, so the
cross-repo contract is deliberately broken until both sides cut over
together (Step 2 then Step 3).
