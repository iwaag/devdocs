# TODO / Done

Cross-project todo list. Per-project items live in each project's own
`devdocs/todo_done.md`.

## TODO

- Unify agent entrances on the unauthenticated `window`. cagent currently
  keeps three doors: the node entrance (`:8788`, mTLS), the human entrance
  (`:8789`, bearer token + chat UI), and — from
  `episodes/better_communication/zulip_cagent_receive` — the unauthenticated
  `window`. The window is the intended single entrance; the other two are
  transitional. Retire them once their remaining consumers are moved:
  `agdevworld`'s `cluster:fetch` (human token) and any node/mTLS caller.
  The same rule applies to every new agent door: new conversational
  entrances are `window`, unauthenticated, with their own guide and their
  own tool-permission set — no new auth mechanism per agent.
  (Recorded 2026-08-12, from the zulip_cagent_receive discussion.)

- An agent answering the last speaker can silently misdirect a whole mission.
  In `episodes/zulip_command` step 4 the notifier's one error line made Front,
  forge and autolab each reply to *it* instead of their real requester; once
  that stalled a mission with nobody waiting on anybody, and only a human
  noticed. Nothing in the system detects "a conversation has stopped" or "a
  conversation has become a loop". Worth an ENT episode.
  (Recorded 2026-09-01.)

## Done
- **agforge's assetrun can use the notifier (2026-09-01).** The paragraph its
  guide carried was un-runnable — no `prompt_id` from a synchronous
  `agforge image generate`, and `[roles.generator]` has no `Bash(agentchat:*)`.
  Resolved the way forge is built rather than by widening the run: `agforge
  video submit` / `music submit` return a `prompt_id`, the generator writes
  `pending.json`, and **the listener** posts the notifier command and holds
  the Work open; the callback triggers the run that collects the outputs with
  `agforge comfy fetch`. The generator still has no Zulip voice. Image stays
  synchronous (SwarmUI has no `prompt_id`, and it returns in seconds).
  See `episodes/zulip_command/report5-agforge.md`.

