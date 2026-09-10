# p2 step 2 — forge's conversations are its work record

## What was two systems

An asset request lived in a Zulip `assetplan-` topic **and** in a Plane
issue. The issue's description was the plan, its last line was a
`[TOOLS] …` footer naming the toolsets, its comments were the results, and
its state group was "finished". The `assetrun-` topic pointed at it with
`[selfnote][work] <project id>/<issue id>`. Understanding one request meant
reading two systems and trusting they still agreed — the cost `refactor` p1
removed from autolab.

## What it is now

**A conversation is its own record.** `src/agforge/record.py` reads and
writes it; `src/agforge/anchor.py` is the note vocabulary.

| was | is |
|---|---|
| a Plane issue keyed `<channel>/<topic>` | the `assetplan-` topic itself |
| `description_html` | a visible Markdown post, named by `[selfnote][doc]` |
| the `[TOOLS] …` description footer | `[selfnote][tools] toolset-image, …` |
| `[selfnote][work] <project>/<issue>` | `[selfnote][assetrun] <request id>` |
| an issue comment carrying `[S3KEY]` | `[selfnote][result] <object key>`, in both conversations |
| a Plane state group | `[selfnote][state]`, newest wins |
| — | `[selfnote][replaces] <message id>` |

Prose is what a person reads, so the plan is an ordinary post. Everything a
program must answer without guessing is a selfnote: hidden from every
chatlog, and never counted as somebody speaking, so writing the record never
buys a run.

### Identity is a message id

The `[asset]` note's **own message id** is the request (`a5912`); the
`[assetrun]` note's own id is the run (`r5913`). `ZulipClient.message`
answers with the conversation an anchor is in *now* — through a resolve, a
hand rename, a move to another channel — and `None` for a deleted message.

Three things follow, and they are why the phase asked for this:

- **The run topic is `assetrun-<stem>-a<request id>`.** The stem is the
  requester's word and is reusable; the id is not. A re-plan keeps the same
  run topic; a replacement under the same stem gets its own and cannot merge
  into it.
- **The delivery follows the anchor home**, not the `[rootchat]` note's name.
  `origin_of` is the request's *current* conversation, so a result lands in a
  topic that has since been resolved or renamed — under the name it wears
  now, which is what keeps a post from opening a twin beside a ✔ topic.
- **A deleted origin is absent.** The run says "the request this topic runs
  (a5912) is gone" instead of delivering to whatever took the name.

### Retirement and replacement, one route

`uv run python -m agforge.retire <channel> <topic> [--replace]`. The request
is marked `retired` *before* anything moves, then both conversations are
renamed to `✔ retired-…-a<id>` — out of the prefix vocabulary the listener
sweeps **and** resolved — which releases the stem. `--replace` opens the
fresh request under the freed name with its own anchor and a `[replaces]`
note. The old-first order is not a preference: Zulip has one topic per name
in a channel, so a replacement created first would merge into what it
replaced.

A job still running is **not** interrupted. It is collected by the old run
topic, whose `[assetrun]` note names the old request, and delivered there —
late replies belong to the request that asked for them.

### The execution side

`workspace_dir` is keyed on the **run's anchor id** (`.local/agentws/r<id>/`)
rather than the Plane issue id, so a replacement has its own workspace and
cannot collect the old attempt's job or overwrite its outputs.
`prepare_workspace` gained one rule: while `watching.json` is there the run
is *collecting*, and `plan.md` and `tools/` are left exactly as the job was
submitted with them. Re-planning changes the request; it must not change an
attempt already queued in ComfyUI.

Asset bytes are untouched — still the object store, still one zip per
delivery, still `[S3KEY] <key>` on the delivery's last line. What changed is
that the key is also a `[selfnote][result]` note in both conversations, so
the durable reference survives the presigned URL by more than an hour.

### What was deleted

`src/agforge/plane.py` (project routing, the `FreeForge` fallback, the
`FORGEAUTO` label, the `[AUTO]` marker, `[TOOLS]` composition) and
`src/agforge/works.py` (`work_by_id`, `report_work`) — 367 lines, replaced by
`record.py`'s 430 and `anchor.py`'s notes. `.local/plane-credentials.env` is
gone from forge, `instance.example.toml` no longer mentions a Plane account,
and no guide ever did.

## Verification

**Plane unavailability is an import-graph property**, which is stronger than
a rejecting client — the code cannot call Plane because the module is never
loaded:

```
$ uv run python -c "import agforge.zulip_listener, agforge.assetplan_topic,
  agforge.assetrun_topic, agforge.record, agforge.cli, agforge.retire, sys;
  print([m for m in sys.modules if m.startswith('agag.plane')])"
[]
```

`tests/test_plane_independence.py` asserts the same in the suite, including a
subprocess that imports the package from scratch.

**The record is exercised against a realm, not a stub.** `tests/realm.py` is
a Zulip small enough to hold one request: posts have ids, `message(id)`
answers with the topic a post is in *now*, `rename_topic` moves a whole
conversation, and a post can be deleted. Anything less would let a test pass
on a record that only works while nothing is ever renamed.

`tests/test_record.py` (21 tests) pins: the anchor is written once; a request
is found through a rename; a deleted anchor is absent; the plan and toolsets
come back out of the conversation; `None` toolsets and `[]` are different
answers; an over-long plan is refused rather than silently truncated by
Zulip; a run is anchored to its request by id and follows it home; a fresh
attempt does not inherit an old verdict; a result is recorded in both
conversations and appends; retirement releases the stem; a replacement takes
the freed name and says what it replaced; and a late result belongs to the
retired request.

`tests/test_assetrun_topic.py` and `tests/test_assetplan_topic.py` were
rebuilt on the same realm and now cover the plan being recorded and read back
across a serving boundary, a re-plan reaching the next run, a collecting run
keeping its own plan and tools, success, failure, and a fresh attempt after a
failure.

```
$ uv run pytest -q        # agforge
215 passed in 6.6s
```

Three pre-existing failures surfaced when agforge's pinned `pyagag` was moved
from `0a33830` to `218cb31` (needed for `agag.document`): the shared entrance
now reads `whoami()` for the execution-option menu, and a run record carries
`started_at`/`ended_at`. Both fixtures were updated; neither is this phase's
change.

## Limitations left standing

- **Old forge work is unreadable, deliberately.** Existing Plane issues carry
  no `[asset]`/`[assetrun]` notes and nothing was migrated, as the plan
  permits. Any request in flight at deployment time is stranded.
- **The operation room still reads forge from Plane** (`closing.py`'s `[work]`
  note discovery). That is step 4, and until then a forge request previews
  there as it did before.
- **Retirement is a command, not a conversation.** forge does not retire a
  request because somebody asked it to in the topic; a person (or an agent
  with a shell) runs `agforge.retire`. The plan asked for one practical
  route, not a conversational one.
- **A channel accumulates retired topics.** `✔ retired-<name>-a<id>` beside
  the next one, distinct by anchor id; nothing prunes them. p1 recorded the
  same for autolab.
- The asynchronous path is carried unchanged here; hardening duplicate
  callbacks and restart recovery is step 3.
