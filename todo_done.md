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

- Decide what agforge's assetrun should do about long generations. Its guide
  carries the notifier paragraph, but an assetrun **cannot act on it**: its
  only image tool is synchronous and returns a download URL rather than a
  ComfyUI `prompt_id`, and the run has no way to post a chat message. Forge
  said so itself in `agforge-agstudio1 > assetplan-red-apple` (2026-09-01,
  `episodes/zulip_command` step 4) and wrote an `idea.md` naming the two tool
  additions. Either give forge an async submit and a chat post, or take the
  paragraph out of its guide — do not leave it teaching something the run
  cannot do. (Recorded 2026-09-01.)

- An agent answering the last speaker can silently misdirect a whole mission.
  In `episodes/zulip_command` step 4 the notifier's one error line made Front,
  forge and autolab each reply to *it* instead of their real requester; once
  that stalled a mission with nobody waiting on anybody, and only a human
  noticed. Nothing in the system detects "a conversation has stopped" or "a
  conversation has become a loop". Worth an ENT episode.
  (Recorded 2026-09-01.)

## Done
