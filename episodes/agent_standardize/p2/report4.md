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
