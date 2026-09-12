# routine_tests p2 ex1 — report

Three defects p2 demonstrated and left open ([problem.md](problem.md)),
repaired in the order the plan set — **B1 → C → A → B2** — and then exercised
live at the ordinary entrance.

Step reports: [1](report1.md) · [2](report2.md) · [3](report3.md) ·
[4](report4.md) · [5](report5.md).

## Before and after

| | p2 | now |
|---|---|---|
| **B** — opening servings that delegated for the run they had just opened | 1 of 1 (`publish`) | **0 of 3** |
| where the plan/completion callbacks of such a run were served | the Front Desk, twice; the run recorded nothing | the run, by `routine_run`, in the run topic |
| **C** — new conversations answered as if empty | 2 of the first 2 requests | **0 of 16 servings** across 5 fresh conversations, 6 of them one turn with no tool calls |
| **A** — a mention from a conversation that replaced the one Front was anchored in | `carries no root note of ours; ignoring`, twice | `serves front/front-anchorlive-…`, with the replacement's text readable in the serving |
| **B2** — correcting a mis-anchored delegation | impossible; a second ordinary note changed nothing | Front ran `agentchat anchor` itself and the run recovered and finished |
| interventions needed | 7 across the trial | 1 during the live sequence (plus fixture setup) |

## What was done

**B1 — guidance, no code.** One paragraph in
`agfront/agent/guides/front/guide.md`, beside the run-opening instruction,
saying that opening the run is the whole of that reply and explaining *why*:
anything sent from a serving is anchored to the conversation being served, so
its answer comes back there and the run never hears it; a delegation the run
makes is anchored to the run. The opening-post contents and the continuation
flow are unchanged. Re-measured with a small routine that actually delegates:
zero opening-serving delegations in three runs. **B3's automatic rewrite
guard is not implemented**, because the guidance did not fail.

**C — the conversation is in the prompt.** `agag.topics.conversation_context`
carries the rendered chatlog — the same bytes the workspace file gets, so one
snapshot and one set of filters — into the prompt, whole when it fits and
bounded with a stated omission when it does not, with an oversized single
post cut visibly and an empty conversation saying so. Wired into Front's three
roles, the shared own-channel entrance (`agag.entrance`) and forge's plan
front. Proved at the harness boundary with a `fake` harness that reads its
stdin and opens no file at all.

**A — one replacement hop.** Retiring a plan renames its whole topic, which
moves every message in it including other agents' root notes, and the
replacement then takes the freed name. The reader side now follows the
`[selfnote][replaces] <message id>` relation the replacing agent already
writes — read **whoever wrote it**, resolved by id, one hop, then stop — and
looks for this agent's own note in the conversation that id is in now. The
topic that called is placed in `threads/` whether or not a note of ours names
it, because in the inherited case nothing else can discover it.

**B2 — a deliberate correction.** `[selfnote][rootchat-moved] <channel>/<topic>`
and one effective-anchor rule every routing reader asks: *the newest valid
explicit move written by this agent wins; otherwise its earliest ordinary
root note wins.* A bare repeat still never redirects a topic. `agentchat
anchor` writes it, and `agentchat --help` says when to use it and when not
to. Because one rule reads it everywhere, a corrected delegate also moves in
`threads/`, in recovery, and in the served mark.

## Validation

Every new behavioural check was run against the corresponding old behaviour
and recorded as failing there:

| check | against the old code | with the fix |
|---|---|---|
| C — `agfront/tests/test_prompt_delivery.py` | 9 failed | 9 passed |
| A — `agfront/tests/test_replacement.py` | 5 failed, 7 passed | 12 passed |
| B2 — `agfront/tests/test_anchor_correction.py` | 6 failed, 6 passed | 12 passed |

The checks that pass either way are the ones asserting that *nothing*
happens — a foreign root note, a deleted target, a malformed relation — and
they are there to pin that the new lookups did not make any of those
reachable.

Suites, each run in its own package against the intended pyagag revision:
`pyagag` 588, `agfront` 157, `agforge` 241, `agautolab` 242, `arxivsage` 16,
`cagent` 198, `agentroom` 267.

## Deployed

`pyagag` `ed65b4e` on GitHub; every consumer's lock moved to it, synced,
validated against the **installed** dependency, and its listener restarted;
the revision and import path verified by asking the installed package in each
service environment rather than reading a lock file.

| | |
|---|---|
| `pyagag` | `ed65b4e` (+ `8c07226`, README) |
| `agfront` | `842d7b1` |
| `agforge` | `1462899` |
| `agautolab` | `b5259cd` (agstudio) |
| `agdevworld` | `809a809` |
| `arxivsage` | `50007a9` |
| `pj-clusterintent` | `e068234` |
| `pj-agdev` | `417bdb2` |

Not upgraded, deliberately: `agecho` (retired) and `comfynotify` (routes
nothing). Not redeployed: the `agautolab1` VM, which runs a gateway and no
listener and is self-consistent on its own older revision.

## The live sequence

A small `anchorcheck` routine was created as the fixture — one delegation to
the arXiv sage, then the end of the run — and three fresh requests were made
at `#front`. All three runs completed the whole lifecycle: callback evidence
in `threads/`, an entry per serving, exactly one `ag-routinerun` block, the
report delivered into the requesting conversation, the delivered note, one
continuation serving, and the run topic resolved.

The replacement callback was built with two agent identities on the deployed
listeners — a real Front delegation, the autolab bot's mission note, the
`change_all` retirement rename (which moved Front's note, read back by id to
prove it), and a replacement naming what it replaced. Front recovered the
anchor and got both conversations as threads.

The anchor correction was **not staged**: run C reused a delegation topic name
run A had already anchored, its callback went to the finished run A, and
after one sentence from the Developer naming the command, Front corrected the
anchor itself and finished the run.

19 agent runs, **$1.3012**. One intervention beyond fixture setup.

## Remaining limits

- **C repaired the chatlog, not the threads.** The last live serving answered
  in one turn with no tool calls — it saw the conversation and answered it
  correctly, and it did not relay the question sitting in `threads/`, because
  a thread is still a file a run has to decide to open. The same one-turn
  shape, one level out. The obvious next question, and not a defect of this
  repair.
- **Nothing stops a run choosing a delegation topic name another run already
  anchored.** Run C did it unprompted. B2 makes it repairable and it was
  repaired; making it unlikely is a separate, unmeasured question, and the
  `routine_run` guide already says to open a new topic each time. Left to
  accumulate evidence rather than guarded now.
- **One intervention was still needed**, for exactly that case.
- **`sweep_rootchats` still cannot see a replacement topic** — correctly, it
  answers "which topics I anchored are waiting on me" — and `sweep_mentions`
  is what recovers those after a restart. A mention older than the mention
  window is still not recoverable, as before.
- **A mention from an unanchored task with no replacement relation is still
  ignored**, which the plan puts outside this repair.
- **No autolab mission was run end to end** on this code, and `agautolab1` is
  not redeployed.

## Documentation

`pyagag/README.md` gains three sections — the conversation in the prompt, the
retirement-rename problem with its one-hop lookup, and the effective-anchor
rule with its command. `devdocs/README_DEV.md` states plainly, where the
routing conventions live, that **retiring a plan renames its conversation and
moves every agent's notes**, and the two lookups that follow from it.
