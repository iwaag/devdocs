# step4 — teaching Front to request execution options

**Commits:** agfront `40c48dd`, agautolab `f197832` (its introduction's
prose), pyagag `201d189` (the pool convention).

## What was built

### Front publishes its own menu, and it is about Front

`agfront.instance.exec_options()` derives Front's options from its
`agents.toml` the way autolab's does: `default` (pool `anthropic`), `agy`,
`agy-claude`, `codex`, `gemini`. What differs is the `covers` phrase — *"my
own conversations: this entrance, the Front Desk and routine runs"* — because
the distinction the plan insisted on has to be visible in the menu itself:

> asking Front to run **its** conversations on agy, and asking Front to have
> autolab run a mission on **autolab's** agy, are different requests.

Honouring the second changes nothing about the first. The listener passes
`context.selection` to the `front`, `character_talk` and `routine_run` runs
alike; a callback reads it from **home**, never from the topic that called
Front back.

### Matching a threshold to the pool it is about

This is the half of the plan that needed a real decision. An option
advertises a **pool**; `agfront.budget` reads a document keyed by **harness**.
"Until agy's usage exceeds 70 %" is only answerable when those two can be put
side by side.

The correspondence is **derived, not invented**: a pool is the provider whose
account the harness spends, and `agag.agent_config.HARNESS_PROVIDER` is
already the table the configuration validates profiles against. `budget.pool_for`
reads it; `render` prints `## agy (plan pro, pool antigravity)`; the contract
document now states the convention (pyagag `201d189`).

`agcode` is deliberately absent from that table — its account follows the
model it is pointed at — so it renders as `pool unknown` and can be matched to
no option at all. That is the honest answer, and the guide says an unmatched
or unreadable pool is an **unobservable** condition rather than a condition at
0. The same rule the whole budget module already lived by, extended to the
new axis.

### The guides

`agent/guides/front/guide.md` gained: how an agent executes is something you
may ask for and never assume; `agentchat options` prints what each has
**published** and those names are the only ones usable; `unknown` is reported
as unknown and never resolved by trying a name; translate the developer's
intent ("using agy", "on the cheap model") into an option that agent actually
publishes and say which you chose; select it with `agentchat use` in the topic
whose work it applies to, *before* posting the request, because it is
configuration and starts nothing; a further delegation means discovering
*that* agent's options again, because an option name is one agent's
vocabulary. And: doing this for somebody else does not change how Front runs.

The routine section now says the opening post must carry the execution
preference **in the developer's own words** — later servings and delegations
read the opening post, and a preference acted on once is a preference the run
forgets.

`agent/guides/routine_run/guide.md` gained: honour that preference at *every*
delegation; `agentchat options <agent>`; an agent publishing nothing is
unknown and is reported through the report rather than guessed at; a
published-but-unavailable option comes back as a failed serving in that
agent's topic and is recorded, never silently retried elsewhere. Its usage
section now begins by naming **which window is being judged**, keeps
"exceeds N" (strictly past) distinct from "reaches N", and restates that the
condition is about the shared account window, not this run's `cost_usd`.

### Introductions

Front's and autolab's `params/intro.md` gained the prose beside the generated
block: the command is configuration and starts nothing, a task topic inherits
its plan's setting at creation and may be given its own afterwards, and an
unpublished name is refused out loud rather than quietly downgraded.

## Checks

agfront: **103 passed** (91 before; 12 new). agautolab: 234. pyagag: 528.
Nothing paid.

The plan's checklist, all with fixtures:

| check | test |
|---|---|
| discover an agy option from an introduction | pyagag `test_options_prints_each_agents_published_menu`; agfront `test_front_publishes_only_profiles_it_actually_has` |
| request it | pyagag `test_use_posts_the_command_line_and_nothing_else` |
| continue across callbacks | `test_a_callback_answers_home_under_homes_selection` |
| a run keeps its own selection | `test_a_run_topic_carries_its_own_execution_selection` |
| explain why a run ended, from the observed usage | `test_an_already_reached_threshold_is_readable_off_the_matched_pool` over `agy.json` (71 %) |
| already-reached | the same fixture, plus the existing `reached.json` |
| unavailable / unreadable | `test_a_failed_read_of_the_matched_pool_is_unknown_not_zero`; `test_an_unpublished_option_is_refused_and_names_the_menu` |
| unmatched pool | `test_a_harness_whose_account_follows_its_model_says_unknown`, `test_the_observation_names_each_sections_pool` |
| a configuration-only post starts no Front run | `test_a_configuration_only_post_in_a_front_topic_starts_no_run` |

New fixture `tests/fixtures/budget/agy.json`: `agy` at 71 % (a "exceeds 70 %"
condition already reached), `gemini_cli` unreadable (unknown, not 0), `agcode`
unmatched.

## Differences from the plan

- The plan says "Keep recipient names, local profile names, and role mappings
  out of Front's routing code". They are: `grep` for `agy` in `src/agfront`
  finds only Front's **own** published menu, which is its own configuration,
  not a recipient's. The routing code names no other agent, as before.
- The plan did not ask Front to publish options of its own. It does, because
  the developer can also ask Front itself to run differently, and because
  publishing makes the distinction between the two requests legible instead
  of implicit.

## Not done in this step

Nothing is deployed: no introduction has been re-posted, so the running
agents still advertise nothing, and no live run has used an option. That is
step5.

## Commits

- agfront `40c48dd`
- agautolab `f197832`
- pyagag `201d189`
- devdocs — this report.
