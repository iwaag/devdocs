# agent_standardize p5 plan — agfront supervises the three workruns

Goal: Front triggers, supervises, and approves the three `workrun-` topics
left planned by p4, until all three are `✔` resolved. The point under test
is **long-running supervision** — can Front stay on a task that outlives a
single quick exchange.

The contract is already there (verified in `serve_run` /
`run_supercoder`): a non-bot post triggers the run, progress and questions
arrive as posts in the topic, and autolab closes out (report, Plane Done,
`✔`) only after the supervisor **posts agreement that the task is done**.
Front's "completion approval" is that agreement message — no autolab change
needed.

Success: all three `workrun-` topics resolved, close-outs confirmed
(Plane Done + devlog lines in the topic), driven by Front with the
developer only permitting in the `front-*` topic.

## Step 1 — `agentchat wait` (the p2 deferred item, now due)

Add to pyagag: `agentchat wait <channel> <topic> [--since <message_id>]
[--timeout <s>]` — block until a message newer than `--since` (default: the
current last), print the new messages, distinct exit code on timeout so a
script or agent can loop. Long-poll or modest-interval poll, implementer's
choice. Update `--help` (it is the doc). Push, re-lock agfront.

## Step 2 — Let a Front run live long enough

- Check Front's role timeout against a realistic supercoder leg and raise it
  if needed; a supervision run is legitimately long.
- If a run still can't span a whole task, the fallback shape is already
  legal: Front reports progress honestly into `front-*` and the developer's
  next post re-triggers a run that picks the supervision back up
  (`agentchat read --since` makes that resumable). Prefer trying the
  long-run path first; whatever happens is the measurement the braindump
  asks for — record it.
- Guide: add a sentence on supervision if the current one doesn't cover
  "keep waiting, answer questions, agree when genuinely done" (developer
  edits the guide, as before).

## Step 3 — Run it and report

1. Developer, in `front-*`: "supervise the three workruns to completion";
   permit when Front proposes. Sequential is fine (the previous-work gate
   may force it anyway).
2. Front per topic: trigger post → wait/read loop → answer questions →
   agreement post when done → confirm the `✔` rename.
3. `report.md`, short: the three topic links, resolution proof, wall-clock
   per task, and the honest account of how long-run supervision behaved
   (timeouts hit, retriggers needed) — that observation is the deliverable.
   Deferred: parallel supervision, unattended (no-permission) supervision.
