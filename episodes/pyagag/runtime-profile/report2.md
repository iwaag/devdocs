# step2 — selection and discovery in pyagag

**Commit:** pyagag `eb74b91` (on `0dbc3e9`, step1's contract).

## What was built

### `agag.execopt` — the whole vocabulary in one module

The published block, the command, the inheritance note, the resolution and
the run-record fields, so every agent means the same thing by them.

- `Option` / `ExecOptions` — a public name with the usage **pool** it consumes
  and the work it **covers**. `with_default` always puts `default` first,
  because the reset command names it.
- `parse_options` returns `None` for a post with no block, and the docstring
  says what the caller must do with that: keep it as *unknown*. Same rule as
  `parse_roster`, same reason.
- `split_commands` / `parse_command` — **a message is a command only when it
  is nothing else.** Every non-blank line must be a command line addressed to
  this bot. That is what makes *"I asked forge to `@**Autolab** use agy`"*
  ordinary speech, and it makes a fenced example fall out for free: a fence
  line is not a command line, so the parser never has to know what a fence is.
- `resolve(messages, bot, up_to=…, known=…)` — the newest directive wins,
  command post or inherited snapshot alike, so "a child overrides what it
  inherited" is message ordering rather than a special case. `known` filters:
  a name that is not published is **skipped**, never adopted, so a typo cannot
  silently stop a topic running on what it was running on.
- `exec_note` / `parse_exec_note` — `[selfnote][exec] <option> from <ch>/<t>#<id>`.
  A selfnote, so it buys nobody a run (pinned by a test).
- `run_meta` — `exec_option`, `exec_source`, `exec_message_id`,
  `exec_inherited_from`.

### `serve_topic` obeys it, so agents get it by naming their menu

`serve_topic(..., exec_options=…)`:

1. **reads the topic before its ack.** A configuration-only post must cost
   neither an ack nor a run, and the ack is our own message, so nothing is
   lost by looking a moment earlier. Without `exec_options` the ordering is
   byte-for-byte what it was.
2. `apply_exec_commands` answers the commands awaiting this bot. When
   *everything* waiting is a command, the setting is applied, one
   deterministic line is posted and the serving returns — no workspace, no
   model. When work is waiting too, the command is still applied (it is in
   the history like any other directive) and the topic is served normally.
3. otherwise the selection is **frozen** at `processed_up_to` and handed to
   the handler as `context.selection`.

Two decisions worth keeping:

- **A line, not a reaction.** The ComfyUI notifier reacts because a *post* by
  a non-owner serves the topic's owner and buys a run. Here the answering
  agent *is* the owner, so the reasoning inverts: a reaction would leave the
  poster as the last speaker and the topic would match every sweep forever.
  The owner's own line is what settles it, and it costs no model.
- **A confirmation names nobody; a refusal names the poster.** An agent that
  asked for `opus` and heard nothing asks again, so a refusal must reach it —
  one run, and the correct one. A confirmation is not worth a run at the
  other end.

### The spec publishes, the spec maps

`AgentSpec.exec_options` is the public menu (without `default`, which is
prepended); `AgentSpec.exec_profile(option, role)` is the private mapping,
**identity by default** — the option *is* the profile name — so an agent whose
option names match its profiles writes no mapping at all, and autolab's
per-role divergence in step3 stays inside autolab.

`run_role(..., selection=…)` turns the selection into `profile_override` and
stamps the public half into the record beside the harness's own
`profile`/`harness`/`model`: the option is what was asked for, those are what
ran.

### Discovery and selection from a run

`agentchat options [<agent>]` prints each agent's published menu — or
`unknown` for one with no block, which is printed as a third answer beside
"does not support", because they are different. `agentchat use <channel>
<topic> <option> --to "<name>"` posts the one command line, says it started
no work, and anchors the topic with the ordinary rootchat note first — which
is what lets a refusal, the one message that names the poster, find the
conversation the request came from.

### The introduction

`intro_text`/`post_intro` take `options=` and append the block after the
roster's; `intro.parse_exec_options` reads it back. `agent.intro_main` passes
`spec.published_options(roster.bot)`, so what is published is generated from
the running instance and addressed by the Zulip *full name* the mention
actually matches (`Front`, not `front-agstudio1`).

## Checks

`uv run pytest` in pyagag: **526 passed** (470 before; 56 new). No paid
harness is launched on any path — the new tests use fakes, and the
configuration-only path is proved to reach no handler at all.

The plan's checklist, and where it is pinned:

| check | test |
|---|---|
| selection | `test_a_bare_command_line_selects`, `test_the_selection_reaches_the_handler_frozen_at_the_servings_start` |
| replacement | `test_the_newest_directive_wins` |
| reset | `test_reset_returns_to_the_defaults_rather_than_a_mode_of_its_own` |
| invalid requests | `test_an_unpublished_name_never_becomes_the_topics_setting`, `test_a_refused_name_is_posted_visibly_and_names_the_poster` |
| restart recovery | `test_a_restart_re_derives_the_same_selection_from_the_topic_alone` |
| resolved topics | reuses `serve_topic`'s swept name and `agentchat`'s `refuse_resolved` — `test_use_refuses_a_resolved_conversation_like_send_does` |
| configuration-only posts | `test_a_configuration_only_post_costs_no_ack_and_no_run`, `test_a_command_beside_unanswered_work_is_applied_and_the_topic_is_still_served` |
| no paid model | the configuration-only tests assert the handler was never called |
| mid-run command | `test_a_command_arriving_during_a_run_is_handled_on_the_next_pass` |

## Differences from the plan

- The plan suggested "existing selfnote filtering and the notifier's reaction
  approach are useful references" for the acknowledgment. The selfnote
  filtering is used directly (`pending_speech` is built on
  `selfnote.is_speech`); the reaction approach is deliberately **not**, for
  the ownership reason above. That is a documented departure, not an
  oversight.
- Availability validation stays where the plan put it — execution time,
  `agent_config`'s existing `E_UNAVAILABLE` — so nothing was added for it.

## Not done in this step

No agent publishes anything yet: `AgentSpec.exec_options` is empty for every
instance, so every introduction is byte-identical to what it was and every
serving behaves exactly as before. Wiring agfront and agautolab is step3/step4.

## Commits

- pyagag `eb74b91` — the implementation and its tests.
- devdocs — this report.
