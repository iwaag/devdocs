# refactor p3 ex1 — the follow-ups are closed

p3 finished by naming five things it had deliberately not done. All five are
done, and the work that closed them turned up four defects nobody had reported
— every one of them the same shape: *a declaration that had stopped being
true, with nothing checking.*

## What p3 left, and where it stands

| p3's open item | now |
|---|---|
| `autolabState.ts` still calls the deleted `/api/autolab/*` gateway | deleted, with the view, its popups, and the node's stub routes |
| agforge and arxivsage read as *unknown* execution-option-wise | both publish menus, and **run on them** |
| `service_missing` on the `agfront` service | resolved — it was an obsolete declaration, not a stale observation |
| two `agent_zulip_channel_unsubscribed` gaps | resolved: one stale expectation replaced, one fixture agent retired |
| the missing selection/budget evidence | live, with written run records |

`nctl drift`: **converged=43, zero errors.** `agentchat options`: every agent
publishes a menu except cagent, which predates the contract.

## Deletions

**The `autolab / now` view** and everything behind it. `src/autolabState.ts`
(248 lines), the view config, the job and project popups and their CSS, the
scene, the nav entry — and on the node, the **stub surface**
`/status`, `/log`, `/jobs…`, `/projects`, `/game`, `/monitor` (181 lines),
whose own docstring said it existed so `agdevworld`'s proxy "keeps working".
That proxy had been deleted a month earlier, so every action on the screen
had been failing silently and its documentation admitted it was reading
sample data. Five views now, and what autolab is doing is read from the realm.

**What was not deleted matters as much**: `POST /window` and `GET /healthz`
stay. One file held six dead routes and one live conversational entrance, and
a dead browser caller does not make a whole service dead — the plan said so
and it was literally true here.

**Disposable pre-refactor work**: seven `work-<Plane label>` channels, each
holding one resolved topic and named after a label that stopped naming
anything when Plane was deleted; three finished per-phase verification
projects (`refactorp1/2/3`) through `agautolab.project_archive`; and the
`agecho` fixture. The agent room's sweep went from **20 channels to 10**.

## Public option coverage

| agent | options | covers |
|---|---|---|
| agforge | `default` (anthropic), `agy`, `agy-claude` (antigravity) | entrance replies, asset planning, generation, callback collection — **not the media model** |
| arxivsage | `default` (anthropic), `agy` (antigravity) | every answer, in any entrance topic |
| agfront | unchanged, pools now derived | its own conversations |
| agautolab | unchanged, pools now derived | entrance, planning, task work, brain-mining |

Both new menus are derived from `agents.toml`, so a name whose profile is
gone is never advertised, and `stub` — the `fake` harness — is on neither.
The sage's committed configuration had *only* `sonnet` and `stub`, which is
exactly the "do not publish fake choices" case the plan named; `agy` was
added and is real on this host.

Forge's selection reaches all four runs a request is made of, and the
`assetrun-` topic it opens carries `[selfnote][exec]` — so a ComfyUI
callback's collecting run continues on what the request was asked for. That
needed no special case: a callback is an ordinary post in the topic, and the
selection is read from home.

## Pools: derived, not declared

Four agents advertised `pool: anthropic` for their default and all four were
right **by coincidence** — each one's roles happened to point at
`claude_code`. The names were already safe; the pool beside a name was a
string in a tuple that nothing compared to a harness.

`agag.execpool` resolves it for every role an option covers
(`AgentSpec.exec_roles`), through the private option→profile mapping,
`agents.toml`, **and this machine's overlay** — which is where a role actually
gets moved and therefore where a declaration goes wrong. What is published is
the resolved value, so the block cannot lie; the declaration survives as an
assertion and every disagreement is logged at startup and before an
introduction is posted.

Three distinctions the shape needed:

- **Mixed is truthful.** `anthropic+antigravity` rather than a rounded single
  name, because an agent whose planning and task work resolve differently
  really does spend two accounts. The `routine_run` guide learned to judge a
  `+` option against every one of their windows.
- **Unavailable is not invalid.** Derivation never checks availability: an
  uninstalled CLI is a runtime failure of one option, never an unpublishable
  contract, and never a reason to take an unrelated conversation down.
- **A degraded derivation says so.** An unreadable config leaves the
  declarations standing and reports *why*, rather than "no mismatch".

Proved on the real instance: forge's overlay was edited to send `generator`
to `agy`, and the published default became `anthropic+antigravity` with the
diagnostic naming the role, the profile and the harness. Restored afterwards.

## Drift resolutions

None of the three was what its row suggested.

- **agfront `service_missing`** — the declared `process_pattern` named
  `agfront.zulip_listener`, the module the listener was entered through
  before `agag_builder` p2 split the entry point. The running process is
  `python -m agfront.listener`, so the pattern matched nothing and never
  would have, however fresh the observation. Refreshing first was still what
  turned a guess into a process id and a pattern side by side.
- **agforge channels** — the expectation named the retired `FreeForge`.
  Subscribing was never available (the channel does not exist) and would have
  been wrong if it were. Replaced with what the introduction advertises and
  the bot is actually in.
- **agecho-agautolab1** — its gap named a channel that does not exist in the
  realm, so "repairing" it meant creating a live agent to turn a row green.
  It had *already* been retired since August; p3's redeploy of the node
  restarted its listener, which re-posted its introduction under the ✔
  topic's freed bare name and opened a **twin**. A retirement quietly undone
  by a deployment.

Registration and process health stayed separate: every liveness line is still
`info`, `autolab-agautolab1` still says `stale` out loud, and nothing was
converted into an error or hidden to get a clean board.

## Four defects found on the way

1. **An archived Zulip channel keeps its `folder_id`** and is not in
   `GET /streams`, so `archive_zulip_folder` saw an empty folder, tried, and
   got a 400 it could not explain. Six orphaned folders had accumulated, one
   per project ever archived. Fixed in `agag.zulip`
   (`channels(include_archived=)`, `clear_channel_folder`) and
   `agautolab.project_archive`; all nine folders retired through the fix.
2. **Retiring an agent had no route off a node.** Removing a placement stops
   the inventory carrying it — and the systemd user unit stays *enabled*, so
   the next boot restarts the listener and un-retires the instance.
   `ansible_agdev/playbooks/agent/retire_agag_agent.yml` is the inverse of
   `setup_agag_agent.yml`, and stops there: it touches neither the bot
   account nor the ✔.
3. **`agentchat resolve` cannot fold a twin.** It answers "already resolved"
   because the ✔ topic exists, and leaves the un-✔ one — which is the one the
   boards read. `resolve_topic` on the twin's own last message merges them.
4. **The pool was never checked** (above).

## Tests and gates

`pyagag` 532 · `nctl` 1335 · `agentroom` 267 · `agautolab` 242 · `agforge`
241 · `agfront` 109 · `arxivsage` 16 (a new suite — it had none) ·
Ansible conformance 4 · `tsc --noEmit` and `npm run build` clean ·
`retire_agag_agent.yml` syntax-check clean.

Browser verification of the retained views on both the dev server and the
rebuilt nginx container: the five-view cycle closes, a detail popup renders,
and `performance.getEntriesByType('resource')` filtered for `/api/` is `[]`.

## Live run evidence

Recorded in full in `report5.md`; the four properties that were missing:

| property | evidence |
|---|---|
| a published option actually runs, and the **written record** says so | sage on `agy`: `profile/harness/provider = agy/agy/antigravity` beside `exec_option: agy, exec_source: topic, exec_message_id: 5974` |
| a reset really returns to the defaults | the same topic's next run: `sonnet/claude_code/anthropic`, `exec_option: None, exec_source: topic, exec_message_id: 5981` |
| forge honours a selection across planning **and** generation | three runs on `agy`, the third `exec_source: inherited` from the plan topic; a real 512×512 badge delivered |
| **an agy-selected autolab task resumes from a callback on the inherited selection** | the continuation woken by a post in *forge's* topic recorded `harness: agy, exec_source: inherited, exec_inherited_from: pj-papers/workplan-ex1-inherit` |

Front's routine decision was exercised on five controlled observations in
**real servings** — below threshold, crossed mid-run, already exceeded at
open, reset, and a failed read with stale numbers. Every entry named the pool
first and said where it got it (the published `agag-exec` block, not a table
of its own), and the stale run reported the condition as *unjudgeable* while
still reaching the routine's goal — the two sentences `refine_routine` p1
separated, keeping "I could not read it" from becoming "it was fine". No real
account was consumed to 70 %; `AGFRONT_BUDGET_URL` was removed and the
listener reloaded.

18 runs, **$1.03**. Nine of them ran on `agy` and cost no dollars — they moved
the Antigravity window instead (5-hour 8.1 % → 8.8 %), which is the pool the
menu advertises for them, observed.

## Test work closed

Through the system's own paths: ✔ on the sage topic, `agforge.retire` for
both asset requests, acceptance → `agautolab.mission_done` → ✔ for the
mission, the throwaway `ex1-budget` routine ✔'d, unsubscribed and archived,
the demonstration's unpushed commit dropped from the `papers` project, and
the temporary plist entry removed. The operation room shows five instances,
three retired, and every row a `done` receipt.

## What is left

- **cagent publishes no execution options** and correctly reads as *unknown*.
  Out of this plan's scope; it is the one agent left that predates the
  contract.
- **Forge's ComfyUI callback collection under an inherited option is fixture
  coverage only.** The live badge went through SwarmUI, which is synchronous,
  so no notifier callback was involved. The plan permits it, and the
  behaviour is not unclear — the collecting run is an ordinary serving of the
  same topic, and the autolab callback above proves the identical
  "read the selection from home" rule live.
- **`.claude/settings.json` carries obsolete allowlist entries** — a
  `POST :8791/mission` route that no longer exists, an `ssh … agautolab/jobs/`
  line from the deleted drive loop, and scratchpad paths from a dead session.
  Harmless (they only ever widen what is permitted, and nothing matches them),
  but it is the developer's own trust configuration and not mine to narrow.
- **`agecho`'s checkout at `/home/eiji/agecho` is deliberately still there.**
  The retirement playbook keeps it unless asked: it holds that instance's
  ignored credentials and its run records, and a stopped unit is reversible
  while a deleted directory is not.

## The through-line

p3 removed a second system that work records lived in. ex1 removed the last
things that *described* a world that no longer existed — a screen whose
backend was a month gone, a stub kept alive for that screen, a channel
expectation naming a retired channel, a discovery pattern naming a renamed
module, a fixture agent that had already been retired twice, and four pools
that were true by luck.

The shape they share is the one worth carrying forward: **a declaration nobody
checks stops being true silently**, and the fix is never to check harder — it
is to derive the fact from the thing that will actually run, and let the
declaration be the assertion that gets tested.
