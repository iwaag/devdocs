# Step 3 — the unresolved work list

`GET /work` returns every unresolved topic of every project channel, flat, one
row per topic:

```json
{ "channel": "work-m-44", "topic": "workrun-task1-…",
  "project": "pj-mediagen", "stream_id": 106 }
```

## Which channels are swept

The enumeration did not exist anywhere — the plan was right about that; only
the create/archive side had the naming rules. It is built from one `GET
/streams`:

- every channel named `pj-<slug>` is a project;
- every channel named `work-<label>` **filed in the same channel folder** as a
  project channel belongs to that project.

The channel folder is the link, not the name. `work-m-44` says nothing about
`pj-mediagen`; its folder does, because a project channel files itself and its
`work-` channels inherit the folder (`README_DEV.md`, 2026-09-04). That is why
a project channel filed wrongly by hand used to drag its whole fleet with it —
and it is exactly the relation this route reads.

Measured on the realm today: 4 project channels, 37 `work-` channels, **0
orphans** — every `work-` channel resolves to a project. A `work-` channel in
no project's folder is simply not swept, rather than being guessed at.

## Unresolved is Zulip's rule, not ours

`agag.zulip.RESOLVED_TOPIC_PREFIX` (`"✔ "`) is imported. There is no second
rule, no per-agent special case, and nothing is inferred from a topic's name.
This is the half of the braindump's worry that does not apply: `workplan-` /
`workrun-` / `assetplan-` mean different things to different agents, but
*resolved* means one thing to Zulip.

Interpreting those prefixes is deliberately not done. The payload carries the
raw topic name and the frontend shows it.

## Live result

```
134 topics across 41 channels — 76 resolved, 58 unresolved

pj-ghtrends    5
pj-mediagen   27
pj-papers      8
pj-studyarxiv 18
```

`/work` returns those same 58, with no `✔ ` name among them. Their name
prefixes are `workplan-` (41) and `workrun-` (17) — a single agent's vocabulary
today, which is precisely why the route does not build anything on it.

## Cost and failure

A full sweep is `1 + 41` Zulip calls; the 30 s cache is what stops a browser
reload from paying twice. One unreadable channel is collected into an `errors`
array and the other 40 still render — the view should be able to say "this one
channel could not be read", never quietly show a shorter list. `errors` was
empty on every run today.
