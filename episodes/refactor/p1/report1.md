# refactor p1 step 1 — Zulip is enough to execute autolab work

## What the step asked

Make the Zulip conversation sufficient to plan, execute and report one
autolab mission: store the plan and each task's executable description where
the work happens, replace Plane-backed lookup, read-back, result recording and
mission completion, initialise a project without Plane, keep the Git
workspace behaviour, and recover pending work from Zulip after a restart.

## What changed

### The record is the conversation (`agautolab/src/agautolab/worklog.py`, new)

`mission.py` — the Plane mirror of one chat topic — is deleted. In its place
`worklog.py` reads and writes the same model out of Zulip:

| was | is |
|---|---|
| a Work keyed `<channel>/<topic>` | the `workplan-` topic itself |
| a Sub-Work keyed `<channel>/<topic>#<N>` | the `workrun-` topic itself |
| `description_html` on an issue | a visible post, Markdown, in that topic |
| a Plane state group | a `[selfnote][state]` note |
| an issue comment | a visible `## Result` post where the task ran |
| the `AUTO` label + `[AUTO]` project marker as an execution filter | nothing — they filtered a queue that had no caller left |

Prose is what a human reads, so the plan and the task descriptions are
ordinary posts. Everything a program must answer without guessing is a
selfnote (`agautolab/src/agautolab/anchor.py`) — machine-to-machine, hidden
from every chatlog, and never counted as somebody speaking, so writing one
never buys a run:

```
[selfnote][mission] <project slug>       in a workplan- topic
[selfnote][task] <mission id>#<serial>   in a workrun- topic
[selfnote][doc] <message id>             which post is the current document
[selfnote][state] <word>                 where the work has got to
```

Two rules, and the difference between them is the model: **identity is
written once**, so the earliest note of its kind wins; **state and the current
document change**, so the newest wins.

### Identity is a message id, not a name

The `[mission]` note's **own message id** is the mission; the `[task]` note's
own id is the task. `ZulipClient.message` (new, pyagag) answers with the
conversation an anchor is in *now* — through a resolve rename, a hand rename,
or a move to another channel — and answers `None` for a deleted message.

That is the separation step 2 asks for, arriving one step early because
nothing else could carry the record: a topic of a reused name is different
work, and a deleted origin is **absent** rather than whatever took its name.
Names follow from the id — `work-m5512`, `workrun-task3-m5512` — so no second
mission can want a channel name, and the old Plane-label names (`work-pa-12`)
are gone.

### The four task states, kept apart on purpose

`open` → `completed` → `accepted`, with `cancelled` to one side.

- `completed` — the run and the developer agreed the work is done. It is what
  the gate in front of the next task waits for.
- `accepted` — a human accepting the request; step 3's business.
- Zulip's `✔ ` is neither: it closes the *conversation*.

A mission is `planned`, `started`, `cancelled` or `done`.

### Project initialization without Plane

`project_init.py` no longer creates or finds a Plane project: `PLANE_ENV`,
`load_plane_config`, `ensure_plane_project`, `plane_identifier`,
`plane_project_name`, `auto_description`, `project_slug` and `PlaneConfig` are
deleted. A pattern-managed workspace is now a genuine no-op rather than "the
Plane project is the one thing ensured". `project_archive.py` loses
`archive_plane_project` for the same reason. Gitea, the clones and the
`[AUTO]` commit marker are untouched.

### Mission completion (`mission_done.py`)

Rewritten to count conversations. It sweeps `pj-` channels for `workplan-`
topics holding a mission note, or resolves one named mission through its
anchor, and writes `[selfnote][state] done`. It was described as "the only
Plane operation the entrance performs"; the entrance now performs none.

### Shared changes (pyagag), kept to what this path required

- `agag/document.py` (new) — a Markdown file's first heading is its title.
  The rule lived in `agag.plane` because the only storage of a document was a
  Plane issue; `agag.plane` re-exports it, so both storages agree.
- `ZulipClient.message(id)` — the anchor read described above.
- `agag/__init__.py` no longer re-exports `PlaneConfig`/`PlaneError`. That
  re-export made `import agag.selfnote` load the Plane HTTP client, so no
  agent could honestly claim not to reach Plane. Nothing outside `agag.plane`
  and its own tests used either name.

## Evidence

**Tests.** pyagag 542 passed; agautolab 224 passed.

```
$ cd pyagag && uv run pytest -q
542 passed in 68.94s

$ cd pj-agdev/agautolab && uv run pytest -q
224 passed in 0.94s
```

`tests/realm.py` is a Zulip realm small enough to hold in a test and honest
about the three behaviours the record depends on: resolving renames a topic,
a message id survives every rename, and a deleted message is gone.
`tests/test_worklog.py` (24 tests) exercises the model against it —
re-planning keeps the mission it already is, an anchor is followed through a
`✔ ` rename, a deleted anchor is absent, a `workrun-` topic carrying another
mission's note is filtered out of this mission's tasks, a completed task
stays completed across a re-plan, and the gate holds task 2 until task 1 is
finished or accepted.

**Plane unavailability, as an import-graph property**
(`tests/test_plane_independence.py`). The live half of the plan's
verification is step 4; the half a test can hold is that autolab's own path
cannot call Plane because it never imports it. Every module of the agent is
checked for names reaching `agag.plane`, and a subprocess imports the whole
package and asserts `agag.plane` is not in `sys.modules`.

## Restart recovery

Nothing new was needed and nothing was added. The sweep already recovers
pending work by reading the realm, and with the record moved into the realm
there is no second copy to lose: `.local/projects/` is re-cloned by
`init_project`, `.local/topics/<channel>/<topic>/<N>/` is per-serving
evidence, and `.local/agent/` holds run records. No local file is consulted to
decide what a mission is, which task a topic runs, or what state either is in.
This is asserted only by inspection at this step; step 2 tests restart
recovery against a replacement, where it can fail in an interesting way.

## Limitations carried into the next steps

- **Step 2's replacement operations are not built.** Identity is now separable
  from the display name, which is the precondition; retiring a conversation
  and opening its replacement is not yet a workflow the agent can run.
- **Step 3 is untouched.** `agdevworld/agentroom`'s `closing.py`/`close.py`
  still discover autolab work through `PlaneBoard`/`PlaneReader`, so the
  operation room will read new autolab missions as having no Work. `accepted`
  exists as a state with nothing yet writing it.
- **Nothing has been deployed or run live.** No listener has been restarted on
  this code and no real mission has been planned through it.
- **This is not Plane removal.** `agag.plane` stays for forge and cagent;
  `nctl`'s agent registration and Nautobot's Plane identities are out of
  scope, as the plan says. Existing Plane issues are not migrated, and old
  `work-<plane label>` channels and their topics are not readable by the new
  code — a deliberate break, permitted by the plan's "old work records and
  formats need no compatibility support".
