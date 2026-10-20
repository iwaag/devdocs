# argue p3 ex1 — guided discussion and structured execution: final report

One discussion, `argue-p3-ex1-trial`, ran the p3 RTS/Jev goal with the Omni
Agent acting for the Developer. It went from knowledge and game facts,
through Front's decisions and delegated project workflows, to a review in
the same place, and then to a preserved summary. The trial ran 16:19:37–17:00:43 UTC on
2026-10-20 (41 min).

- **Omni messages**: three. Opening #181, then #192 and #199.
- **System changes**: none.
- **Automatic recoveries**: two.
- **Omni cleanup**: one cancellation of an orphaned wait.

Step reports: [report1](report1.md) (readiness, coordination boundary),
[report2](report2.md) (discussion and Front's decisions),
[report3](report3.md) (work graph, execution, return),
[report4](report4.md) (review, disposition, termination, archive).

## Task, protocol, participants, revisions

- **Goal** (Developer, verbatim, p3 braindump): 「rtsでjevを取り入れいたAI同士を戦わせる実験を１回行う。最初の実験なので品質や問わない、とにかくjevを少しでも取り入れてAI同士の対戦が成立したと言えるならOK。」
- **Expectation** (Developer, verbatim, ex1 braindump): 「…pj-jevlabが既存の文献知識を提供、pj-rtsがゲームの仕様を説明して場に十分な情報が提供されたところでfrontがその後追加議論をするかどうか、誰にどんな提案をさせるか、といった意思決定をして、ワークフローを組んで進めていくという展開だった。」 and 「ex1ではfront丸投げではなく、omni agentが人間の代理で指示とヒントを与えながらargueを実行してみる。」
- **Protocol** (Omni's reading, labelled as such in #181):
  - roles for this discussion only;
  - the facilitator acts on Front's stated choices;
  - multi-turn work is ordinary project workflow;
  - one execution, then one review that re-plans nothing;
  - later Omni posts are hints, not Developer requirements.
- **Participants**:
  - argue (`argue/facilitate`, argue-agent);
  - front (`front/discuss`, front-agent);
  - rts-vs-bot (`rts-vs-bot/discuss`, rts-vs-bot-agent; its `improve-game`
    runs as executor autolab);
  - jevlab (`jevlab/discuss` and `experiment`, jevlab-agent).

  All used profile sonnet.
- **Revisions** (unchanged from p3):
  - WFE agdevworld `978a5ed` (store v6), agautolab `797b2d0`, pj-agdev pins
    `9554a6d`;
  - projects at start: pj-argue `29db1ce`, pj-front `d638a06`,
    pj-rts-vs-bot `d90d517`, pj-jevlab `2f44d31`;
  - quotes checked against devdocs `76492e5`.

## Timeline

| UTC | Stage | Messages |
| --- | --- | --- |
| 16:19:37 | Opening (Omni) | #181 |
| 16:20–16:22 | Knowledge (jevlab) and game facts (rts-vs-bot), each answering the other | #182–#184 |
| 16:23–16:26 | Front judges sufficiency, assigns ownership and dependencies; facilitator gives the go; Front dispatches | #185–#188 |
| 16:26–16:49 | Delegated work: `rts-vs-bot/improve-game/run-007` (4 min), then `jevlab/experiment/run-002` (16 min) | #189 (progress) |
| 16:49:57 | Result back in the discussion (awaited invitation 36) | #190, synthesis #191 |
| 16:50:51 | **Omni hint 1**: no follow-up; the review is next | #192 |
| 16:51–16:55 | Review by the three owners and synthesis | #193–#198 |
| 16:55:59 | **Omni hint 2**: keep a summary; state artifact disposition | #199 |
| 16:56–17:00 | Two `synthesize` runs, sha256 read, owners' disposition | #200–#207 |
| 17:00:43 | Closed by Omni; archive `6213c6c` | — |

## Findings

| Question | Answer and evidence |
| --- | --- |
| Did existing knowledge inform the discussion? | **Yes.** jevlab cited `study/game`, game_test/p2, set6 p2 and `local-failures.md`, labelling each item as a source claim, a local measurement or an inference (#183). The final design used its advice: small fixed-order option sets with opaque keys, code-side legality, a scripted control and pre-registered criteria (#184, #186, run-002 `plan.md`) |
| Did game facts constrain the proposal? | **Yes.** rts-vs-bot's facts (#184) set the shape: the controller contract, the 1.5 s bot tick, no fog, and determinism, which means N matches need N controller pairs. They ruled out per-frame control, fog questions and single-pair win-rate claims. Its baselines then exposed the seat outcome that undercut p3's "win" |
| Did Front coordinate? | **Yes, substantively.** Front alone judged the information sufficient, chose no further round, assigned ownership (game side / model side), set the dependency, left the slot and view choices to the specialists, defined the bar and the evidence (#186), built its own graph `rts → jev → present` and judged each child before going on (#188–#190). The facilitator only sequenced the invitations and gave the "go" Front asked for, explicitly "acting on front's #186" |
| Was work structurally separated from discussion? | **Yes.** No specialist did work in a discussion turn (contrast p3). There were two delegations from Front's run, waits that released the slot, child results that continued Front, and records, reports and integration for each run |
| Did the same discussion review the results? | **Yes, after hint 1.** The owners checked their claims against files and corrected their own and Front's overstatements (#194–#197). Front narrowed its "satisfied" to plumbing. The synthesis kept the open questions (#198). All left goal satisfaction to the Developer |
| How much help was required? | Two process messages after the opening; see *Assistance* |
| What persisted? | Archive and summary in pj-argue `6213c6c` (`summary.md` rev 2). The controller, baselines, experiment folder and reports are integrated on each project's `main`. **The specialists' stated disposition ("not on main") was wrong** (report4) |

**AI-versus-AI and Jev, as evidenced.** Six full matches were played with
a local `qwen3.8:27b-mxfp8` option-letter reader. It made one in-game
choice per bot tick through rts-vs-bot's guarded controller: 993 logged
decisions and 0 fallbacks, and all four pre-registered criteria were met.
That is more Jev involvement than p3 (six static parameters before tick
0). It shows nothing about Jev's effect on play: the reader lost all
seat-1 matches, as random play did. It is still not TypeSafe's Jev.
Whether the goal is met is left to the Developer, as every agent said.

## Assistance (all of it)

| When | What | Class | Effect | Stayed agent-produced |
| --- | --- | --- | --- | --- |
| 16:19:37 | #181 opening: quotes, roles, protocol | Instruction (the trial condition) | The order knowledge → facts → Front → workflows was followed literally | Sources, facts, design, graph, criteria |
| 16:29:57 | `agops/recover/run-029` continued run-007 (attempt ended with `verify` ready) | **Automatic recovery** (Observer) | run-007 completed | — |
| 16:50:51 | #192 answering #191's question to the Developer: no follow-up, the review is next, "how to run it is the facilitator's and front's choice" | Answer + process hint | 44 s later the facilitator opened the review with its own questions | The review questions and all review content |
| 16:52:00 | `agops/recover/run-030` resumed Front's run-005 (`present` left ready) | **Automatic recovery** | Completed, with no duplicate post | — |
| 16:55:59 | #199: keep a source-linked summary; owners state their disposition; sha256 is in-scope reading | Question + process hint (plan step 4.4) | Two `synthesize` runs, the sha256 read by the owners, three dispositions | Where and how the summary is kept, who checked, each answer |
| 17:00:43–53 | Closed the discussion; cancelled `argue/facilitate/run-009` (orphaned wait) | Operational cleanup | — | — |

- No technical content, parameter, mechanism or task graph came from Omni.
- No direct instruction was needed, and no hint was ineffective or
  repeated.
- No system defect was repaired, because none blocked the trial.
- Claims about independent agent reasoning hold for:
  - the discussion and Front's coordination;
  - the execution;
  - the review content.

  They do not hold for:
  - the *decision to hold a review*, which came from the protocol through
    hint 1;
  - the *decision to preserve a summary and state dispositions*, which
    came from hint 2.

## Comparison with p3 (descriptive, not causal)

| | p3 (no intervention) | ex1 (guided) |
| --- | --- | --- |
| Knowledge and facts before decisions | One round, quickly to direction | Explicit, source-labelled, cross-answered |
| Who decided the work | Facilitator (Front: ordering hint only) | Front (facilitator: sequencing and "go") |
| Where work ran | Inside discussion turns, no project records | Delegated `improve-game` and `experiment` runs with plan, reports and integration |
| Awaited-result path | Not exercised | Exercised; the completed-facilitator branch also fired (run-012) |
| Jev in play | 6 static parameters before tick 0 | One in-game choice per bot tick, 993 logged |
| Review | Reached unprompted | Reached after hint 1, more file-based |
| Persistence | Transcript only | Summary (2 synthesize runs), dispositions |
| Duration | 11 min 36 s | 41 min |

ex1 differs in the opening message, adds two hints, and let the agents
reuse p3's artifacts (rts-vs-bot reused `run_match.js`; everyone
re-examined the p3 match). It cannot show that guidance caused the
difference, or that the agents would do this autonomously.

## Limitations and observations (not new work)

- **Front asked permission it did not need**, twice: "a go" before
  dispatching (#186) and before any follow-up (#190). Its own delegations
  needed nobody. The facilitator supplied the go without adding content.
- **The facilitator did not start the protocol's review by itself.** At
  #191 it asked the Developer to choose between stopping and a follow-up,
  and had to be pointed back to #181's protocol.
- **Attempts ending with ready nodes** happened twice (an autolab-executed
  `improve-game`, front-agent's `present`). It is the same family as p3's
  unrecorded rts-vs-bot turns. Automatic recovery handled both within
  about 1.5 min.
- **A facilitator turn that asks the person a question and then waits is
  orphaned.** The answer starts a new turn by design, so the old one
  waits forever (run-009, cancelled by Omni).
- **Wrong "not on main" claims in the final disposition** were not caught
  by any reviewer. The executor had already integrated both specialist
  runs. The agents most likely read a stale local `main`, since the
  coordinator workspaces do not follow `origin/main` by themselves; Omni
  fast-forwarded them after the trial.
- **Link and wording defects**:
  - #189's run-view link lacked `/wfe/`;
  - #190 had a truncated line;
  - the result JSONs hash the wrapper instead of the controller (owned
    by the agents in #196 and #202).

## Future improvement ideas (separate from this trial)

- A facilitator guide fact about asking the person: the question and the
  answer belong to a new turn, so the asking turn can complete instead of
  waiting.
- A way for a discussion run to read a project's published `main`, so that
  disposition answers do not rest on a stale workspace.
- A fact for Front and the facilitator that a participant's own
  delegations need no "go", so the extra round is unnecessary.
- Have `run_match.js` hash the controller module itself.

## Records

- Archive:
  `http://agstudio.local:3000/developer/pj-argue/src/commit/6213c6ca9175/discussions/argue-p3-ex1-trial`
  (summary: `…/src/branch/main/discussions/argue-p3-ex1-trial/summary.md`).
- Meeting room:
  `http://agstudio.local:8093/hangout/?discussion=argue-p3-ex1-trial`.
- Work:
  - `http://agstudio.local:8093/wfe/#/ws/front/run/discuss/run-005`
  - `…/#/ws/rts-vs-bot/run/improve-game/run-007`
  - `…/#/ws/jevlab/run/experiment/run-002`
- LAN check from agpc: all passed (report4).
- Ignored evidence: `pj-agdev/.local/argue-p3-ex1/`
  - `notes.md`, `ledger.md`;
  - `step2/brief.txt`, `step2/watch.log`;
  - `step3/hint1.txt`, `step3/hint2.txt`;
  - `step4-lan-agpc.txt`.

Deus Ex Machina notes:
- I cancelled `argue/facilitate/run-009` at the trial's end, a record for
  argue-agent.
- I fast-forwarded the coordinator workspaces of rts-vs-bot, jevlab, front
  and argue after the trial.
