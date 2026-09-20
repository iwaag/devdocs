# Adventure game p1 — phase report

Source: [braindump.md](braindump.md). Plan: [plan.md](plan.md). Steps:
[1](report1.md) environment and round trips · [2](report2.md) the human's argue ·
[3](report3.md) study and hand-over · [4](report4.md) the first build. Executed
2026-09-20 13:00–16:50 UTC (22:00–01:50 JST) by the Omni Agent with the Developer.

**Ended by the Developer after step 4.** The plan's steps 5 (play evaluation → next
argue → revision → re-evaluation) and 6 (cycle assessment) were not run; the Developer
chose to close p1 with the first build delivered and unplayed. This report records that
as the fact it is, as the plan asks: the production cycle was demonstrated from desire
to a playable build once, not around a revision loop.

## What happened, in one line each

1. The environment was converged (46 targets), and Front, autolab, cagent and the
   Arguing Room answered marked test posts on the explicit_reply pins ($0.65).
2. The Developer brought **ProtoPrey** to `#argue › argue-20260920-142954`; archsage said
   plainly its trees held nothing and laid out five lines of study; Front chose study
   first; the Developer agreed, set no time or cost limit, and had forge draw a
   tiny-viewpoint meadow, judged good from the images themselves ($2.87).
3. Research round 1 ran as mission m7601 in `#pj-protoprey-research`: eleven gore-free
   depiction techniques with sources, the rating envelope, a smallest-experiment method
   and three experiment proposals, in 20 minutes ($8.41); `sage:protoprey` now reads
   the study.
4. The Developer said the premise needs no scientific basis and to build; Front's
   `front` role was given `agproject` (the one basis change), opened `#pj-protoprey`,
   and autolab's mission m7732 delivered a Godot build **v0.1.0** — three text variants,
   one still-and-sound scene, a three-cycle eaten-and-return loop with a biome choice —
   verified headless by autolab and again on a fresh clone, windowed, by the Omni Agent
   ($9.50).

## Proven in this phase

- **A human can run an argue with the agents and come out with a plan they chose**:
  desire, study-first, budget stance, first asset and its acceptance are all the
  Developer's own posts, traceable by message id.
- **The route from argue → study → research mission → project → build mission → tagged
  build holds together on the ordinary channels**, with Front driving task sequences
  without the Omni Agent once a request exists.
- **Study reaches production**: the technique catalogue from round 1 is what the
  three text variants vary; the rating envelope is in the task text and the README.
- **forge is in the loop** through its `assetplan` route from both an argue and a
  mission, and says no honestly when it cannot (sound).
- **The human judged assets from the images**, not from relayed numbers, twice.

## Not proven

- Whether the build is any good to play: nobody has played it, heard its sound or
  seen its layout. The plan's core question — can "being eaten" be vivid and gripping
  — is untested.
- Iteration: no evaluation, no revision, no re-evaluation, no continuity check on a
  second version.
- Posting from the Project Room (reading verified; no real post was made there).
- Human-led closure of an argue from the room on this code (the Developer un-resolved
  by hand and Front resolved twice).

## Cost

| Step | Runs | Cost |
|---|---:|---:|
| 1 round trips | 10 | $0.65 |
| 2 argue | 17 | $2.87 |
| 3 research m7601 (+ hand-over) | 17 | $8.41 |
| 4 build m7732 (+ setup) | 68 | $9.50 |
| **Total** | 112 | **$21.43** |

Wall time from the Developer's first post to the tagged build: 2 h 17 min, of which
the Developer's waits for the Omni Agent's relays and the basis fix were about 40 min.

## Where the cycle stopped, and the decisions lost or relayed

- Front closed the argue 54 s after asking the Developer a question (guide gap); the
  Developer recovered it by un-resolving.
- Front's close-out named the next work but no one asked for it; the Omni Agent relayed
  the Developer's recorded decision into `#front` (stand-in: the requester).
- The Developer's "build the MVP" was said to the Omni Agent, not in Zulip; relayed and
  marked as such (#7712). The GOAL cites the relay.
- The meadow still the Developer liked (a7560 B) never reached the project and forge
  could not re-serve it; the build carries a remake (a8007).
- The Omni Agent chose "no placeholder, ask forge" when task 4 stopped (stand-in:
  requester), and attached the study repository to `sage:protoprey` (owner's step).

## Basis and guide changes made

| Repository | Commit | Change |
|---|---|---|
| agfront | `4538f9e` | `front` and `desk` roles carry `Write` + `Bash(agproject:*)`; guide paragraph for a decision the developer stated; origin lines say "conversation" outside an argue; 181 tests |
| pj-agdev | `247533c` | submodule pin |
| devdocs | this episode | reports 1–4, this report |

Not committed: `archsage/sages/protoprey/` (holds an internal URL; the sage stays a
local, untracked definition as `worldtrend` does).

## Candidates for the next basis work (observed, not done)

1. Front's argue guide: do not write `complete` while a question of Front's own to the
   human is unanswered; and, when the human has authorized a start, open the first
   `workplan-` from the close-out instead of naming it as next work.
2. forge: keep results by request id and re-serve them on request; give the generator
   role a way to run the scripts it writes, or tell it to call `agforge` directly;
   an audio toolset beyond music if synthesised sound turns out not to serve.
3. A route that moves an asset the human accepted in an argue into the project that
   later needs it.
4. The doubled leading mention in Front's and autolab's replies (consumer guides or
   `agag.reply`).
5. Mission acceptance and closure when the delivery goes to a requester's conversation
   rather than into the mission topic (m7732 is 5/5 tasks done and still open).
6. autolab lays out `direction/` and `devlog/` before reading a setup request (third
   observation).

## What exists for whoever continues

- `#pj-protoprey` (goal #7718), `autodev/protoprey` at `v0.1.0` (`205037d`): README
  with start command and controls, `VERIFY.md`, `ASSETS.md`, `report.md`.
- `#pj-protoprey-research`, `autodev/protoprey-research` at `89c5240`: four sourced
  documents and twenty open issues; research items 3–5 unstarted by decision.
- `sage:protoprey` synced to that tree.
- The evaluation the plan's step 5 wants is still the natural next act: play `v0.1.0`
  and say, in a new argue naming it, what to keep and what to change.
