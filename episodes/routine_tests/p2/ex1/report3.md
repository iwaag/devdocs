# Step 3 — A: an anchor recovered through one replacement relation

Plan step: [plan.md](plan.md) §3. Problem: [problem.md](problem.md) §A.

## The baseline

p2, [../report3.md](../report3.md). Front opened a run, delegated it into
`#pj-studyuspolitics` › `workplan-collect-and-analyze-contributions` and
anchored that topic with its root note, message **6367**. The plan stalled,
autolab retired mission m6371, and the replacement took the freed display
name and wrote `[selfnote][replaces] 6371` (message 6401) into itself.

Autolab then named Front correctly — twice, in two topics — and Front
refused both:

```
mention in 'work-m6400'/'workrun-task1-m6400' carries no root note of ours; ignoring
mention in 'pj-studyuspolitics'/'workplan-collect-and-analyze-contributions' carries no root note of ours; ignoring
```

Neither agent was wrong. `retire_conversation` renames the **whole topic**,
and a rename moves every message in it — including root notes other agents
wrote there. Front's 6367 went into
`✔ retired-workplan-collect-and-analyze-contributions-m6371`; the live topic
under the old name carried no Front anchor at all. Autolab cannot repair it
by copying the note, because a root note is identified by its **sender**: a
note autolab writes is autolab's note.

## The repair

The information was already written. The change is entirely on the reader
side, in shared code, so every agent gains it at once.

**`agag.selfnote`** — the relation becomes a shared convention:

- `REPLACES_TAG`, `replaces_note`, `parse_replaces` (moved out of
  `agautolab.anchor`, which now re-exports them, so the spelling of a
  cross-agent relation does not live in one consumer);
- `replaced_anchor(messages)` — the earliest valid relation, **whoever wrote
  it**. Deliberately not sender-filtered: the relation is written by the
  replacing agent and read by the third parties that were anchored in the
  conversation whose name it took. Filtering it to the reader's own id
  would leave exactly the agent it exists for unable to read it, which is
  the bug.

`own_rootchat` is untouched and stays a pure history selector.

**`agag.zulip`** — the network half:

- `conversation_of(client, message_id)` resolves an id to the conversation
  it is in *now*, topic as it stands (a `✔ ` prefix kept, because that is
  the name it can be read under). Deleted is **absent**.
- `inherited_rootchat(client, history, self_id)` reads the pointer, resolves
  it, and looks for **this bot's own** root note in that conversation. One
  hop, then stop.
- `rootchat_home` asks for the current topic's own anchor first, and only
  falls back to the hop.

**`agag.topics`** — `serve_topic(..., extra_threads=…)` and
`TopicContext.extra_threads`. A callback reached through the relation comes
from a topic carrying no note of ours, so `remotes_for_home` — which reads
the `sender:me search:rootchat` narrow — structurally cannot find it, while
its text is the whole reason the serving is happening.

**`agfront`** — `handle_mention` passes the calling conversation as an extra
thread whether or not a note of Front's names it, and `serve` merges it into
`threads/`. The first callback needs it; from the moment Front posts into
that topic its own note names it and the ordinary lookup takes over.

### The rules, as implemented

| rule | behaviour |
|---|---|
| current topic's own anchor | always wins; no pointer is even read |
| inherited anchor | only when the current topic has none |
| hops | exactly one; a replacement of a replacement is not followed |
| pointer missing or malformed | no anchor |
| target deleted | no anchor — never the topic of the remembered name |
| target holds a **foreign** root note | no anchor |
| target moved to another channel | followed; the id is what is resolved |

## Routing readers, checked

The plan named five: `rootchat_home`, `rootchat_notes`, `remotes_for_home`,
`sweep_rootchats`, and the mention recovery route.

- `rootchat_home` — changed, above.
- `rootchat_notes` / `remotes_for_home` / `sweep_rootchats` — all three read
  **this bot's own** notes out of one Zulip search narrow. They are
  unchanged, deliberately: a replacement conversation contains no note of
  ours to find, so there is nothing for them to select differently. What
  they cannot supply is supplied instead by the mention route
  (`extra_threads`), which is the one reader that knows which topic called.
- **mention recovery** — `sweep_mentions` is `is:mentioned`, not a note
  search, so a mention in a replacement topic that arrived while the
  listener was down is recovered at startup and then goes through the same
  `rootchat_home`. Both routes therefore reach the repair: the event path
  (`mention_from_event` → `handle_mention`) and the queue-registration
  recovery path.
- `sweep_rootchats` still cannot see a replacement topic, and that is
  correct rather than a hole: it answers "which topics I anchored are
  waiting on me", and this one is not a topic Front anchored. Its recovery
  twin `sweep_mentions` covers it, and the served mark
  (`[selfnote][served]`) written into home stops the exchange being replayed
  on every restart — asserted.

## Validation

Behavioural checks, each verified to fail against the old behaviour.

`pj-agdev/agfront/tests/test_replacement.py` — twelve checks driving the
whole mention route with **two agent identities** over the realm p2 left:

```
against the previous code (pyagag eb9a0bb + agfront b41beb3):
  5 failed, 7 passed
  FAILED test_the_mention_serves_the_run_that_was_anchored_in_the_retired_plan
  FAILED test_the_replacement_s_own_words_are_readable_in_that_serving
  FAILED test_the_callback_is_marked_served_so_a_restart_does_not_replay_it
  FAILED test_a_second_replacement_hop_is_not_followed
  FAILED test_a_predecessor_that_was_moved_to_another_channel_is_still_found
  stderr: mention in 'pj-studyuspolitics'/'workplan-collect-and-analyze-
          contributions' carries no root note of ours; ignoring
with the fix:
  12 passed
```

That stderr line is p2's log line verbatim — the old code does ignore the
same mention, which is the half the plan asked to be asserted as well.

The seven that pass either way are the ones asserting that **nothing**
happens (foreign root note, deleted target, malformed relation): they pass
against the old code because it did nothing at all, and they are here to
pin that the new lookup has not made any of them reachable.

`pyagag/tests/test_zulip.py` — ten checks on the lookup itself: the hop, the
relation read whoever wrote it, precedence of the current topic's anchor, a
moved target, a deleted target, a malformed pointer, a target with no note
of ours, one hop only, and the cost check — an ordinary callback still
spends exactly one history read and no `message` call, which matters because
every mention in the realm goes through this lookup.

`pyagag/tests/test_selfnote.py` — six checks on the convention: the format,
what is not a note of this kind, that it is read whoever wrote it, and that
the earliest valid relation wins.

| package | result |
|---|---|
| `pyagag` | 561 passed (541 after step 2) |
| `agfront` | 145 passed (133 after step 2) |
| `agautolab` | 242 passed |
| `agforge` | 241 passed |
| `arxivsage` | 16 passed |
| `cagent` | 198 passed |

## Deliberately not done

- **No note is copied or forged.** A root note somebody else wrote would
  make the record lie about who is party to what.
- **Retirement still releases the name.** That is the point of retiring, and
  a replacement created before the rename would merge into the conversation
  it replaces.
- **A mention from an unanchored task with no replacement relation is still
  ignored.** It is outside this repair and remains so.

## Revisions

| | |
|---|---|
| `pyagag` | `db89b4c` |
| `pj-agdev/agfront` | `d5f9382` |
| `pj-agdev/agautolab` | `06a9182` |

## Assistance

None. No live service was touched and no agent was run; the live exercise of
retirement and replacement belongs to the final step.
