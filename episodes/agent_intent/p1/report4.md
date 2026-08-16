# agent_intent p1 — Step 4 report: the liveness status file

AI-generated (Omni Agent, 2026-08-17).

## The file

`<workspace>/.local/agag-status.json`, written by `agag.status` (pyagag) after
every Zulip poll that actually returned:

```json
{"schema": "agag.status.v1",
 "last_poll_ok": "2026-08-16T18:30:55.410993+00:00",
 "queue_id": "60dabf65-e02f-4b3d-b3ff-7440e759b7f0",
 "last_error": null}
```

Three properties, each deliberate:

- **Written only on success.** A poll that timed out, hit a dead queue, or
  failed calls `record_error` — which *remembers* the reason and writes
  nothing. The reason is reported as `last_error` on the next real success and
  then cleared. So a failing listener cannot refresh its own liveness, and
  staleness is the *absence* of a write, which no bug in the writer can fake.
  This is the plan's anti-lying rule, and it is what makes the signal worth
  reading at all.
- **Written atomically.** Temp file in the same directory, `fsync`, then
  `os.replace`. A reader never sees half a document, and a crash mid-write
  cannot destroy the last honest one.
- **Never fatal.** A listener that cannot write its status keeps listening; the
  failure is logged once, not once per poll. An observability file that can
  take down the thing it observes is worse than no file.

**On by default.** With no configuration the path is
`<cwd>/.local/agag-status.json`, and every listener's launchd
`WorkingDirectory` is already its checkout. An agag listener therefore becomes
observable by following the existing convention rather than by being
configured — which is the braindump's point: conform to agag and you join the
system. `AGAG_STATUS_PATH` relocates it; an empty value turns it off.

Both loops carry it: `serve` (DMs) and `sweep_serve` (channel topics).

## The pickup

`nodeutils_collect.observe_workspace` reads the file when a probed workspace
has one, and attaches an `agent_status` entry to that workspace's existing
observation — so the established ingest path carries it with no new transport,
exactly as the plan asked.

`age_seconds` is computed **on the node**, against the same clock the listener
wrote with, so node-vs-Nautobot clock skew cancels instead of being baked into
a comparison. Absent, oversized, unparsable, and foreign-schema files all read
as "no liveness claim" — never as an error, and never as liveness.

## Live evidence

agforge, on agstudio, end to end:

1. pyagag `cb3e4d2` pushed; agforge's lock bumped (`b7bc722`); listener
   restarted with `launchctl kickstart -k`.
2. The file appeared within seconds of the first successful poll.
3. `nctl reconcile agstudio --refresh-observation --yes` collected and ingested.
4. The agforge workspace observation in Nautobot now reads:

```json
"agent_status": {
  "present": true, "readable": true,
  "last_poll_ok": "2026-08-16T18:30:55.410993+00:00",
  "queue_id": "60dabf65-...", "last_error": null,
  "age_seconds": 33.835, "checked_at": "2026-08-16T18:31:28+00:00"
}
```

A new `DesiredWorkspace` (`agforge` → `agstudio`) was declared first, because
nodeutils never guesses a workspace path — it probes only declared ones — and
`DesiredAgent agforge` now points at it.

| gate | result |
|---|---|
| pyagag (`uv run pytest -q`) | 206 passed (10 status tests + 2 listener-loop tests) |
| nodeutils (`uv run pytest -q`) | 95 passed, 2 subtests (4 new) |

The listener tests are the ones that matter: a scripted poll sequence of
`[] → timeout → queue-expired → events` produces exactly
`ok, error, error, ok`. An empty-but-returned poll counts as alive; a poll that
never returned does not.

## Honest limitation

agforge runs **two** poll loops in one process — a DM `serve` thread and the
main `sweep_serve` — and both write the same workspace file. Atomic writes make
that safe, and the observed `queue_id` simply alternates between them. But it
means the signal is *workspace* liveness, not *per-loop* liveness: if the DM
thread died while the sweep kept polling, the file would still look fresh.

Left as is for p1. Splitting the file per loop would add structure before there
is any evidence that a half-dead listener is a real failure mode here — and the
plan's own instinct (resist promoting things until false-positive data exists)
cuts the same way.

## Deployment

pyagag `cb3e4d2`, nodeutils `d68b963`, agforge `b7bc722`, superproject pinned
to the new nodeutils (`4a508a1`) — the pin is what the observation deploys to
a node, so pushing nodeutils alone would have collected with the old collector.

Only agforge is enrolled so far. The other listeners (cagent, the two autolab
nodes, the devworld assistant) get the same file for free once their pyagag
pins are bumped and they restart; agautolab1 additionally needs an Ansible run,
which is an approval boundary and is queued for the developer.

Bumped a pyagag dependency and restarted a listener that in-system agents
maintain — handoff candidate.
