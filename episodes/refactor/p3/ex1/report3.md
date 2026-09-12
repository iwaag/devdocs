# refactor p3 ex1 — step 3: a published pool is what actually resolves

## The lie that had not happened yet

Four agents advertised `pool: anthropic` for their default option, and every
one of them was right. They were right **by coincidence**: each one's roles
happened to point at `claude_code`, and the pool beside the name was a string
in a tuple that nothing ever compared to a harness.

The name half was already safe — `exec_options()` filters public names against
the configured profiles, so a name cannot be advertised without a profile
behind it. The pool half was not. One line in a machine's
`agents.local.toml` moving a role to `agy` and every published default keeps
saying `anthropic` while that role spends the Antigravity account — and a
routine run bounded by "until agy's usage exceeds 70 %" reads the wrong
window and stops at the wrong time, or never.

Front's own configuration is where this was closest to happening: the Front
Desk voice has its own profile **specifically so it can be moved** without
touching the ordinary entrance.

## What replaced it

`pyagag/src/agag/execpool.py` — one derivation, for every agent:

    option -> profile (the agent's private mapping, per role)
           -> harness (agents.toml + this machine's overlay)
           -> pool    (agent_config.HARNESS_PROVIDER)

`AgentSpec` gained `exec_roles`: the roles an option covers, which is what the
pool is derived from. Each agent now names them beside the sentence it already
had, and a test in each repository asserts the two are about the same work:

| agent | `COVERS` | `exec_roles` |
|---|---|---|
| front | this entrance, the Front Desk and routine runs | `front`, `character_talk`, `routine_run` |
| autolab | my entrance, mission planning, task work and brain-mining | `front`, `superdirector`, `supercoder`, `director` |
| forge | entrance replies, asset planning, generation, collection | `front`, `generator` |
| arxivsage | every answer I give | `front` |

autolab's `mediator`, `coding` and `summarizer` are configured but no live
serving launches them, so they are not covered and not derived from.

**What is published is the derived value, never the declaration.** The block
is generated from the running instance so that it cannot drift; making it
carry an unverified string was the one place that promise leaked. The
declaration survives as an *assertion*, and it is compared against the
derivation — `pool_diagnostics()` at listener startup and again before an
introduction is posted, naming the option, both pools, and the
role/profile/harness that produced the derived one.

## Three distinctions the shape depends on

**Mixed is truthful, not an error.** An agent whose planning resolves through
`claude_code` and whose task work resolves through `agy` really does spend two
accounts, and `pool: anthropic+antigravity` says so. Forcing a single name
would make the menu lie in the one case where the lie costs the most. The
`routine_run` guide learned the rule: an option whose pool is several names
joined by `+` is judged against **every** one of their windows, and the first
to reach the threshold decides.

**Unavailable is not invalid.** Derivation runs with `check_available=False`.
A harness whose binary or secret is missing right now is a runtime failure of
that one option (`E_UNAVAILABLE`, which the contract already keeps separate)
— never an unpublishable contract, and never a reason to take an unrelated
conversation down. Availability is collected alongside and reported
separately, and a test in every repository pins that an uninstalled CLI
produces **no** wrong-declaration diagnostic.

**A degraded derivation says so.** Following the review correction, a
configuration this instance cannot read leaves the options available with
`pool: unknown`, and `pool_diagnostics()` returns why. The original fallback
kept declared pools and logged the failure, which hid the uncertainty from
the requesting agent; that fallback has been removed.

Pool vocabulary stays provider-level, which is this environment's convention:
one account per provider. Nothing found two accounts behind one provider, so
no account registry was built — if one ever appears, the distinction belongs
in the execution and the observation, not in a table here.

## Verified

`pyagag` 532 passed · `agfront` 109 · `agautolab` 242 · `agforge` 241 ·
`arxivsage` 16.

Coverage of the four cases the plan named:

| case | where |
|---|---|
| **wrong pool** — a declaration that disagrees is corrected in the block and named | `test_with_derived_pools_replaces_the_declaration_and_keeps_the_prose` |
| **changed default overlay** — a role moved in `agents.local.toml` moves the published default | one test per agent, on that agent's own real committed config |
| **role-specific mapping** — an option meaning different profiles for different roles | `test_a_role_specific_mapping_is_honoured` |
| **mixed defaults** — two providers cannot become one advertisement | `test_a_mixed_default_is_said_truthfully_rather_than_rounded`, and `agcode`'s unpriceable account survives as `unknown` rather than being dropped |

And once **live on the real instance**, which is the evidence that matters
most here because the failure is a configuration one: forge's own
`.local/agents.local.toml` was edited to send `generator` to `agy`, the menu
was read back, and the file was restored.

```
with generator moved to agy in the real overlay:
  default    pool: anthropic+antigravity
  agy        pool: antigravity
  agy-claude pool: antigravity
  diagnostic: option 'default' declares pool 'anthropic' but resolves to
              'anthropic+antigravity': front -> sonnet/claude_code (anthropic),
              generator -> agy/agy (antigravity)
restored; declarations now agree again: True
```

All four agents currently report **no** diagnostics: every declared pool is
one they really spend.

## Not done here

The listeners are still running the previous code, so nothing above is in
force in the realm yet and no introduction has been re-posted. Deploy,
restart and repost are step 5's, together with the live demonstrations.
