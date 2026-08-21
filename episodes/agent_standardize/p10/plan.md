# agent_standardize p10 plan — the entrance answers about the work

Goal: an instance's own channel stops being a placeholder. Asked there, the
agent surveys its plans and their execution stage by reading Zulip, explains
them, and — when told to — resolves finished topics and marks the Plane side
Done. First driven by the Omni Agent, then by agfront through nothing but the
introductions.

Success criteria:

1. **forge**: a question in `agforge-agstudio1` ("list your plans and where
   each stands") is answered from a live scan of its `assetplan-`/`assetrun-`
   topics; "resolve the finished ones" verifies and resolves them.
2. **autolab**: the same in `autolab-agstudio1`, scanning `pj-` channels,
   their `workplan-` topics and the derived `work-` channels; the done pass
   also closes a mission Work whose Sub-Works are all completed — the gap p9
   found (S2-30 `unstarted`) closes here.
3. **agfront can do it**: the developer asks Front for a cross-agent status
   check and a cleanup; Front reaches both entrances knowing only the
   introductions. Attributability grep as always: no entrance knowledge in
   `agfront/src` or its guide.
4. The entrance guides are terse — same register as forge's existing ones
   (tens of lines, not hundreds).

Decisions already made ([brandump.md](brandump.md) + discussion):

- **No new role.** The entrance run is the existing `roles.front` of each
  agent (the in-agent conversational subagent — unrelated to agfront).
- **Every role runs on `sonnet` this phase, the front roles included.** The
  `local` (ollama) profile has been a source of trouble and has raised cost
  rather than lowered it. autolab's `roles.front` is the one committed
  `local` role left — flip it to `sonnet` (and drop its `nested_harness`
  requirement, which belonged to the in-process backend). Then check all
  three `.local/agents.local.toml` overrides for anything still steering a
  role to `local` and remove it. The `local` profile definition itself may
  stay in the files — unused, not deleted; Agent ≠ Model, and every run
  records its backend anyway.
- Reading is Zulip-only: the Work data is mirrored into the chat and the
  execution state exists nowhere else. Plane is touched only to mark Done.
- The `workrun-`→plan binding the entrance reads is the **channel
  description** — the human-readable copy p9 deliberately kept. Selfnotes
  stay hidden; the entrance never needs `read --all`.
- Execution is not under test. The seeded fixtures are trivial (a work that
  writes `text.md`, or one image plan) — or p9's own residue where it
  already shows the un-resolved/un-Done state.
- Resolving is done when asked, not proactively. That is contract, not
  shackle: the entrance answers questions and follows instructions; it does
  not tidy on its own.

Constraints: secrets in `.local/`; pyagag → push → re-lock consumers; push
both agents; every entrance question is one paid run now (was a free canned
line) — fine, do not economize.

## Step 1 — agentchat: `channels` and `resolve`

- `agentchat channels [--prefix <p>]` — thin wrapper over
  `client.channels()`, name and description per line (the description is the
  binding autolab's entrance reads).
- `agentchat resolve <channel> <topic>` — wrapper over `resolve_topic`.
  Verify early that one bot may resolve a topic another bot opened (p1
  precedent: archiving needed the creator or an admin; resolving is a topic
  rename and should be lighter — confirm, and if a right is missing, decide
  owner-resolves-on-request instead of escalating credentials).
- `--help` grows the two entries; it is still the only manual.
- Tests, push, re-lock agforge/agautolab/agfront.

## Step 2 — autolab: the mission-done command

A small deterministic CLI (e.g. `python -m agautolab.mission_done`): find
mission Works whose Sub-Works are all completed and mark them Done —
or one issue id given explicitly. Reuses the existing Plane modules; prints
one line per transition. This is the entrance's "Plane operation is Done-ing
only" tool, and it closes p9's open item by construction.

## Step 3 — the entrance runs on `roles.front`

Both agents, same change:

- The model unification first: `agautolab/agents.toml` `roles.front` →
  `sonnet`, the `.local` override sweep on all three agents, and a listener
  restart — so every entrance answer in Step 4 is already a `sonnet` run.
- Replace the canned `entrance_reply()` with a normal `serve_topic` serving
  of `roles.front`, guide from a new folder (naming per the existing
  convention, e.g. `entrance_front/guide.md`). Own-channel sweep, home,
  callbacks — all already there; the entrance topic is just an owned topic.
- **forge guide** (aim ≤ 10 lines): questions about this instance's work;
  `agentchat topics <own channel>` shows plans (`assetplan-`) and runs
  (`assetrun-`); a `✔` name is finished; `read` for detail; look up only
  what was asked; if asked to resolve finished ones, read to verify, then
  `agentchat resolve`.
- **autolab guide** (aim ≤ 12 lines): `agentchat channels --prefix pj-` are
  the projects, `workplan-` their missions; `channels --prefix work-` the
  execution channels, each description naming its project and mission;
  `workrun-task*` with `✔` are finished tasks; if asked to close out, verify,
  `agentchat resolve` the finished topics, and run the mission-done command.
- **Intros**: one line each — ask about my work in my channel. Re-post both.
  This line is what lets Front find the entrances in Step 4 with no code.
- Tests, push, kickstart both listeners.

## Step 4 — Prove and report

1. Seed: one trivial forge plan (single image) and one trivial autolab
   mission (a task writing `text.md`), run to completion but left
   un-resolved/un-Done — or reuse p9 residue if it still shows the state.
2. **Omni pass**: ask each entrance for the survey; check the answer against
   the real topic lists; then "resolve the finished ones" and verify `✔` and
   Plane Done (mission Work included).
3. **Front pass**: fresh `front-*` topic — "check both agents' work status
   and clean up finished items"; permit; Front asks both entrances via
   `agentchat`, reports, resolves are executed by the entrances on Front's
   instruction. Zero Omni posts after the permission.
4. Evidence: message trails, before/after topic lists, Plane states, the
   attributability grep, run costs per entrance question.
5. `report.md` deferred candidates: proactive tidying (cron-like sweeps),
   entrance answering for humans vs agents differently, scan cost at tens of
   projects, `brandump.md` filename.
