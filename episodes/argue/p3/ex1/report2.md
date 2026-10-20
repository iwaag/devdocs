# argue p3 ex1 step 2 — shared understanding and Front's decisions

The trial was launched once at 2026-10-20 16:19:37 UTC. Discussion,
contributions, Front's judgement and its dispatch took 7 min 28 s, ending
at 16:27:05. **The Omni Agent posted nothing after the opening message.**
No hint was needed: each step the plan expected happened without
prompting.

## Identity

- Discussion `argue-p3-ex1-trial`, "rtsでjevを取り入れたAI同士の対戦実験（argue
  p3 ex1: guided）". Members at creation: Omni Agent (for Developer), argue,
  front, rts-vs-bot, jevlab.
  - Meeting room: `http://agstudio.local:8093/hangout/?discussion=argue-p3-ex1-trial`.
  - Archive: `http://agstudio.local:3000/developer/pj-argue/src/branch/main/discussions/argue-p3-ex1-trial`.
- Opening message #181 was posted once with `wfe discussion post`. Its token
  is `argue-p3-ex1-brief-1`, its author is `Omni Agent (for Developer)` and
  it has no addressee, so it invited the facilitator. The text is in
  `pj-agdev/.local/argue-p3-ex1/step2/brief.txt`.
  - **Developer-authored, verbatim** (each quote checked with `grep -F`
    against `devdocs@76492e5`):
    - the task goal (p3 braindump);
    - the sentence describing the expected development (jevlab supplies
      literature, rts the game specification, then Front decides on
      further discussion and on proposals, and builds workflows);
    - 「ex1ではfront丸投げではなく、omni agentが人間の代理で指示とヒントを与えながらargueを実行してみる。」
  - **Omni-authored, labelled "not the Developer's words"**: a reading of
    each participant's role for this discussion only. It says that
    Front's choices of whom to hear are acted on by the facilitator citing
    Front's message. It also says that multi-turn work is ordinary project
    workflow, that the earlier p3 match's artifacts may be used,
    questioned or ignored, and that the protocol is one execution and one
    review.
  - Not given: a Jev mechanism, a game interface, parameter values, a task
    graph or a quality bar beyond the goal.

## Chronology

| UTC | # | Who (run) | What |
| --- | --- | --- | --- |
| 16:19:37 | 181 | Omni (for Developer) | Opening message → invitation 31 → `argue/facilitate/run-009` |
| 16:20:18 | 182 | argue (run-009) | Plan in #181's order: invites jevlab (32) and rts-vs-bot (33) first, front after both |
| 16:21:05 | 183 | jevlab (`discuss/run-004`) | Existing knowledge, each item labelled source claim / measured here / inference (below). Asks rts-vs-bot what state and actions it exposes and what a match costs, and asks about the p3 match |
| 16:22:37 | 184 | rts-vs-bot (`discuss/run-005`) | Game facts and interfaces (below). Answers jevlab's two questions. Critiques the p3 match. Offers a `jev_controller.js` skeleton as its ordinary workflow |
| 16:23:08 | 185 | argue | Summarizes the convergence and open points ("my reading, not theirs"); invites front (34) |
| 16:23:57 | 186 | front (`discuss/run-004`) | **Enough to start; no further round.** Ownership, dependency, evidence; asks for "a go" |
| 16:24:36 | 187 | argue | "Acting on front's #186": invites front to dispatch (35) |
| 16:26:24 | 188 | front (`discuss/run-005`) | Dispatched d1 → `rts-vs-bot/improve-game`; jevlab request waits for it |
| 16:27:05 | — | argue | `wfe discussion await` on `front/discuss/run-005` (invitation 36, `awaits: result`); waits |

## Knowledge brought by jevlab (#183)

jevlab chose its own sources and stated their limits:

- **Integration pattern** [source synthesis of 7 game examples, not re-run;
  `study/game/README.md`]: game code builds the legal options, and the
  model only picks one.
- **Local readouts** [measured; `devdocs/episodes/start/p2/report.md`,
  game_test/p2]: warm latency 49–176 ms, which suits 1–10 Hz macro
  decisions and not per-frame control.
- **Baseline discipline** [game_test/p2]: compare against a scripted or
  rule arm, with gates registered in advance.
- **Multi-agent precedent** [set6 p2]: a decision cache over coarse
  views. The rule baseline failed there first, which jevlab took as the
  lesson to validate the scripted bot first.
- **Measured failure modes** (`study/hype_and_reality/local-failures.md`):
  - option order changed 17–27 % of top-1 answers;
  - irrelevant state lines flip answers;
  - confidence does not separate costly errors (AUC 0.73);
  - option keys named in the text steer the answer;
  - Japanese options are weak.

  Its design advice: small, fixed-order option sets with opaque keys,
  English, and legality re-checks in code.
- **Unknown, stated as such**: any Jev-style model in an RTS (none found);
  whether it beats a rule; long-horizon state; two Jev bots interacting.

## Game facts brought by rts-vs-bot (#184)

rts-vs-bot cited `game/` at `d90d517` (headless tests 73/73 run in the
turn) and the p3 run files.

- **The game and its runner**:
  - a fixed-map 1v1 at 20 ticks/s with no fog of war;
  - deterministic: a re-run of the p3 match took 0.09 s with identical
    ticks;
  - the command API is shared by human and bot;
  - controller contract `createController(owner) → {update(g)}`;
  - the stock bot acts every 30 ticks;
  - tunable knobs with their tested ranges;
  - `run_match.js` reusable from p3 `discuss/run-002`.
- **Constraints**:
  - the stock bot builds one barracks, soldiers only, no towers or
    research;
  - one map and determinism, so N matches need N different controller
    pairs or a seeded or sampled controller;
  - a match leaves a result JSON but no per-decision trace.
- **Critique of the p3 match**: Jev made 6 one-shot parameter choices
  before tick 0. It was one deterministic match without a control arm, so
  the win cannot be attributed to Jev.

## How the participants used each other's evidence

- rts-vs-bot answered jevlab's two questions directly. It adopted jevlab's
  vocabulary (≤6 legal opaque-keyed options, code-side legality, logged
  requests) and matched jevlab's 1–10 Hz macro cadence to the bot's
  1.5 s tick.
- Both independently judged the p3 match a plumbing check, not evidence of
  Jev's benefit. Front accepted that (#186).
- Rejected or revised options:
  - p3's static parameter plan was demoted to a plumbing check;
  - per-frame control was ruled out (jevlab: latency; rts-vs-bot: not
    needed);
  - fog-of-war and scouting questions were ruled out (no fog);
  - win-rate claims from one pair were rejected (determinism).
- The proposal moved from jevlab's generic "bot B with one decision slot"
  to a concrete module contract (`options(g, owner)`, pluggable
  `choose(view, options)`, logging hook, scripted fallback) that rts-vs-bot
  says needs no game change.

## Front's decisions (#186, #188), separated from argue's

| Decision | By | Evidence |
| --- | --- | --- |
| Invite specialists first, Front after both | argue (following #181's order) | #182 |
| Information is sufficient; no further discussion round | **front** | #186 "Why enough", citing #183/#184 |
| rts-vs-bot owns the game side via `improve-game`; jevlab owns the model side via `experiment`; (1) before (2) | **front** | #186 items 1–2, "Dependencies" |
| Decision slot and view format left to the specialists ("Both specialists choose; Front does not") | **front** | #186 item 3 |
| Bar: "a match where jev makes in-game decisions, logged trace, scripted control arm"; no win-rate claim | **front** (from specialists' limits) | #186 "Honest limits" |
| Evidence to return: run links, files, decision log, verdict as `--kind result`, judged before "delivered" | **front** | #186, #188 |
| The "go" to dispatch | argue, explicitly "acting on front's stated plan" | #187, invitation 35 |
| Work graph `talk → rts (delegate d1) → jev → present` | **front** (its own `wfe run adjust`) | `front/discuss/run-005` nodes |
| Await Front's run for the results | argue | invitation 36 (`awaits: result`) |

- Front waited for a go instead of dispatching in its first turn. It
  treated its contribution as unable to start work ("my contribution
  starts nobody"), although delegating from its own run needs nobody's
  permission. The facilitator gave the go without adding content, so the
  substance stayed Front's. The extra round cost about 2.5 min.
- One small inconsistency is recorded for later observation. #186 had
  jevlab propose the slot and view format first so that rts-vs-bot would
  build against them. In d1 (#188), Front left the slot choice to
  rts-vs-bot and jevlab without sending jevlab anything first. The
  skeleton has a pluggable `choose`, which may make the order matter
  little.

## Guidance that mattered

- **The opening message only.** It named the roles and the order
  (knowledge → facts → Front's decisions → workflows). The facilitator
  followed that order literally (#182, #185).
- **No intervention after #181.** The ledger has no rows. No hint, no
  reminder of workflow separation and no repair.
- **Automatic recovery**: none. Every turn recorded its node. rts-vs-bot
  `discuss/run-005` started and completed `talk`, unlike p3's run-001 and
  run-004.

## Readiness for step 3

At 16:27:12:

- `rts-vs-bot/improve-game/run-007` (child of d1) is executing on studio;
- `front/discuss/run-005` waits on the delegation, without a slot;
- `argue/facilitate/run-009` waits on the external discussion, awaiting
  Front's result (invitation 36).
