# agent_standardize p8 — selfnotes: the conversation remembers its root

AI-generated (Omni Agent, 2026-08-21). Steps:
[1](report1.md) · [2](report2.md) · [3](report3.md) · [4](report4.md).

## What the phase did

A run that posts in another agent's topic used to record that participation
in a local ledger file. The memory is now **in the chat**, as a hidden post
`agentchat send` writes before its first real message:

```
[selfnote][rootchat] <channel>/<topic>
```

Three things follow from it, and together they close p7's two problems:

1. **A called-back run knows where it came from.** The topic that named the
   agent carries the agent's own root note, so the home to serve is read off
   the conversation. No file survives the end of a run.
2. **A called-back run answers at home.** p7 replied into the topic that
   called, which made every progress report a post in somebody else's
   conversation — and a post in somebody's topic serves them. That reflex is
   gone. Speaking to another agent is now only ever a deliberate
   `agentchat send`.
3. **A selfnote is never somebody speaking.** Every "who spoke last" check in
   the listeners goes through `last_real_sender`, so the note an agent writes
   in another agent's topic buys nobody a run.

agforge's `assetrun-` topic became workrun-like on the same machinery: the
plan opens it and anchors it with a `[rootchat]` note and a `[work]` note, so
a trigger is answered by reading the topic rather than by guessing at a
queue. `works.next_work` and the requester's burden of "one trigger, one
Work" are deleted.

## Verdict

All four success criteria met, first attempt ([report4](report4.md)). One
image, seven agent runs, 96 seconds from the developer's go to silence,
1.10 USD, zero human or Omni posts after the go, and eleven minutes of
silence in all three listener logs.

The answer p7 asked for turned out not to be a rule for ending a
conversation. It was removing the reflex that could not end: **an agent's
reply always goes to its own requester.** Two agents then stop the way an
agent and a human always did — because one side runs out of things to send.

## Commits

| repo | commit | what |
|---|---|---|
| pyagag | `6d5853d` | `agag.selfnote`; the root note in `agentchat send`; `last_real_sender` everywhere; `agag.participation` deleted |
| agforge | `b6f6c8e` | the plan opens and anchors `assetrun-`; `work_by_id`; the chatlog as input; `next_work` deleted |
| agfront | `586cc6f` | home read off the chat; the callback replies at home; `AGENTCHAT_LEDGER` dropped |
| pj-agdev | `715b209` | submodule pointers |
| devdocs | `807d323` | this episode |

565 tests green across the three repositories (349 / 191 / 25).

## Open, and why each is open

### Deferred by the plan

- **autolab on selfnotes.** agautolab still imports `agag.participation` and
  is pinned to the pre-p8 pyagag (`065de12`). It cannot take a pyagag bump
  until `workplan-`/`workrun-` adopt root notes, and its `handle_mention`
  moves off the ledger. Its bot is not subscribed to `agforge-agstudio1` or
  `front`, so it took no part in this proof and nothing it does today is
  wrong — but it is the one place in the system where the two memories
  coexist, and it should not stay there long.
- **Silent mentions.** p7 listed `@_**name**` as a candidate terminator and
  p8 did not need it. Whether it is still wanted is now a real question
  rather than a guess: the proof stopped without it.
- **Two agents with root notes in one topic, both named.** Each agent finds
  *its own* note (`own_rootchat` filters by `sender_id`), so each serves its
  own home. Untested live, and the interesting case is not correctness but
  cost: one post can then start two runs in two different conversations.
- **Selfnote garbage in long topics.** Notes are never deleted and never
  rendered, so they accumulate invisibly. `LAST_SPEAKER_LOOKBACK = 10`
  bounds how many a last-poster check must skip; a topic where eleven
  consecutive selfnotes are the newest messages would read as "nobody has
  spoken". Not reachable today — an agent writes one note per topic — but it
  is a real bound, written down here rather than discovered later.

### Found in the proof

- **Two callbacks, two identical answers.** forge names the trigger in both
  the `assetrun-` report and the `assetplan-` delivery, so Front was served
  twice and told the developer "done" twice (891, 893). Harmless and
  non-looping; costs one extra run per delivery. The fix is to name the
  trigger in one of the two posts, and which one is a genuine design
  question — the delivery is what the requester waited for, the run topic is
  where they last spoke.
- **The startup rootchat sweep has no notion of "already served."**
  `sweep_rootchats` asks whether a topic's last real post names this bot.
  Since a called-back run now answers at *home*, the bot never becomes the
  last poster in the other agent's topic, so that condition stays true
  forever. It did not fire in this proof (no queue re-registration happened
  during it), but a listener restart after a completed exchange would re-serve
  every anchored topic whose last post named it. p7's `sweep_mentions` was
  self-silencing for exactly the reason p8 removed. This wants a marker —
  the served message id, or a note — before the next phase relies on
  recovery.

### Not a p8 question, noted in passing

The delivered icon carries a red rounded-square border nobody asked for.
Asset quality was never one of this phase's criteria and should not be read
into the result; it belongs to the generator's own guide.
