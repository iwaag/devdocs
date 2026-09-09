# refine_routine p1 — step2 report: Front starts a run and resumes in the right conversation

Date: 2026-09-09

## Design

A run is a `routinerun-<id>` topic in the routine's own channel, and it is a
**conversation Front owns**, served like a `front-*` topic but by a new role,
`routine_run`, with its own guide and profile (`agfront/agents.toml`,
`agent/guides/routine_run/guide.md`).

- **Opening.** Asked to run a routine (Front Desk or any `front-*`
  conversation), Front reads the guide and posts **one opening post** with
  `agentchat send` into `#routine-<name>` › `routinerun-<id>`: the request
  in the developer's words, the execution and end conditions as Front read
  them, the guide post it read (message id), and the requesting
  conversation. `agentchat send` writes Front's root note into that topic,
  which is what records the origin — in a topic Front owns the note means
  "opened from", never "serve that instead", because an owned topic is
  always served as itself (the shared sweep already routes owned topics to
  the owner handler and never to the mention handler).
- **Starting.** The owner sweep serves a topic only when somebody else
  spoke last, so a topic Front opened alone would never start. The listener
  now starts it: right after the serving that opened it has replied
  (`start_opened_runs`, reached from both the topic and the callback
  handlers), and at listener startup for anything opened just before a
  crash (`recover_runs`). *Unstarted* is structural — only Front's speech
  in the topic and no serving ack — so a start happens once; progress
  entries follow an ack and never re-start anything, and a run opened by
  hand is served by the ordinary owner route.
- **Resuming.** Delegations made from a run serving are anchored to the run
  (the run is `AGENTCHAT_HOME`), so a delegate's answer naming Front
  resumes the run through the existing callback route, answers at home in
  the run topic, and is marked served there. The requester conversation
  hears nothing until the end. Recovery after a restart is the existing
  root-note and served-mark machinery; nothing new persists anywhere.
- **Finishing.** The run ends its entry with one fenced `ag-routinerun`
  block (`ag.routinerun-finish.v1`: `achieved`, `reason`, `report`). The
  listener validates it, posts the report into the origin conversation
  directly (no root note is written there), names the run, and resolves the
  run topic after the record. A broken block is recorded in the run topic
  as an error fence and the run stays open. A run with no origin reports in
  its own topic.
- **Sweep.** `SPEC.extra_prefixes = ("routinerun-",)`, so runs are swept
  wherever Front is subscribed — the four routine channels. Both routes use
  `handle_topic`; a run topic is served without the empty-topic guard.

## Focused tests (`agfront/tests/test_routine_run.py`, 77 passed in all)

- start: a run opened by a desk serving is served once, after the desk's
  reply, as its own conversation; a started run is not started again and
  appears as a thread of the desk; a run opened by a callback serving starts.
- resume: a delegate's answer serves the run (not the requester), replies
  at home, marks the callback served; two runs with their own delegate
  topics each get their own callback and nothing crosses.
- restart: `recover_runs` starts the unstarted run and skips the started
  one; a served mark keeps an answered callback from being served again
  (`sweep_rootchats` with and without the mark).
- finish: the report lands in the origin with the run named and no root
  note; the run is resolved after the record; a broken block keeps it open;
  a hand-opened run reports in place; live mentions are stripped.
- role and guides: prefix → role; the three guides say what they must.

The suite found one real bug: a callback into a run hit the shared
empty-topic guard, because a run holds nothing but Front's own posts.

## Left for later

- Reuse of a delegate topic across runs would route the second run's
  callbacks to the first (a topic is anchored once); the run guide asks for
  a fresh topic per delegation.
- Observation and the continue/end judgment are step3; the screen is step4;
  a live run is step5. The listener was restarted on this code so that the
  guides and the running listener agree.
