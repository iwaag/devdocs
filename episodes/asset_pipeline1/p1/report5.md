# Report 5 — agautolab: asset state machine on run- topics

Status: **done**. `agautolab` suite green (103 passed, was 88).

A `run-` trigger that picks an `asset`-labelled Work no longer runs coding
blindly. It asks Plane where the asset stands and takes one of three routes.

## `next_work()` returns a `Work`

The five-tuple became `mission.Work(slug, name, description, project_id,
issue_id, is_asset)`. `is_asset` is read off the issue's labels using the same
id-set lookup `eligible_works` uses — and it has to come from Plane, because
Step 2 deletes the `task[N].md` that carried the `[Asset]` marker the moment
Plane accepts it. The label is the only surviving record.

A project with no `asset` label at all yields `is_asset=False` for everything;
a missing label id is never mistaken for a match (pinned by a test).

## The ledger is agforge's Work, not a local file

```python
asset_topic(issue_id)            -> "create-asset_<issue_id>"
asset_order_key(channel, id)     -> "pj-<slug>/create-asset_<id>"
asset_order(project_id, ch, id)  -> ("absent" | "working" | "done", issue)
```

`asset_order` looks the key up under external source `agforge` in the same
Plane project — agforge's `resolve_project` routes a `pj-<name>` channel to
the Plane project of that name, which is the autolab project — and reads the
state group off the **list row**, because the external-key endpoint may answer
with a thin object. Same rule `write_mission_workspace` already follows.

There is no local marker file anywhere in this. A wiped `.local/` cannot make
autolab order the same asset twice.

### Naming

`create-asset_<work_id>`, underscore before the id. agforge's listener matches
only the `create-` prefix, so the rest of the name is free, and the underscore
keeps it clear of agforge's own `create-YYYYMMDD-HHMMSS-<id>` hyphenation.

One topic per asset work is load-bearing, not cosmetic: agforge keys **one
Plane Work per `<channel>/<topic>`**, so a reused topic would overwrite the
ledger entry and mix two specs into one chatlog.

## The three states

**1. absent** → post the order into `pj-<slug>/create-asset_<id>`, report
`asset ordered in …`, finish the serving.

The order is `# <task title>` + the task description + the project's
`direction/aesthetics.md` under an "Art direction for this project:" heading.
Written to stand alone on purpose: agforge's front reads the whole topic
chatlog and its generator plans from that, so nothing in the order may point
at a file agforge cannot open. A project with no `aesthetics.md` yet produces
a shorter but still self-contained order.

Posting it also arms agforge: its sweep matches a `create-` topic whose last
poster is not the forge bot.

**2. working** → report `asset in progress in …`, do nothing else, finish.
**No `runcreate-` post is ever emitted** — the Omni Agent fires that by hand
this episode. A test asserts the string `runcreate` never appears in anything
the handler does.

**3. done** → recover the key, re-sign, run coding.

The key comes from the newest `[S3KEY] <key>` line in the asset topic — the
footer Step 4 puts on every delivery. Reading it out of Zulip needs no new
Plane API and no shared code between the two agents; only the marker is
shared, and it is documented on both sides. A completed agforge Work whose
topic carries no footer raises rather than guessing, and the topic gets
`failed during asset check: …`.

`resign(key)` then calls `POST http://localhost:8092/api/resign`
(`AGFORGE_URL`-overridable). **The ordering is the whole point**: the re-sign
happens immediately before `run_work`, so a 1200 s coding run cannot outlive
the 60-minute URL it was handed. A URL recovered any earlier could die
mid-run against an unknown remaining TTL.

The coding prompt is `work_prompt(url)` — the run guide, then:

```
Note: the asset required by this work can be downloaded from the URL below.
If the asset does not match the spec, try to compromise; only if truly
unacceptable, treat the work as failed:
<url>
```

The director check that would normally weigh the delivered asset against the
spec is deliberately skipped this episode, so that judgement is handed to the
coding run itself, in the prompt.

## What deliberately does not happen

The autolab Work stays `unstarted` through states 1 and 2 — `report_work` is
not called, no state transition is made. That is what makes `next_work()` keep
choosing it, and it is also what blocks every other Work behind it until the
asset lands. Accepted for now, per the plan.

## Tests (103 passed, +15)

`tests/test_mission.py` — `next_work` returns a `Work`; the asset label is
read off the issue; a project without the label yields plain works; the topic
and key naming; and `asset_order`'s three states, including the thin-object
case where the list row is what decides.

`tests/test_zulip_listener.py` — the whole gate through `handle_run`:

- unordered → the order text is exactly title + description + art direction,
  posted to `create-asset_i9`, no coding run, no Plane write
- unordered with no `aesthetics.md` → still self-contained
- in progress → reports, runs nothing, emits no `runcreate`
- completed → call order is `order-state → find-key → resign → run`, and the
  fresh URL is the one the run receives
- completed with no `[S3KEY]` footer → `failed during asset check`
- a plain Work never touches the asset route at all
- `work_prompt` with and without a URL
- `find_asset_key` takes the **newest** footer, and answers `None` when a
  message merely mentions the marker in prose

## Noted

The dirty-workspace check still runs before the asset gate, so a leftover
`.local/work/` from a crashed run blocks even the ordering of an asset, which
does not touch that directory. Left as is: a dirty workspace already means
"stop and look", and that discipline predates this step.
