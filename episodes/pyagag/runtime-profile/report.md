# runtime-profile — final report

**The braindump's ask:** let a request like *"run `routine-study-realworld`
using agy until its usage exceeds 70 %"* reach the agent that will actually do
the work, without the requester having to know that agent's configuration.

**What was delivered:** `ag.exec-options.v1` — a public name for a way of
executing, discovered by reading, selected by posting, frozen per serving,
inherited by child conversations, and recorded beside what actually ran. It
is implemented in the shared skeleton, published by agfront and agautolab, and
was exercised end-to-end in the live realm on 2026-09-09.

## The contract, in one page

`pyagag/docs/exec-options-v1.md` is the whole of it. Its nine decisions:

1. **Two names, not one.** An *execution option* is public — a name, the
   **usage pool** it consumes, the **work** it covers. A *profile* is the
   private `agents.toml` name it maps to. The mapping is the recipient's, and
   one-to-one is a valid start (both agents use it). No consumer's code
   contains a recipient's profile name.
2. **Discovery rides the introduction.** A fenced `agag-exec` block beside the
   existing roster block, generated from the running instance so it cannot
   drift, and deliberately a menu rather than a dump of `agents.toml`.
3. **A missing block is *unknown*, never "no".** The roster block's rule, for
   the roster block's reason. `supported: no` is a different, published
   answer.
4. **Selection is a topic-local mention command**, `@**<them>** use <option>`,
   recognized only when the message is nothing else — so discussing the
   command, or quoting it in a fence, cannot fire it.
5. **A configuration-only post is not work.** Applied, answered with one
   deterministic line, no model. A line rather than a reaction, because here
   the answering agent *is* the topic's owner and a reaction would leave the
   poster as the last speaker forever.
6. **Frozen at each serving's start**, from the topic history as it stood
   then. A command posted mid-run lands on the next serving.
7. **Precedence is one rule**: the newest directive in this conversation
   (command post or inherited snapshot, by message id), else the agent's
   existing defaults. `use default` resolves to *no selection*, so it falls
   through — no third mode.
8. **Inheritance is a snapshot**, `[selfnote][exec] <option> from
   <channel>/<topic>#<id>`, written into a child before its visible
   description. Later parent changes reach new children only; a child
   overrides by carrying its own command, which is newer.
9. **A refusal is visible and never a downgrade.** An unpublished name is
   refused in a post that names the poster and states the standing selection,
   and never becomes the topic's setting. Availability failures stay where
   they were — `E_UNAVAILABLE` at execution time.

**The topic is the store.** There is no selection file anywhere: a restart
re-derives the same answer, and "why did this run on agy" is answered by
reading the conversation and the run record.

## Where it lives

| repo | what |
|---|---|
| pyagag | `agag/execopt.py` (the vocabulary), `serve_topic`'s configuration-only path and frozen `context.selection`, `AgentSpec.exec_options`/`exec_profile`, `run_role(selection=…)`, the introduction block, `agentchat options` / `agentchat use`, the record fields |
| agautolab | its menu derived from `agents.toml`; the selection through the entrance, `superdirector`, `supercoder` and `director`; the `[selfnote][exec]` snapshot at child-topic creation; introduction prose |
| agfront | its own menu (about *its own* conversations); the selection through `front`, `character_talk` and `routine_run`; `budget.pool_for` and pool-labelled observations; the front and routine_run guides; introduction prose |

## Commits

| step | commits |
|---|---|
| 1 | pyagag `0dbc3e9` |
| 2 | pyagag `eb74b91` |
| 3 | pyagag `7e7de55`, `325f4aa`, `5c277e5`; agautolab `61bebe3`, `d20fc09` |
| 4 | agfront `40c48dd`; agautolab `f197832`; pyagag `201d189` |
| 5 | pyagag `e951079`; lock updates in agautolab and agfront |
| 6 | this report, `devdocs/README_DEV.md`, `devpolicy/agent_records.md` |

Reports: `report1.md` … `report5.md` beside this file.

## Test results

| repo | before | after |
|---|---|---|
| pyagag | 470 | **530** |
| agautolab | 219 | **234** |
| agfront | 91 | **103** |

No test launches a paid harness. The configuration-only path is proved to
reach no handler at all.

## Live evidence (2026-09-09, agstudio)

Front was asked, in words, to have autolab do a small piece of work "using
agy", and to **find out** what autolab publishes rather than assume a name.
It read `agentchat options`, reported *"autolab publishes an option literally
named `agy` … That's the real name; I'm not guessing"*, selected it in the
workplan topic (message **5558**), posted the request separately, and reported
back when autolab answered.

The run records:

| run | role | profile / harness | exec fields |
|---|---|---|---|
| `entrance_front/run-0026` | entrance | agy / agy | `option agy`, `source topic`, `message 5539` |
| `superdirector/run-0167` | **planning** | agy / agy | `option agy`, `source topic`, `message 5558` |
| `supercoder/run-0232` | **task work** | agy / agy | `option agy`, `source **inherited**`, `message 5558`, `from pj-ghtrends/workplan-devlog-note` |
| `entrance_front/run-0027` | entrance | **sonnet / claude_code** | `source default`, no option |

That settles the plan's central worry — an option that covered only the
entrance's answer would have been a menu that lies — and its isolation
requirement: the same agent, the same minute, two topics, two backends.
`exec_message_id: 5558` and Front's own report of "message id 5558" agree
without either reading the other.

Also live: the configuration-only post (applied in the same second, no ack, no
run), the refusal of `opus` (naming the poster, still no run, the topic still
on `agy`), and `agentchat options` correctly reporting agforge and arxivsage —
which predate the contract — as **unknown**.

Fixture rather than harness evidence, stated as such: the five controlled
budget observations, and autolab's *callback* continuation (the deliberately
tiny task delegated to nobody; the mechanism is unit-tested, and Front's own
callback continuation was live).

## What this cost, and what it did not

Obeying the contract adds **no Zulip call per serving**: the topic is read
before the ack instead of after it, and that read is the serving's chatlog.
The one call the menu did add — `whoami`, because the command is addressed by
the Zulip full name a mention matches — was removed again by memoizing
`whoami` per client (pyagag `5c277e5`), which is a small net saving across the
fleet.

## The one defect, and where it came from

autolab's first real agy run was correct in every visible way and its record
on disk carried **no `exec_option` at all**. `write_run_record` copies an
allowlist; every step-2 test asserted on the dict `run_role` *returns*, which
did carry the fields, and nothing asserted on the file — which is what the
plan says to inspect and what the cost gauge reads.

Fixed in pyagag `e951079` with a test that reads the file back, and recorded
as a rule in `devpolicy/agent_records.md`: **a recorded field is only recorded
if it reaches the file.** A green suite did not find this; one live record
did.

## Differences from the plan

- **The acknowledgment is a post, not the notifier's reaction.** The plan
  offered the reaction as a reference; it is right for the notifier, which is
  not the topic's owner, and wrong here, where a reaction would leave the
  poster as the last speaker and the topic would match every sweep forever.
- **Front publishes options of its own**, which the plan did not ask for. The
  developer can also ask Front itself to run differently, and publishing makes
  the two requests — "run *your* conversations on agy" and "have autolab run
  *its* work on agy" — legible rather than implicit.
- **A pool name is defined**, as the provider whose account the harness spends
  (`agag.agent_config.HARNESS_PROVIDER`). The plan asked to "match the
  observation to the execution pool" without saying how; deriving it from the
  table the configuration already validates against beats a mapping invented
  in Front.
- **`whoami` memoization** is an unplanned change, made because the contract
  would otherwise have cost one extra Zulip call per serving across the fleet.
- **autolab needs no per-role mapping today** — one profile serves every role
  — so `AgentSpec.exec_profile` is unset there. The seam exists and is tested
  in pyagag for the day a role has to diverge.

## Remaining limitations

- **A live callback continuation of an autolab task was not exercised.** The
  demonstration task delegated to nobody. What is live-proven is the adjacent
  property (the same topic served twice keeps its selection) and Front's own
  callback; the task-topic case is fixture-tested only.
- **No live run has been judged against a controlled budget observation.**
  Pointing a running listener at a fixture needs a plist edit and a full
  `bootout`/`bootstrap` (`kickstart -k` does not re-read the plist). The
  renderer was exercised on all five fixtures through agfront's own CLI
  instead.
- **The pool convention is a convention, not a check.** Nothing verifies that
  an agent's published `pool` matches the provider its profile resolves to; a
  wrong pool would be a threshold judged against the wrong window. A
  validation at publish time is the obvious next step.
- **`agforge` and `arxivsage` publish nothing**, so a request to run *their*
  work a particular way is `unknown`. That is correct behaviour, not a gap to
  hide — but it is a gap.
- **agautolab1 (the VM node) was not updated.** Only agstudio's instances
  carry the contract.
- **`"project": null`** in autolab's superdirector/supercoder records is a
  pre-existing quirk unrelated to this episode; noticed while reading records,
  not fixed here.

*Deus Ex Machina note: the Omni Agent drove the live verification — posting
the commands, starting the task, closing Work G-21. Handoff candidate:
exercising a newly shipped contract in the realm is a routine an agent could
run.*

## step6 record

Documentation and close-out:

- `devdocs/README_DEV.md` — a new section, *How an agent is asked to run a
  particular way*, and a line under *Agent ≠ Model*.
- `devpolicy/agent_records.md` (`4fa3a19`) — a run records what was asked for
  beside the backend, and the rule the live defect taught: **a recorded field
  is only recorded if it reaches the file.**
- `pj-agdev/.local/devenv.md` (ignored) — the machine-specific half: restart
  and re-post after a menu change, check for a harness child first, that
  adding a profile to `agents.toml` is not enough to publish it, which pools
  this host's `/budget` actually answers for, how to place a controlled
  observation, and not to post a courtesy line into a `workplan-` topic.
- The braindump, which had never been committed, is now in the episode folder.
- Superproject `pj-agdev` `832d62c` bumps agautolab `3d8f73b` and agfront
  `2e06644`.

Every affected repository is committed and pushed: pyagag, agautolab, agfront,
pj-agdev, devdocs, devpolicy.
