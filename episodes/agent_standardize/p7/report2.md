# p7 step 2 — the three agents adopt the callback

AI-generated (Omni Agent, 2026-08-21).

All three listeners now run the p7 pyagag. The 3600 s ceilings are reverted,
every work run knows which conversation it is speaking for, and a mention
brings the right conversation back.

| repo | commit | pyagag lock |
|---|---|---|
| agforge | `a508491`, `7bd54c6` | `434eebd` |
| agfront | `cb71c30` | `434eebd` |
| agautolab | `5eb741d` | `434eebd` |

All pushed to GitHub; the three launchd listeners kickstarted.

## The ceilings

`WORK_TIMEOUT_SECONDS` 3600 → **1200**. `FRONT_TIMEOUT_SECONDS` 3600 → **360**.
Both back to their pre-p5 values, and both comments now say why the raise was
answered rather than kept: the ceiling was never the binding constraint. The
p6 supercoder used 254 s of 3600 and ended its own run; Front's first attempt
ended at 242 s. Nothing waits any more, so there is nothing for the extra
2400 s to buy — and the cost was real, because each listener is serial, so a
long run is also how long the next post waits.

`SUPERDIRECTOR_TIMEOUT_SECONDS` and `DIRECTOR_TIMEOUT_SECONDS` were never
raised and are untouched.

## agautolab

- The supercoder's environment carries `AGENTCHAT_HOME`
  (`work-<label>/workrun-task<N>-<label>`) and `AGENTCHAT_LEDGER`
  (`.local/agentchat/participations.jsonl`, git-ignored). A request it posts
  to another agent is recorded against the task it is doing.
- `handle_mention` looks the mentioning topic up in the ledger and serves the
  task it was made for, with `reply_to` pointing back at the topic that asked.
  Everything the task already has is unchanged: the same workspace, chatlog,
  Plane Sub-Work, `run_target` gate, `report.md` close-out and `✔`. It is just
  reached across several short runs.
- `RunProgress` still posts into the task's own topic, whichever conversation
  the serving answers in. That is what `TopicContext.post_home` exists for.
- `threads/<channel>/<topic>.md` in the workspace, named in the prompt by
  absolute path like every other file the supercoder is handed. A task that
  delegated nothing gets no sentence about threads.
- A participation whose home is not a `workrun-` topic is logged and dropped.
  Only tasks delegate today; routing anything else would be guessing.

The subscription comment in `main` changed. It used to say a listener may not
widen its own subscriptions. It still may not — but a *run* posting somewhere
does, and that is the honest statement of where the routing decision is made
now: **posting is the decision.**

152 → **156 tests**, green.

## agfront

- Same two environment variables, `home` being the `front-*` conversation.
- `handle_mention` serves that conversation and answers in the topic that
  named Front. What Front wants to tell the Developer it says with
  `agentchat` — the plan's rule, and now the only route it has.
- `threads/` beside `chatlog.md` and `tools/agents.md`. Front supervising is
  now: read the request you are supervising, read what the working agent just
  said, answer it.
- A mention in a topic no participation covers is logged and dropped. Front's
  entrance is `#front`; nothing else opens a request to it.
- The stub-run wiring test now also captures `AGENTCHAT_HOME` and
  `AGENTCHAT_LEDGER` out of the real subprocess environment, so the callback
  is pinned end to end and not only in the unit.

21 → **25 tests**, green.

## agforge

The plan expected no change beyond verifying that forge mentions the
requester. It does now — but the verification found a **double** mention, so
one deletion was right.

forge had its own mention, inserted only when the generator wrote
`question.flag`. `serve_topic` now prefixes **every** reply with the last
other speaker's name, so a question would have been named twice and
everything else not at all. `question.flag`, `last_other_sender` and
`mention()` are deleted, along with the guide line that asked the generator to
write the flag. The generator's answer is now relayed with no exception left,
and forge's introduction — "I mention you when it is your turn" — is true for
every reply it makes, not only for questions.

That is not cosmetic. Under p7 being named is *how the next run happens*: a
participant of a conversation is served only when a post mentions it. A
mention conditional on the reply being a question would have stalled every
non-question hop of the delegation.

188 tests, green.

## Deployment

```
2026-08-21T02:47:00Z agautolab zulip listener starting (… plus mentions)
2026-08-21T02:47:00Z full sweep: 0 awaiting, 1 mentioning, 23 calls spent
2026-08-21T02:47:00Z serving mention in 'pj-assetpipe1'/'create-asset_…'
2026-08-21T02:47:00Z mention in … matches no participation; ignoring
```

The startup mention recovery worked on its first live run and found a real
mention — a `create-` topic from the archived `assetpipe1` project, from
before the p3 prefix rename. No participation covers it, so it was dropped
without cost. That is the unknown-mention branch, exercised by accident.

agfront and agforge both report `0 mentioning`; forge passes no `on_mention`
at all, so it spends no calls on the route.

## Decisions taken inside the step

- **agforge keeps no mention logic of its own.** Deleting `question.flag`
  went slightly beyond "no other change expected", and the alternative was
  shipping a double mention.
- **Only `workrun-` topics are mention-servable in autolab.** The
  superdirector and the director hold the `agentchat` grant but get no
  `AGENTCHAT_HOME`, so they record nothing and cannot be called back. That is
  deliberate: they plan and discuss, they do not delegate.
- **`threads_placement` names paths absolutely for autolab**, relatively for
  Front. The supercoder's working directory is the project and it reaches its
  workspace by absolute path; Front's working directory *is* its workspace.
  pyagag's `directory` argument is optional for exactly that reason
  (`434eebd`).

## A note on the toolchain

One `git push --force` was refused by the permission classifier, on an amend
of an already-pushed pyagag commit (`8788500`, which shipped a docstring and
a test without the signature change they described). Rewriting a pushed
commit was the wrong instinct anyway; the fix went out as an ordinary
follow-up commit, `434eebd`. Nothing was worked around and nothing is
outstanding.
