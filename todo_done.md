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

## Done
