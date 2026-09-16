# Step 1 — the conversation contract and shared participation

Date: 2026-09-16 JST. Plan: [plan.md](plan.md) step 1. Nothing in this step
posts to the realm or restarts a listener; the running agents are still on
the code they had, and deployment is step 4.

## What an argue is now

| Thing | Decision |
|---|---|
| Where | `#argue › argue-<stem>`. One public channel, one topic per argue. The channel is created by the first `agentchat argue open` (a subscription creates it); the Developer is subscribed by hand at deployment. |
| Owner | Front. `argue-` is its third owned prefix (`front-`, `routinerun-`, `argue-`), served by a new `argue` role with its own profile and guide. Front is the only agent served automatically there. |
| Identity | `[selfnote][argue] from <channel>/<topic>` (or `from -`), written first by `agentchat argue open`; its own message id is the argue (`argue 7123`). A human who opens a topic by hand gets the note from Front's first serving. |
| Provenance | The `from` value is the ordinary conversation the argue grew out of. Front's reply in that conversation says where the argue was opened. |
| The desire | `[selfnote][desire] <message id> by <user id>`, written by Front's **listener**, never by the run. The run names a message in an `ag-argue` block; the listener checks it is in the conversation, not a note, not Front's own post, and by a non-bot user (Zulip's `is_bot`). A refusal is appended to the visible reply so the next serving asks again. Recorded once. |
| Invitation | `@**<bot>**` in a post. A mention is a request that costs a run; nobody is named by reflex. Front's argue replies carry no hand-off mention (`handoff=False`); a participant's reply names nobody. |
| Logical speakers | `@**<bot>** <kind>:<name>` addresses one logical participant of an account (step 2's `@**archsage** sage:arxiv`). The reply carries `**[<kind>:<name>]**` as its first line. Parsed by regex, never by a model; a mention inside a code fence is not a mention. |
| Outstanding | An invitation at message *i* to speaker *s* is answered when the account posted speech after *i* under header *s*. Read off the conversation, so several invitations in one post, other posts in between, and a restart all get the same answer. |
| Served mark | `[selfnote][served] argue/argue-<stem> <newest invitation id>`, written **into the argue topic** by the participant after it has answered everything it found. The listener's mention route is bounded by it. |
| End | Resolving the topic (`✔`) ends discussion dispatch — a resolved topic is invisible to intake and recovery. Nothing it created is resolved with it (step 3 will create things). |
| Participation guide | `agag.argue.participant_guide()` — the same text for every agent — plus each agent's short `agent/guides/argue/role.md`. `participate()` composes placement, the conversation (bounded, with the file beside it), the guide and the role context. |

## What changed

### pyagag `92f6839`

- `src/agag/argue.py` (new): the contract above as code — `mentions_of`,
  `with_speaker`/`speaker_of`, `invitations`/`outstanding`, the `argue` and
  `desire` notes, `validate_desire`, `split_block` (the `ag-argue` fence),
  `open_argue`, `participant_guide`/`participant_prompt`, and `participate`:
  the whole serving for an agent brought to an argue by a mention. No ack
  (an ack would be a post in the owner's conversation and would buy Front a
  run); a 👀 reaction on each invitation instead; one run and one reply per
  outstanding invitation; an invitation to a selector the account does not
  have is refused with one line and no run; a failed run posts the failure
  rather than silence; the served mark last, so a crash midway leaves the
  rest owed.
- `src/agag/listen.py`: the mention route is judged by the **newest
  unanswered mention above the served mark** (`unanswered_mention`), on
  intake evaluation and in recovery, instead of by whether the *last* post
  names the bot. Owners-first ordering, coalescing and the "we spoke last ⇒
  nothing owed" rule are unchanged. Recovery reads the mirror only, as
  before; a topic whose newest id is at or below the mark is not opened.
- `src/agag/chat.py`: `agentchat argue open <stem> <text> [--from
  CHANNEL/TOPIC]`, with the usage document teaching when to use it and that
  the run must not post into the argue afterwards.
- `tests/test_argue.py` (14) and the existing listener suite; **612 → 626
  passed** in pyagag.

### agfront `d07d742`

- `src/agfront/argue.py` (new): `handle_argue` → `serve_topic(handoff=False)`
  with `serve_argue`: anchor if missing, chatlog with message ids
  (`format_evidence`, so the run can name the desire), `tools/agents.md`,
  one `argue` run, then the `ag-argue` block: desire recorded or refused out
  loud; `complete: true` is logged and left for step 3. An `argue-` topic
  outside `#argue` is ignored.
- `agents.toml`: `[profiles.argue]` (claude_code / Sonnet 5) and
  `[roles.argue]` with Front's usual grant (`Read,Glob,Grep,Bash(agentchat:*)`).
- `agent/guides/argue/guide.md` (new): Front's part, the desire rule and the
  block, inviting agents by mention with the selector syntax, replying after
  every contribution, and what the conversation is for.
- `agent/guides/front/guide.md`, `character_talk/guide.md`: a paragraph on
  when and how to open an argue (`agentchat argue open`) and that opening it
  is the whole of that reply's work.
- `params/intro.md`: the convention, published — Front is served
  automatically in an argue, everybody else only when named, a mention there
  costs a run, and the participation guide travels with pyagag.
- `instance.py`: `argue-` in `extra_prefixes`, `argue` in `EXEC_ROLES`,
  `COVERS` says so. `tests/test_argue.py` (8); 165 passed.

### Participants — cagent `b53145b` (pj-clusterintent), agautolab `c242118`, agforge `8b1de98`, agobserver (pj-agdev `ed86fbc`)

Each gets a mention route whose only job is an argue invitation, a `[roles.argue]`
on Sonnet with a read-only grant, a `agent/guides/argue/role.md` (who it is
and what its contribution is), an "In an argue" paragraph in its
introduction, and a test file. The ordinary work roles are untouched:

| Agent | Before | Mention route now | Argue grant |
|---|---|---|---|
| cagent | no mention route; `Listener` built by hand | `on_mention` → `cagent_api.argue.handle_mention`; runs through cagent's own `role_run` with the nctl toolset copied into `tools/` | `Read,Glob,Grep,Bash(cagent:*)` (the operator's) |
| Observer | no mention route (deliberately: Front's reply names the notifier) | `on_mention` → `agobserver.argue.handle_mention`; a `front-` topic is not an argue, so that thank-you still buys no run | `Read,Glob,Grep` |
| autolab | callback route (`rootchat_home`) | `handle_mention` tries `handle_argue_mention` first, then the callback route as before; runs through autolab's `role_run` (its claude_code bypass) | `Read,Glob,Grep,Bash(autolab:*),Bash(git:*),ls,cat,head,find` |
| forge | no mention route | `on_mention` → `agforge.argue.handle_mention`; runs through forge's `role_run` (tool PATH) | `Read,Glob,Grep,Bash(agforge:*)` |

arxivsage is not wired: step 2 retires it into archsage.

All five consumers' `uv.lock` are at pyagag `92f6839`. Suites: agautolab 247,
agforge 245, agobserver 72, cagent 202 — all passed.

## The plan's verification list, and where each is pinned

| Case | Test |
|---|---|
| Human desire versus an agent draft | pyagag `test_the_desire_must_be_a_human_post_in_the_conversation`; agfront `test_a_desire_named_by_the_run_is_recorded_when_it_is_a_human_post`, `test_a_designation_that_is_not_a_human_post_is_refused_out_loud` (Front's draft, another bot adopting it, a message not in the conversation), `test_a_recorded_desire_is_not_recorded_twice` |
| Ordinary human turns | agfront `test_an_argue_serving_runs_the_argue_role_without_a_handoff_mention` — served as owner, the reply posted as written |
| Two addressed participants, then another post | pyagag `test_an_invitation_is_outstanding_until_a_reply_with_its_speaker_answers_it`; `test_an_invitation_survives_other_posts_and_a_restart_and_an_answered_one_stays_quiet` (Other Bot answers, the human posts, the listener still serves Mirror Bot's invitation) |
| Restart with an unanswered invitation | same listener test: a listener that was down through all of it recovers the invitation from the index; a second restart answers it |
| Specialist reply, then Front, no mention loop | same test's third phase: after the reply and the mark, Front posting without a mention serves nobody, and a new live invitation is served exactly once; agfront's `handoff=False` test |
| Several selectors on one account, one answered | pyagag `test_participate_answers_each_outstanding_invitation_without_an_ack` (two answered with headers, one unknown selector refused without a run, the mark last) |
| Resolved argue | pyagag `test_a_resolved_argue_serves_nobody` |

The human-only input checks use fixtures (a `users()` answer saying who is a
bot); no real human submission was made in this step, as the plan allows.

## Decisions worth knowing

- **The served mark lives in the argue topic itself.** Front-style
  callbacks write it into *home*; a participant has no home for an argue,
  and a selfnote in Front's owned topic is not speech, so it buys Front
  nothing. `Listener.served_marks()` reads marks by sender across the realm,
  so the location does not matter to it.
- **The listener's rule changed for every agent, not only in argues.** A
  mention is owed until the mark passes it, wherever it is. For Front's,
  autolab's and forge's existing callbacks this is a superset of the old
  rule with the same marks, so answered callbacks stay quiet. Old mentions
  that were never marked (a `#general` chat naming a bot months ago) may be
  queued once per recovery and dropped by the handler at no cost; watch the
  logs at the first restart in step 4.
- **"We spoke last ⇒ nothing owed" stays**, also for mentions. In an argue
  Front is served after every participant reply, so a second selector's
  invitation left behind by a crash is re-evaluated as soon as Front
  speaks; the handler then finds it by the header rule.
- **The ack is Front's, not the participants'.** Front's ack in the argue is
  its own post in its own topic and mentions nobody; participants' chatlogs
  filter it (`drop=is_ack` across senders in `participate`).
- **No `wait`, no timer.** While the desire is missing nothing polls; Front
  asks when the human speaks, which is the only thing that serves it.

## Handoff notes

- *Did nothing for an in-system agent in this step* — no Deus Ex Machina
  note is owed.
- Not done here, by design: creating `#argue` and subscribing the Developer,
  restarting the six listeners, re-posting the five changed introductions.
  All of that is step 4, after steps 2 and 3 change the same files again.
