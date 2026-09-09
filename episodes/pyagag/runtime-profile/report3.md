# step3 — carrying selections through autolab work

**Commits:** agautolab `61bebe3`, `d20fc09`; pyagag `7e7de55` (the entrance),
`325f4aa` (`with_default` keeps a published default), `5c277e5` (whoami cache).

## What was built

### autolab publishes a menu it cannot lie about

`agautolab.instance.exec_options()` **derives** the menu from `agents.toml`
instead of restating it: `PUBLIC_PROFILES` names the profiles autolab is
willing to be asked for, and a name whose profile is not configured is not
advertised at all. The mapping is one-to-one (the option *is* the profile
name), which the contract calls a valid start, so there is nothing to keep in
step except existence — and `configured_profiles` checks that rather than
trusting the list.

What it publishes today: `default` (pool `anthropic`), `agy` and `agy-claude`
(pool `antigravity`), `codex` (`openai`), `gemini` (`google`).

Two properties the plan asked for explicitly, both pinned by tests:

- **`covers` is true.** Every option says "my entrance, mission planning, task
  work and brain-mining". An `agy` option that changed only the entrance's
  answer would be a menu that lies, so no auxiliary role quietly stays behind
  on another pool and there is no usage-pool exception to explain.
- **`default` names its pool.** A condition of the form "until the pool is
  70 % used" cannot be judged against a default that declines to say which
  pool it consumes, so `with_default` was changed in pyagag to keep an agent's
  own `default` option rather than overwrite it with a bare one.

An unreadable `agents.toml` publishes **nothing** — leaving a reader at
*unknown* — rather than a menu that might be wrong. It also cannot take the
listener down at import: the profile names are read straight with `tomllib`,
not through the validating loader.

### The selection reaches every run

`role_run.run_role` gained `selection`, and its precedence is now three
levels with a reason for each: an explicit `profile=` from a caller (that
caller has already decided) beats the **conversation's selection** (somebody
asked *this* topic to run this way) beats the **project's standing setting**
(`.local/projects/<p>/agents.toml`) beats the role's default.

The three handlers pass their frozen `context.selection` into
`run_superdirector`, `workrun_supercoder` and `run_director`; the entrance
follows the shared skeleton (pyagag `7e7de55`).

### A task inherits its plan's selection as a snapshot

`anchor_run_topic` writes a third selfnote when the planning serving had a
selection:

    [selfnote][exec] agy from pj-agdev/workplan-runtime-profile#5731

beside the existing `[selfnote][rootchat]` and `[selfnote][work]`, before the
visible task description — so opening a topic still fires nothing. A
snapshot, not a reference: a re-plan hands its *current* selection to the
tasks it creates now and leaves the ones already under way with what they
were opened with, and a child overrides it simply by carrying its own
command, which is newer and wins by message order.

### Obeying the contract costs no extra Zulip call

The serving now reads its topic **before** the ack rather than after it, and
that same read is the serving's chatlog — so the number of reads is
unchanged and a configuration-only post buys neither an ack nor a run. The
one call the menu did add, `whoami` (the command is addressed by the Zulip
full name a mention matches), was removed again by memoizing `whoami` per
client in pyagag `5c277e5`: a bot's own id and name do not change while a
process runs, and every serving now asks for them twice.

## Checks

`uv run pytest` in agautolab: **234 passed** (219 before; 15 new). pyagag:
528 passed. No paid harness on any path.

The plan's checklist:

| check | test |
|---|---|
| two simultaneous missions use different options | `test_two_missions_keep_their_own_selections` |
| children retain the correct selection | `test_a_task_inherits_the_plans_selection_when_its_topic_is_opened`, `test_a_child_topic_runs_on_what_it_inherited` |
| a child may override | `test_a_child_command_overrides_what_it_inherited` |
| callback continuations | `test_a_callback_runs_on_the_tasks_selection_not_the_remote_topics` |
| normal defaults still work | `test_a_plan_with_no_selection_writes_no_snapshot`, `test_the_project_setting_still_applies_when_nothing_was_selected` |
| an option covers planning and task work, not the entrance alone | `test_every_published_option_covers_the_working_roles_not_just_the_entrance`, and the two "reaches the … run" tests |
| the menu comes from the real configuration | `test_autolab_publishes_only_profiles_it_actually_has`, `test_an_unreadable_config_publishes_nothing_rather_than_a_wrong_menu` |
| a configuration-only post starts no task | `test_a_configuration_only_post_in_a_task_topic_starts_no_run` |

Eighteen existing listener tests were updated: their call sequences moved the
topic read ahead of the ack and gained a second `whoami` (the fake clients
count every call; a real one answers the second from cache). Several
brittle positional assertions (`calls[5][2]`) were rewritten to select the
call by kind, so the next ordering change does not have to touch them.

## Differences from the plan

- The plan allowed "an option that uses different profiles for auxiliary
  roles, kept inside autolab". autolab needs no such mapping today — one
  profile serves every role — so `AgentSpec.exec_profile` is left unset and
  the identity mapping applies. The seam exists and is tested in pyagag
  (`test_an_agent_may_map_an_option_differently_per_role`); autolab uses it
  the day one of its roles has to diverge.
- The `whoami` memoization is not in the plan. It was added because the
  contract would otherwise have cost one extra Zulip call per serving across
  the whole fleet, which this realm has been bitten by before (the p1 sweep's
  HTTP 429).

## Not done in this step

Nothing is deployed and no introduction has been re-posted, so the running
autolab still advertises no options — that is step5. Front does not yet ask
for one; that is step4.

## Commits

- agautolab `61bebe3`, `d20fc09`
- pyagag `7e7de55`, `325f4aa`, `5c277e5`
- devdocs — this report.
