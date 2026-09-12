# Step 5 — validation, rollout, and the live sequence

Plan section: [plan.md](plan.md), *Validation, rollout, and completion*.

## Environment, before anything was deployed

Read through Nautobot/nctl rather than assumed
(`pj-clusterintent/nctl/README.md`):

```
✓ nautobot   version 3.1.3, authenticated, intent_catalog + intent_graphql
✓ worker     celery workers: 1, pending jobs: 0
✓ dumps      8 hosts, the five live ones collected ~4.6 h ago
✓ submodule  ansible_agdev / nauto / nctl / nintent / nodeutils all clean
ok: True
```

Those describe **recorded infrastructure state**, not which Python revision a
listener is running; the revision was verified separately, below. Nothing
host-specific is in this file — the environment memos stay in `.local/`.

## Rollout, in the plan's order

1. **Shared code first.** `pyagag` → GitHub `main`, `a7bc4a5 → ed65b4e`
   (three commits: C, A, B2), and `8c07226` afterwards for the README.
2. **Consumer locks** (`uv lock --upgrade-package pyagag`), then `uv sync`,
   then that package's own suite — against the **installed** dependency, not
   a `PYTHONPATH` shim:

   | consumer | lock before | lock now | suite |
   |---|---|---|---|
   | `agfront` | `a7bc4a5` | `ed65b4e` | 157 passed |
   | `agforge` | `5036ee9` | `ed65b4e` | 241 passed |
   | `agautolab` | `5036ee9` | `ed65b4e` | 242 passed |
   | `arxivsage` | `5036ee9` | `ed65b4e` | 16 passed |
   | `cagent` | `0d26145` | `ed65b4e` | 198 passed |
   | `agdevworld/agentroom` | `0d26145` | `ed65b4e` | 267 passed |
   | `pyagag` itself | — | — | 588 passed |

   **Not upgraded, deliberately**: `agecho` (retired in `refactor` p3 ex1)
   and `comfynotify` (`0a33830`) — the notifier reads mentions and writes a
   ticket, holds no root note and routes nothing, so a shared-routing bump
   buys it nothing and costs a restart of a daemon whose command memory is a
   file.

3. **Import path and revision verified in each service environment**, by
   asking the installed package rather than the lock file:

   ```
   agfront / agforge / agautolab / arxivsage / cagent / agentroom
     agag/topics.py  conversation_context ✓  MOVED_TAG ✓  inherited_rootchat ✓
   ```

4. **Listeners restarted** (code is held in the process; guides are not):
   `com.agdev.agfront-zulip`, `com.agdev.agautolab-zulip`,
   `com.agdev.agforge-zulip`, `com.agdev.arxivsage-zulip`,
   `com.agdev.agentroom`, `com.clusterintent.cagent-api`,
   `com.clusterintent.cagent-zulip`. Each came back and logged its full
   sweep; Front's was `0 awaiting, 1 mentioning, 78 calls spent`.

5. **Consumers and superproject pointers committed and pushed**: `agfront`
   `842d7b1`, `agforge` `1462899`, `agautolab` `b5259cd`, `agdevworld`
   `809a809`, `arxivsage` `50007a9`, `pj-clusterintent` `e068234`,
   `pj-agdev` `417bdb2` (which also catches up the agfront pointer left
   uncommitted after p2).

**`agautolab1` (the VM) was not redeployed.** Its checkout is `b38a6af` with
the pyagag it was deployed with — self-consistent, and it runs the gateway,
not a listener, so nothing there routes a callback. Redeploying it is the
ordinary Ansible run and is outstanding.

## The live sequence

Through the ordinary entrance, on the deployed code, as the Developer. A
small routine `anchorcheck` was created for it: `#routine-anchorcheck`
(channel folder `routine`, Developer and Front subscribed), whose guide asks
for **exactly one delegation** — one question to the arXiv sage in a topic of
its own channel — and then the end of the run. Cheap, and it exercises the
whole lifecycle.

### B1 — the re-measurement

Three fresh requests at `#front`, worded differently, no hint about
anchoring:

| | request | run opened | delegations **by the opening serving** | the run's own delegation | its anchor names |
|---|---|---|---|---|---|
| A | 6528 | `routinerun-20260912T1811Z` | **0** | `entrance-anchorcheck` (6535) | the **run** (note 6534) |
| B | 6547 | `routinerun-20260913T1815Z` | **0** | `entrance-anchorcheck-tree` (6554) | the **run** (note 6553) |
| C | 6566 | `routinerun-20260913T1900Z` | **0** | `entrance-anchorcheck` (6572) | see below |

**Zero opening-serving delegations out of three**, against p2's one out of
one. Front's reply in run A even states the causal link back:

> That's the whole of my work here — the run will do its own delegating to
> the sage from its next serving, and its report will land back in this
> conversation when it ends.

Callback destinations, from the listener log:

```
mention in 'arxivsage-agstudio1'/'entrance-anchorcheck' serves routine-anchorcheck/routinerun-20260912T1811Z
mention in 'arxivsage-agstudio1'/'entrance-anchorcheck-tree' serves routine-anchorcheck/routinerun-20260913T1815Z
```

— the `routine_run` role, in the run topic, which is the plan's success
condition. **B3 is therefore not implemented**, as the plan and p1's policy
both direct.

Complete lifecycle, all three runs: callback evidence in `threads/`, entries
written per serving, **exactly one** `ag-routinerun` block each (grepped),
report delivered into the requesting conversation, delivered note written,
requester served once by `continue_deliveries`, run topic resolved.

### C — the intermittent empty answer

Sixteen Front servings across five fresh conversations. **Six of them ran in
one turn with no tool calls** — the exact shape that produced p2's *"I don't
see a message or request from the developer yet"* — and every one of them
answered the conversation correctly:

| record | turns | cost | what it answered |
|---|---|---|---|
| run-0606 | 1 | $0.0367 | the routine already reported here; nothing outstanding |
| run-0608 | 1 | $0.0353 | same, for request B |
| run-0610 | 1 | $0.0384 | same, for a late answer after run A ended |
| run-0611 | 1 | $0.0377 | same, for request C |
| run-0612 | 1 | $0.0339 | quoted the developer's request back and asked to confirm |
| run-0614 | 1 | $0.0348 | reported the post it had already made, by message id |

Zero empty answers in 16 servings. p2's rate was 2 of the first 2 requests.

### A — a replacement callback

Built from two agent identities on the deployed listeners, reproducing p2's
sequence exactly:

1. Front, asked at its ordinary entrance, posted into `#ops-testbed` ›
   `plantest-anchorlive` — a **real** `agentchat send`, so a real root note
   (6595) naming `front/front-anchorlive-20260912T1817Z`.
2. The **autolab bot** registered the plan there (`[selfnote][mission]`,
   message 6598).
3. The **retirement rename**, the same call `retire_conversation` makes —
   one PATCH with `propagate_mode: change_all` to
   `✔ retired-plantest-anchorlive-m6598`. Read back by id:

   ```
   mission 6598              now in ops-testbed / ✔ retired-plantest-anchorlive-m6598
   Front's root note 6595    now in ops-testbed / ✔ retired-plantest-anchorlive-m6598
   ```

   That is the defect's whole mechanism, live: the rename moved another
   agent's anchor.
4. The **replacement** took the freed name and named what it replaced:
   `[selfnote][mission]` 6600, `[selfnote][replaces] 6598` (6601), a visible
   post (6602). Front had written **nothing** in it.
5. The autolab bot named Front there (6603).

Front's listener:

```
serving mention in 'ops-testbed'/'plantest-anchorlive'
mention in 'ops-testbed'/'plantest-anchorlive' serves front/front-anchorlive-20260912T1817Z
marked 'ops-testbed'/'plantest-anchorlive' served up to 6603 in front/front-anchorlive-20260912T1817Z
```

p2's log line for the same situation was *"carries no root note of ours;
ignoring"*. Both threads were placed in the serving's workspace — the live
replacement (through `extra_threads`) and the retired conversation (through
the root note that moved with it) — and the replacement's text was readable
in it:

```
threads/ops-testbed/plantest-anchorlive.md
threads/ops-testbed/retired-plantest-anchorlive-m6598.md
```

### B2 — a mis-anchored delegation, repaired through the new command

This one was not staged. Run C, choosing a topic name for its delegation,
reused `entrance-anchorcheck` — which run A had already opened and anchored.
A topic is anchored once, so run C's own note was never written and the
sage's answer (6575) was routed to run A, which had already finished:

```
mention in 'arxivsage-agstudio1'/'entrance-anchorcheck' belongs to
  routine-anchorcheck/routinerun-20260912T1811Z, which is finished; not reopening it
told front/front-anchorcheck-a-20260912T1811Z that ... was answered after it ended
```

Resolved-home handling worked as p1 built it: nothing reopened, no bare-name
twin, the origin told and given the delivered note. But run C was stranded —
which is precisely the state B2 exists for.

The Developer posted one message into run C's topic (6579) saying the
delegation was anchored to a finished earlier run and that
`agentchat anchor` could correct it. **Front ran the command itself**:

```
[selfnote][rootchat] routine-anchorcheck/routinerun-20260912T1811Z        (run A, 6534)
[selfnote][rootchat-moved] routine-anchorcheck/routinerun-20260913T1900Z  (run C, 6581)
```

and then read the topic, recorded the sage's answer with its message id, and
ended the run with one finish block whose `reason` names the correction. Its
entry:

> Anchor correction: posted `agentchat anchor arxivsage-agstudio1
> entrance-anchorcheck` (note 6581), pointing that topic's answers to this
> run … the topic had been anchored by an earlier, already-finished run,
> which is why the sage's reply first went there instead of here.

An agent given the tool and its documentation used it correctly on the first
attempt, from one sentence of context.

## Assistance, recorded separately

Everything an agent did on its own is above. These are the interventions:

1. **Created the `anchorcheck` routine** — a channel, its `guide` post, and
   Front's subscription. A test fixture, not a repair.
2. **Posted the three requests and one follow-up confirmation** at Front's
   entrance. Ordinary use of the entrance; Front asked to confirm before
   posting on the Developer's behalf, which is its guide working.
3. **Told run C its delegation was mis-anchored and that `agentchat anchor`
   exists** (message 6579). Front did the repair; the pointer was the
   intervention, and it is the one that should not be needed twice — see the
   limits below.
4. **Built the retirement and replacement by hand** with the autolab bot's
   credential rather than by running an autolab mission: the mission note,
   the `change_all` rename and the replacement's notes. The *reader* side —
   the thing under test — was the deployed Front listener, untouched.

> **Deus Ex Machina note:** *performed a mission retirement and replacement
> for agent autolab, by hand, to exercise Front's recovery — handoff
> candidate.*

## Cost

| | runs | cost |
|---|---|---|
| Front (`front` ×11, `routine_run` ×5) | 16 | $1.1333 |
| arXiv sage (`front` ×3) | 3 | $0.1679 |
| **live sequence total** | **19** | **$1.3012** |

All `claude_code` on `anthropic/claude-sonnet-5`, `exec_source: default`.

## Limits this did not remove

- **C carries the chatlog, not the threads.** Run-0614 answered in one turn
  with no tool calls; it saw the conversation and answered it correctly, and
  it did **not** relay autolab's question from the replacement topic —
  because `threads/` is still a file a run has to decide to open. The
  routing was right and the file was there; the same one-turn shape now bites
  one level out. This is a candidate for the next episode, not a defect of
  this repair.
- **A run can still choose a delegation topic name another run already
  anchored.** Run C did, unprompted. B2 makes that repairable — and it was
  repaired — but nothing yet makes it unlikely. The `routine_run` guide
  already says "open a **new** topic for each delegation"; the evidence says
  the wording is not enough when the name is one the routine's own guide
  suggests. Measured once, left to accumulate rather than guarded now.
- **One intervention was still needed** for the whole sequence (item 3
  above), and it was needed for the case just named.
- **`agautolab1` is not redeployed**, and no autolab mission was run
  end-to-end on this code.

## Revisions tested and deployed

| | |
|---|---|
| `pyagag` | `ed65b4e` (deployed), `8c07226` (README, after) |
| `pj-agdev/agfront` | `842d7b1` |
| `pj-agdev/agforge` | `1462899` |
| `pj-agdev/agautolab` | `b5259cd` (agstudio only) |
| `pj-agdev/agdevworld` | `809a809` |
| `arxivsage` | `50007a9` |
| `pj-clusterintent` | `e068234` |
| `pj-agdev` | `417bdb2` |

## Tidying

The `anchorcheck` routine is **retired** — a ✔ on its `guide` topic, which is
the realm's way of saying so — and its three runs, the four `front-`
conversations, the two sage topics and the `#ops-testbed` test topic are all
resolved. Un-✔ the guide and the fixture is back; it costs nothing while it
sits there and it is the cheapest way to re-measure B1 after a guide change.
