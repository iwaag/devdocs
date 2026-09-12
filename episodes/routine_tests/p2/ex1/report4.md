# Step 4 — B2: a deliberate anchor correction

Plan step: [plan.md](plan.md) §4. Problem: [problem.md](problem.md) §B2.

## The baseline

p2, [../report5.md](../report5.md). The `publish` run's delegation topic was
anchored to the Front Desk (message **6482**). The repair attempted by hand
was a second ordinary root note naming the run, message **6500** — and the
completion callback still resolved to the Desk.

That is `own_rootchat` working as designed: *"The **earliest** of our own
root notes wins. A topic is anchored once, by the run that opened it; a
later note would be a repeat."* The rule is right — a repeat must not be
able to redirect a live conversation — and it left no way at all to say a
topic was anchored **wrongly**. p1 needed the same capability four times,
for a different cause, and worked around it each time.

## The convention

```
[selfnote][rootchat-moved] <channel>/<topic>
```

and **one effective-anchor rule**, `agag.selfnote.effective_rootchat`:

> the newest valid explicit move written by **this agent** wins; otherwise
> its earliest ordinary root note wins.

Newest, because a correction is not identity: an agent that gets it wrong
twice must be able to say so twice. Sender-filtered, because only an agent's
own move moves its own anchor — the exact opposite of the `replaces`
relation of step 3, and for the opposite reason: that one is written *for*
other agents, this one is written *by* the agent it binds.

A later ordinary repeat still loses, so nothing about the default changed.

## Where the rule is asked

One rule, every reader, so a correction lands everywhere at once:

| reader | effect of a correction |
|---|---|
| `rootchat_home` (current topic) | the callback serves the corrected home |
| `inherited_rootchat` (step 3's hop) | a correction in the retired conversation is inherited by its replacement |
| `rootchat_notes` | the topic is listed under its new home |
| `remotes_for_home` | the delegate becomes a thread of the run, and stops being one of the Desk |
| `sweep_rootchats` | restart recovery attributes the callback to the run |
| `served_marks` / `note_served` | the mark is written into the new home, so an answered callback is not replayed |

`rootchat_notes` now reads **two** Zulip search narrows —
`sender:me search:rootchat` and a new `sender:me search:rootchat-moved` —
rather than filtering one. Whether a full-text search for `rootchat` also
matches `rootchat-moved` depends on how the server tokenizes a hyphen, and a
routing lookup is not the place to depend on that. One extra call per
recovery sweep.

`own_rootchat` itself is untouched and stays the pure "earliest ordinary
note" selector.

## The command

```
agentchat anchor <channel> <topic> [--home <channel>/<topic>]
```

Writes the move as the authenticated agent, naming the conversation this run
is serving (`AGENTCHAT_HOME`) unless `--home` says otherwise. It refuses a
resolved topic — a post under the bare name of a resolved conversation opens
an empty twin beside it, and a correction written into a twin corrects
nothing — and refuses a conversation anchored to itself.

Its use is explained in `agentchat --help`, which is this tool's
documentation: what the automatic anchoring already does, why saying it
again does not help, what the correction does instead, and that it is for
when the anchor is known to be wrong rather than a habit. Ordinary `send`
still anchors automatically, unchanged.

The move is a selfnote: hidden from every chatlog, every `threads/` file and
`agentchat read`, its author included, and never counted as somebody
speaking — so writing one buys nobody a run.

No guide was changed. The capability is a tool with its documentation
attached; telling three role guides about it before anybody has needed it
would be guidance without evidence.

## Validation

Behavioural checks, verified against the old behaviour.

`pj-agdev/agfront/tests/test_anchor_correction.py` — twelve checks driving
p2's own sequence through Front's mention route:

```
against the previous shared code (pyagag db89b4c):
  6 failed, 6 passed
with the fix:
  12 passed
```

The sequence itself: Desk anchor → ordinary run anchor repeat → explicit
correction → another ordinary repeat. Only the correction moves the home,
and after it the callback is served by the `routine_run` role in the run
topic. Also covered: a second correction supersedes the first; a move
written by another agent moves nothing; two malformed moves move nothing;
the delegate becomes a thread of the run and the Desk's serving no longer
places one; the served mark goes to the new home; the correction is invisible
in the rendered thread; a corrected callback to an **already resolved** run
is not reopened and opens no bare-name twin — its origin is told and given
the delivered note, exactly as before; and a replacement plus a correction
resolve together, the correction living in the retired conversation and
reached through step 3's hop.

`pyagag/tests/test_selfnote.py` — eight checks on the rule itself, including
the full four-step sequence and the hidden-from-the-conversation property.

`pyagag/tests/test_zulip.py` — eight checks on the readers: the callback
lookup, that without a correction the repeat still loses, a foreign
correction, the note search and `remotes_for_home` under the new home, the
recovery sweep, an already-served corrected callback not swept again, and a
correction inside an inherited conversation.

`pyagag/tests/test_chat.py` — seven checks on the command: what it writes,
`--home`, the two refusals, that ordinary `send` is unchanged, and that the
usage document explains when **not** to use it.

| package | result |
|---|---|
| `pyagag` | 588 passed (561 after step 3) |
| `agfront` | 157 passed (145 after step 3) |
| `agautolab` | 242 passed |
| `agforge` | 241 passed |
| `arxivsage` | 16 passed |
| `cagent` | 198 passed |

One fixture change travelled to the consumers: the fake clients in
`agfront` and `agautolab` gained `own_moved_notes`, answering with nothing
unless a test says otherwise.

## Deliberately not done

- **Not "latest wins".** That would reintroduce exactly the misdirection
  "earliest wins" exists to prevent, which the problem statement names.
- **No guard, no lock, no automatic rewrite.** B3 stays unimplemented until
  a measured run shows B1's guidance failing.

## Revisions

| | |
|---|---|
| `pyagag` | `ed65b4e` |
| `pj-agdev/agfront` | `fee20d3` |
| `pj-agdev/agautolab` | `ffd754c` |

## Assistance

None. No live service was touched and no agent was run.
