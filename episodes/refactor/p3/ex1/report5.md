# refactor p3 ex1 — step 5: deployed, and proved live

## Deployed

| what | how | checked |
|---|---|---|
| `agdevworld` static build | `docker compose up --build -d web` | `:8090` HTTP 200; `grep autolabState\|api/autolab` over the served assets finds nothing |
| forge, sage, front, autolab listeners | `launchctl kickstart -k` each | each logs its startup sweep clean, and **no pool diagnostic** — every declared pool is one the instance really spends |
| dependency locks | `uv lock --upgrade-package pyagag` in agfront, agautolab, agforge, arxivsage, agecho | all five moved `0d26145a → 36653e30` |
| submodule pointers | committed in `pj-agdev` and `pj-clusterintent` | — |
| introductions | `python -m agforge.intro`, `python -m arxivsage.intro` | see below |

Front's and autolab's introductions were **not** re-posted, and that is the
result rather than an omission: their published blocks are byte-identical
before and after the derivation, because their declared pools were already
the pools their roles resolve to. Re-posting to say the same thing would add
a post to `#agents` and change nothing a reader acts on.

Gates: `nctl` 1335 · `agentroom` 267 · `agforge` 241 · `agautolab` 242 ·
`agfront` 109 · `pyagag` 532 · `arxivsage` 16 · Ansible conformance 4 ·
`retire_agag_agent.yml` syntax-check clean.

## Discovery, live

`agentchat options` now reads every agent. The two that p3 left as *unknown*
are not unknown any more:

```
agforge-agstudio1: @**agforge-agstudio1** use <option>
    `default` … pool `anthropic` … `agy` … pool `antigravity` … `agy-claude` …
arxivsage-agstudio1: @**arxivsage-agstudio1** use <option>
    `default` … pool `anthropic` … `agy` … pool `antigravity`
```

One agent still reads as unknown — `cagent-agstudio1`, which predates the
contract and is outside this plan.

## Live: the sage, end to end

| step | evidence |
|---|---|
| selection | `@**arxivsage-agstudio1** use agy` alone → one deterministic line, **no run** |
| the run | asked a question; answered from the tree, citing `knowledge/README.md` |
| the **record** | `profile: agy, harness: agy, provider: antigravity` beside `exec_option: agy, exec_source: topic, exec_message_id: 5974` |
| refusal | `use opus` → *"I do not publish an execution option named `opus`. Mine are: `default`, `agy`. This topic still runs on `agy`."* — names the poster, names the menu, states the standing selection |
| reset | `use default` → confirmation; the **next** run in the same topic recorded `profile: sonnet, harness: claude_code, provider: anthropic`, `exec_option: None, exec_source: topic, exec_message_id: 5981` |

That last row is the one the plan asked for by name. "Nobody selected
anything, and here is the message that said so" is a different fact from
"this predates the contract", and the written record tells them apart —
returned dictionaries would not have.

## Live: forge, across planning and generation

Request: a 512×512 badge, in `assetplan-ex1-badge`, under `use agy`.

| run | profile / harness | exec |
|---|---|---|
| planning front | `agy` / `agy` | `option agy, source topic, message 5986` |
| planning generator | `agy` / `agy` | `option agy, source topic, message 5986` |
| execution generator | `agy` / `agy` | `option agy, source **inherited**, from `agforge-agstudio1/assetplan-ex1-badge`` |

The inheritance is visible in the realm, not only in the record — the run
topic forge opened carries it as its second post:

```
5996: [selfnote][rootchat] agforge-agstudio1/assetplan-ex1-badge
5997: [selfnote][exec] agy from agforge-agstudio1/assetplan-ex1-badge#5986
5998: [selfnote][assetrun] 5991
```

The asset was really produced and delivered — a dark-navy circle with a pale
green check, 512×512 RGBA, zipped to MinIO and posted back with its durable
key.

**And it illustrates the distinction the menu makes.** Asked to run on `agy`,
forge planned the badge with Pillow rather than a diffusion model — a
perfectly good plan for flat vector-ish artwork. The execution option chose
*which harness thought about the request*; what makes the asset stayed the
plan's business, exactly as `covers` says.

## Live: an agy-selected autolab task resuming from a callback

This is the property p3 did not establish. `workplan-ex1-inherit` in
`#pj-papers`, `use agy`, a one-task mission: ask forge for a 128×128 grey
square, wait, and record the delivered key.

```
superdirector  agy/agy   option agy   source topic       message 6009
supercoder     agy/agy   option agy   source inherited   from pj-papers/workplan-ex1-inherit
supercoder     agy/agy   option agy   source inherited   ← the callback continuation
supercoder     agy/agy   option agy   source inherited   ← the delivery continuation
```

and in the listener log, the callback route firing:

```
serving mention in 'agforge-agstudio1'/'assetplan-ex1-grey-square'
mention in 'agforge-agstudio1'/'assetplan-ex1-grey-square' serves work-m6013/workrun-task1-m6013
```

**The continuation ran on the inherited option.** The post that woke autolab
was in *forge's* topic, which has no selection of its own; the selection was
read from home, and the run record proves it — `exec_source: inherited`,
`exec_inherited_from: pj-papers/workplan-ex1-inherit`, `harness: agy`.

One more thing worth writing down: **forge served the delegated request on
its own default**, not on `agy`. autolab did not forward the name, and it
should not have — `agy` is autolab's public vocabulary and forge's happens to
spell it the same way. A further delegation means discovering *that* agent's
options and translating the intent again.

## Live: Front's routine decision, on five controlled observations

Five `ag.budget.v1` fixture documents were put in front of **real servings**
via `AGFRONT_BUDGET_URL` in the listener's plist (temporary, removed below).
Each ran the `routine_run` role for real. The condition throughout: *"until
agy's usage exceeds 70 %"*.

Every document also carried a `claude_code` card well under any threshold, so
a run that matched the condition to the wrong harness would still have found
a number. None did — every serving named the pool first.

| case | observation | Front's decision |
|---|---|---|
| **below** | agy 5-hour 31 %, weekly 22 % | pool `antigravity`, section named, both windows read, **not met**; run left open |
| **exceeded mid-run** | the same run, re-read at 84.5 % | **met**; run ended with the `ag-routinerun` block; entry cites both reads |
| **already exceeded** | a fresh run opened at 91 % / 73 % | **met at the very first read**, "no work was started" — not one lap first |
| **reset** | 3 %, after an opening post stating an earlier 84.5 % | **not met**: *"that drop is recorded as a reset … judged fresh rather than treated as already-satisfied"* |
| **stale / failed read** | read failed; last good 88 %, its window already reset | **cannot be judged** — *"A failed read is not 0 and not 'reached', and a stale number whose window has already reset says nothing about the current window either"* |

Two confirmations the plan asked for, in Front's own words:

- **the requested pool is the pool observed** — every entry names
  `antigravity` and says where it got it: *"the pool the `agy` execution
  option consumes (confirmed in `tools/agents.md`, e.g. front-agstudio1's
  exec-options block: `option: agy | pool: antigravity`)"*. It read the pool
  off a published introduction, not a table of its own.
- **unknown usage is not reported as success** — the stale run's finish block
  says `achieved: true` about the *routine's* goal (having judged the
  condition on the observation in hand) while reporting the condition itself
  as unjudgeable. Those are the two sentences `refine_routine` p1 separated,
  and keeping them apart is what stops "I could not read it" from becoming
  "it was fine".

No real account was consumed to 70 %. `AGFRONT_BUDGET_URL` was removed from
the plist and the listener reloaded; `agbudget` reads the relay again
(`agy … 9.4 % weekly, 8.8 % 5-hour`, live).

## Fixture evidence versus live evidence

Kept separate on purpose. Everything in step 2's table is fixture coverage;
everything above is a real serving with a written run record. The one
property still covered only by a fixture is **forge's ComfyUI callback
collection under an inherited option** — the badge was an image, which
SwarmUI returns synchronously, so no notifier callback was involved. The
plan permits exactly that ("use fixtures for expensive asynchronous edge
cases unless live behavior remains unclear"), and the behaviour is not
unclear: the collecting run is an ordinary serving of the same topic, and the
*autolab* callback above proves the identical "read the selection from home,
never from the topic that called you back" rule live.

## What the demonstration cost

18 runs by this session. Nine of them ran on `agy` and cost **no dollars** —
they moved the Antigravity window instead (`agy` 5-hour 8.1 % → 8.8 %), which
is the pool the menu advertises for them, observed. The nine
`claude_code` runs cost **$1.03**.

(Other run records written in the same window belong to a second Omni session
on this account and are not counted here.)

## Test work closed out

Through the system's own paths, not by hand:

- `arxivsage-agstudio1 › entrance-ex1-agy` — ✔
- both forge requests — `python -m agforge.retire`, which renamed and ✔'d the
  plan and run topics as a pair
- the autolab mission — accepted in the task topic, autolab marked it
  completed, `python -m agautolab.mission_done m6013`, `workplan-ex1-inherit` ✔
- the demonstration's local commit in the `papers` project — dropped; it was
  never pushed, and a verification file is not that project's work
- the throwaway `ex1-budget` routine — ✔ on its guide (the realm's retirement
  flag), Front unsubscribed, channel taken out of the `routine` folder and
  archived
- the `AGFRONT_BUDGET_URL` plist entry — removed, listener reloaded

The operation-room board is back to five instances, three retired
(`agecho-agautolab1` among them), and every row on it is a `done` receipt —
nothing stalled, nothing awaiting.

## One non-issue, noted so it is not re-investigated

The forge listener logged `unknown toolset 'toolset'; skipped` during the
delegated request. That is the header row of a `toolsets.csv` the front
wrote, and `toolsets.parse_names` documents the behaviour: "a header row
naming no toolset resolves to nothing later and costs only a log line". The
right toolset (`toolset-image`) was placed.
