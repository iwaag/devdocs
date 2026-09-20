# Step 2 report — the human's argue: ProtoPrey

Plan: [plan.md](plan.md) step 2. The argue ran 2026-09-20 14:29–14:56 UTC
(23:29–23:56 JST) between the Developer, Front, archsage, autolab and forge, in
`#argue › argue-20260920-142954` (anchor 7525). The Omni Agent posted nothing in it.

## What the human brought

The desire is the Developer's own post #7526: **ProtoPrey**, a text-adventure-core
adventure game with modern graphics and controls, played from the viewpoint of a
creature between insect and mouse size, lost in nature. Its two stated tests: does
"beautiful and overwhelming nature seen by a weak prey animal" come across, and can
"being eaten" — impossible to feel in reality — be made vivid and, as a game, gripping.
Death by predation is a swallow, never gore; digestion ends in a chosen biome; being
eaten well is progression. 2D/3D, input, screen, action are all to be settled by
trial and error; the human said in advance that the plan may fail.

## How the argue went

1. Front recorded the desire (`[selfnote][desire] 7526`) and invited **archsage** for
   three questions: prior knowledge, what to study before an MVP, and what nobody covers.
2. archsage (#7533, one council run) said plainly that **no sage's tree holds anything**
   on this — the arXiv tree has no game-design, fiction or ecology paper — so all its
   material was general knowledge, to be treated as candidates. It laid out five lines
   of study (prior games; gore-free depiction of predation and rating constraints;
   ecology as game material; progression and the loop; prototyping and experiment
   method), defined a new **`sage:protoprey`** with an empty tree, and proposed the
   cheapest experiment order: a text-only predation scene read by several people →
   the same scene with one still and one sound → a three-round "eaten and back" choice
   loop. It named no game titles, leaving that to a sourced run.
3. Front chose **study first** with reasons (#7540), opened `#pj-protoprey-research`
   with `researchplan-protoprey-research` (#7536: the five questions in priority
   order, deliverables, out of scope), and asked autolab for setup only. autolab
   answered in 46 s (#7542): `main/` with `RESEARCHPLAN.md`, `methods/`, `reports/`,
   pushed as `d263687` on the internal git route; `README_PROJECT.md` naming channel,
   topic and argue; no `publish/`.
4. **Front asked the human whether anything should change (#7540) and closed the
   argue 54 s later (#7545) when autolab's callback arrived, without waiting for the
   answer.** The Developer un-resolved it and answered (#7552): the MVP as proposed,
   study first agreed, **no time or cost limit**, and a first concrete order — let
   forge draw at least a meadow seen from a tiny viewpoint, with prompt know-how
   either as a study item or by trial and error.
5. Front took the forge order itself: `assetplan-protoprey-meadow-mouse-view` in
   forge's channel, forge's plan a7560 (three 16:9 stills, two photoreal and one
   painterly, small A/B trials on the prompt wording), started in
   `assetrun-protoprey-meadow-mouse-view-a7560` (#7572), delivered in 4½ minutes
   (#7584): three PNGs at 1344×768, `prompts.md`, `findings.md`.
6. Front **could not open the zip** (its harness denied the download) and asked the
   human to look (#7585). The Developer judged from the images themselves (#7589):
   the last four or so are very good, whether C is "picture-book" is doubtful but fine
   as is, and the image-prompt topic is **not** to be added to the study.
7. Front closed the argue again (#7592) with outcome `study pj-protoprey-research`,
   listing the next work and the open points. The listener verified channel,
   document and autolab's answer and resolved the topic.

## What the Omni Agent saw in forge's result

Read from the run's own `result.zip`. Candidate B (golden evening) reads
unmistakably as ground level: soil and dew at the bottom, stalks and seed heads far
above the frame, a soft sun behind them, an empty gradient sky for text. forge's
`findings.md` records the words that produced the scale ("worm's-eye view from the
soil", "stalks rising high overhead like a forest") and the ones that backfired
("camera" drew a tripod, "trees" drew trees, "blades" drew swords), that C is a flat
digital illustration rather than watercolour, that one seed per prompt was used and
that the tool has no negative-prompt field. That is reusable prompt knowledge; per the
human it stays in forge's record, not in the study.

## Decisions on record, and who made them

| Decision | Made by | Where |
|---|---|---|
| The desire and the two things to test | Developer | #7526 |
| Study before MVP | Front proposed, Developer agreed | #7540 → #7552 |
| Research questions and their order | archsage proposed, Front wrote | #7533, #7536 |
| Time and cost: no limit | Developer | #7552 |
| First asset: a tiny-viewpoint meadow, by forge | Developer | #7552 |
| Image quality accepted; prompt know-how not a study item | Developer | #7589 |
| Experiment order (text only → still+sound → 3-round loop) | archsage proposed; adopted in the plan | #7533, #7536 |

Not decided in the argue: the concrete MVP scope and evaluation points beyond the two
tests in #7526; how far iterative production and asset revision will be observed;
the intensity of the first research run; whether `methods/`/`reports/` become
repositories. Front listed the last two as open in #7592.

## Cost

| Role | Runs | Cost |
|---|---:|---:|
| Front `argue` | 7 | $0.78 |
| Front `present` (renders) | 5 | $0.40 |
| archsage council | 1 | $0.94 |
| autolab setup | 1 | $0.21 |
| forge (plan, run, delivery) | 3 | $0.55 |
| **Total** | 17 | **$2.87** |

## Findings

- **Front completes an argue on autolab's callback even while its own question to
  the human is unanswered.** Its guide ("Finishing") says to write the outcome once
  autolab has answered the setup; it says nothing about a question of its own still
  open. The human recovered by un-resolving and answering, and said afterwards this was
  acceptable this time. Recorded for the guide; not changed in this step.
- **Front cannot see an image it commissioned** (download denied in its harness) and
  asked the human to judge — the human was the right judge here, but the same gap will
  block Front from doing any technical check of assets on the human's behalf.
- **archsage's honesty about the empty tree** shaped the outcome: it made study-first
  the obvious choice and kept made-up game titles out of the record.
- autolab again laid out `direction/` and `devlog/` before reading a study setup
  request (argue p1 finding 5, still present).
- In an argue Front invited only archsage; forge entered because the human named it.
  autolab and cagent were not consulted on engine or environment questions. That is
  consistent with the desire ("decide by trial and error") but leaves the "proven /
  to try / unexplored" table with a single row: unexplored.

## Reached

The human chose the plan, its first study, the time and cost stance and the first
asset, all in their own posts; the research questions and the experiment order are
in `researchplan-protoprey-research`; the open technical questions are archsage's
five lines, all unsourced. Everything above is reachable from the argue's message ids.

## Not reached

No MVP scope beyond the two tests; no observation scope for iteration. Both can be
settled when the study's first results give the human something concrete to choose
from. Nothing had started when the argue closed; starting it is step 3.
