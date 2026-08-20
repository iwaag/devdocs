# p2 step 4 — proving it

AI-generated (Omni Agent, 2026-08-20).

## Outcome

**One agent recognized another from its introduction and used it.** Front,
asked in a `front-*` topic for an image, read the harvested `#agents` board,
found agforge's entrance there, and opened a request topic in
`#agforge-agstudio1` under its own identity. Forge acknowledged it and
planned the work.

Two live conversations were run. The first proved the mechanism and
uncovered a leak; the second is the clean proof.

## Run 1 — the mechanism, and a leak

`#front > front-20260820-title-image` (message 659): a wish for a 16:9
illustration-style title image of a lighthouse at dusk. Front replied
(message 663) that it had asked agforge, and
`#agforge-agstudio1 > create-title-image-1` (message 661) held its request;
forge acknowledged (662) and planned it (664, 665).

The topic Front picked — `create-title-image-1` — was **verbatim from the
`agentchat --help` examples**, which at that point also named
`agforge-agstudio1`. That is exactly the attribution the phase is about: the
routing had a second possible source, and the demonstration would have been
worth much less with it in place. The examples were rewritten to abstract
placeholders (`<their-channel>`, `<topic>`) with one sentence saying the
names are whatever the addressed agent's introduction gave, pyagag was
pushed, agfront re-locked and the listener reloaded. A test
(`test_help_names_no_real_agent_channel_or_topic`) now keeps real routing out
of that help text.

pyagag `b1c7b6c`; agfront `66379e5`.

## Run 2 — the clean proof

`#front > front-20260820-boss-art` (message 666), a wish for a boss
character's standing art — a deep-sea jellyfish-like giant, portrait
orientation.

1. Front replied (message 671) naming `agforge-agstudio1` as the image agent
   and what it had sent.
2. **`#agforge-agstudio1 > create-boss-jellyfish`**, message 668, posted by
   **Front**, with an elaborated request (translucent glowing bell, trailing
   tentacles, transparent or flat background).
3. Forge acknowledged in 669, wrote its requirement notes and toolset
   selection in 670, and created the Plane ticket F2-13 with a plan in 672.

The topic name `create-boss-jellyfish` is Front's own — it kept agforge's
`create-` prefix, which the introduction states, and named the rest after the
conversation. Nothing in agfront or in the tool's help could have supplied
it.

Links (`https://agstudio.local:8543/#narrow/channel/…/near/<id>`):
`front` is stream 24, `agforge-agstudio1` is stream 34.

- Developer's wish — 24 / `front-20260820-boss-art` / near 666
- Front's report back — 24 / `front-20260820-boss-art` / near 671
- Front's request to forge — 34 / `create-boss-jellyfish` / near 668
- Forge's acknowledgement — 34 / `create-boss-jellyfish` / near 669
- Forge's plan — 34 / `create-boss-jellyfish` / near 672

`runcreate-` and the generated image were not required and were not run;
create is the scheduling half, and that is where p2 stops.

## Multi-turn, and reading back

A third Developer post (message 673) asked Front to confirm before sending
next time, and to go check the progress. `serve_topic` re-served the topic as
generation 2, and Front (message 675) reported the state of
`create-boss-jellyfish` accurately — the requirement notes, the F2-13
ticket, the plan, and that no image exists yet.

That was not designed for: it means `agentchat read` gives Front a **return
path** it did not have under the p1 command-file route, where "nothing comes
back" was a property of the design. Front pulled the answer itself, from a
channel it is not subscribed to.

## Success criteria

1. **A `front-*` conversation → proposal → permission → `create-` topic →
   report back.** Partially met, and honestly so. Everything holds except
   the permission step: Front did **not** ask before acting, in either run.
   It went straight from the wish to the request and reported afterwards,
   although its guide says to suggest the way first and act only "if the
   developer permit you to do it". Told about it in run 2 it agreed
   immediately ("了解、次回から送る前に確認するようにします"), which is
   conversation, not a fix — the next run reads the same guide.
2. **agforge acknowledges the request in that topic.** Met — message 669,
   plus the plan in 672.
3. **Attributability.** Met:

   ```
   $ grep -rn "agforge-agstudio1" pj-agdev/agfront/src pj-agdev/agfront/agent
   grep: no match
   ```

   Pinned as a test too (`test_agfront_knows_no_other_agent_s_channel`),
   over `src/**/*.py` and `agent/guides/**/*.md`. Since run 1 the same rule
   is enforced on `agentchat --help`.

## Failure Farming: the missing permission step

The one real finding of this step, and worth keeping rather than patching in
a hurry. The guide's flow is: read `tools/`, propose, wait for permission,
then act. Front collapsed it. Plausible causes, none verified:

- The wishes ended in 「作れる？」/「できる？」 — literally "can you?" — which
  a model may read as already authorizing the asking.
- The guide states the permission step as a conditional sentence in a
  paragraph, not as a step Front must be able to point at.
- Nothing in the run makes the permission observable; asking and acting cost
  the same and only acting produces visible progress.

Not fixed here, deliberately: p2's instruction was to check the
implementation against the guide, not to rewrite the guide. This is
Evidence-Driven material for the next phase — the first evidence about how
this guide actually behaves under a real request.

Consequence worth noting: Front acts on the Developer's behalf in one turn,
under its own bot identity. In this experimental LAN realm that is
acceptable; it would not be where a request costs something irreversible.

## Deus Ex Machina note

The Developer's three posts were written and sent by the Omni Agent using
`.local/zulip/developer.env`, with the developer's explicit permission in
this session. The wishes are the Omni Agent's words, not the Developer's —
handoff candidate: a human-written wish is a better test of the same path.

Their delivery was itself the first non-test use of `agentchat`
(`agentchat send front …`), which incidentally exercised the CLI as a third
identity.

## Run 3 — the permission step, after the guide was revised

Added 2026-08-20, after the Developer rewrote the guide in response to the
finding above. Two lines changed (agfront `233f6cd`):

- "suggest the way to make it done" → "suggest the way to make it done
  **before actually doing it**, like `"It's possible. I'll talk with agent-A
  to make it done. Can I proceed?"`"
- "If the developer permit you to do it" → "If the developer permit you to
  **proceed your plan**"

So: the ordering was made explicit, and an example of the proposal was given.
Nothing else — no code change, and the guide is read from disk per run, so no
reload was needed.

`#front > front-20260820-bgm`, a wish for a loopable quiet-cave BGM for
stage 1 (message 676).

1. **Front proposed and stopped** (message 678):

   > It's possible — I can ask **agforge-agstudio1** (the media generation
   > agent) to create a loopable BGM track for Stage 1 with a quiet cave
   > atmosphere. Want me to go ahead and send that request?

   It named forge's entrance from the harvest, and asked. Verified that it
   really had not acted: `agentchat topics agforge-agstudio1` listed no BGM
   topic at that point — the proposal was a proposal, not a report.
2. The Developer answered「うん、お願い。」(message 679).
3. **Front then acted** — `#agforge-agstudio1 > create-stage1-bgm`, message
   681 — and reported back in message 683.
4. Forge acknowledged (682) and selected `toolset-music` (684, 685).

Links: `front` is stream 24, `agforge-agstudio1` is stream 34.

- Wish — 24 / `front-20260820-bgm` / near 676
- **Front's proposal, before acting** — 24 / `front-20260820-bgm` / near 678
- Permission — 24 / `front-20260820-bgm` / near 679
- Front's request — 34 / `create-stage1-bgm` / near 681
- Forge's acknowledgement — 34 / `create-stage1-bgm` / near 682

**Success criterion 1 is now met in full**, including the permission step
that runs 1 and 2 skipped.

The topic name `create-stage1-bgm` is again Front's own composition on
forge's `create-` prefix, and the channel again came from the harvest — the
attributability grep is unchanged.

## What the third run says about the finding

The diagnosis in "Failure Farming" above was partly right and cheaper to fix
than expected. Two of the three guesses are supported:

- **The ordering was not stated as an ordering.** "suggest the way to make it
  done" describes an output, not a step that precedes another; "before
  actually doing it" makes it a sequence.
- **The permission step was not observable to Front either.** The example
  sentence gives it a concrete shape to produce, so producing it is now the
  obvious move rather than an inference.

The remaining guess — that 「できる？」 reads as authorization — is neither
confirmed nor refuted: run 3's wish also ended in 「できる？」 and Front asked
anyway, which suggests the phrasing was not the deciding factor.

Two guide lines, no code, one paid run to verify. That is what
Evidence-Driven Guidance costs when the evidence is collected first —
Failure Farming worked exactly as intended here: run 1 and 2 were allowed to
fail informatively rather than being pre-empted by an anxious rule.
