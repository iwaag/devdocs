# Workflow editor p7/ex1 — step 7: the research request with an agent

**Isolated trial, not a human-reviewed run.** All three runs ran in an isolated area: a scratch registry, catalog, exec store and workspace clones whose origins were local bare copies, so publication could reach neither Gitea nor the live workspace. The deployed release `6058c0f` and agautolab `f1c466d` ran them. The agent was the real `wfrun` role (claude_code), launched by `agautolab.wfexec --once`. The one plan answer in trial A was recorded as **"Omni Agent (isolated-trial stand-in for iwaag)"**, not as the person. `improve-game/run-004` and the live area were not touched. No new live RTS run was requested: that needs the person's own request and answers.

## Setup

- **Game state:** `pj-rts-vs-bot` at `9f5a5a1` (run-003 accepted), the state run-004 started from.
- **Catalog:** the two seeded presets, plus *Preliminary research*. The person has not designed a research template of their own, so the trial used the example exactly as the User nodes page offers it. Its expectation reads: "When I ask for research or investigation before implementing, I expect a study node before the work it informs, with its sources and findings in its own report."
- **Runs:**

| Trial | Workflow | Request |
| --- | --- | --- |
| A | `improve-game` as deployed | run-004's braindump, verbatim |
| B | `improve-game` with an existing `survey` study node before `implement` (a trial-only commit) | run-004's braindump |
| C | `improve-game` as deployed | 「兵士の移動速度を少しだけ上げてください。一般的なRTSの歩兵の速度感を軽く参考にして、上げすぎないようにしてください。」 |

## Compared with run-004's execution information

run-004's brief had no catalog, and the guide said nothing about requested research. Each trial attempt's prompt contained:

- the catalog statement ("The person recognizes these activities as distinct steps…");
- the three templates with their expectations;
- the line naming `wfe nodes instantiate … --run … --file` and `wfe run adjust --source request`;
- the guide's new sections on expected steps and research evidence.

Each attempt recorded `settings.catalog` with the run's snapshot hash.

## Outcomes

| | Plan and graph | Work and evidence |
| --- | --- | --- |
| **A** | Before starting `_plan`, the agent applied `r2` with source `request braindump.md` (reason: "braindump asks to research RTS formation movement basics on the web before implementing"), adding `preliminary-research → implement`. All work was still pending, so this was a valid boundary. `plan.md` lists `preliminary-research` first and says its sources, findings, choice, alternatives and uncertainties go to `report-preliminary-research.md`. The node keeps its template provenance | After the stand-in agreement: `preliminary-research` → its report → `implement` (the history confirms the order). The research report has **7 linked sources**, findings attributed to them, the chosen approach (grid offsets) and why, three rejected alternatives with reasons, and two open uncertainties. `report-implement.md` cites the research report for its approach. The agent then verified, presented (p1 served) and asked the review question (q2) |
| **B** | No adjustment and no second research node. `plan.md` assigns the research to the existing `survey` node and says the sources and choice go to `report-survey.md` | Stopped after planning, by design (only the design choice was under test) |
| **C** | No research node. `plan.md` has a "ノードについて" section: the request asks only to "軽く参考", not for an investigation, so a `preliminary-research` node would be too much. The light reference is folded into the plan's rationale, and the agent offers to add the node if the person wants a fuller comparison | Stopped after planning. The rationale's claim about "general RTS" infantry speeds cites no source. Under the guide's research-evidence fact that is a gap, though a small one for a 'light reference' |

## Assessment

- **The expectation reached the agent, and it acted on it in its own way.** In A it added the study node from the request without asking permission separately, named the integration choices in `plan.md`, and put the research evidence where the plan said. That is what run-004 lacked.
- **No duplicate, no automatic expansion.** B reused existing research work. C chose integration and gave a reason consistent with the request. Neither added a node because a keyword appeared.
- **Node creation is usable from the agent's side.** A made its change with the documented tools (`run.adjust` with the request source, provenance kept in `r2`). Like the deterministic test in step 5, this is evidence from the record, not from private logs.
- **Readable from the ordinary records.** Everything above was read from `plan.md`, the node reports, `wfe run show` and `wfe run revisions`.
- **Open point.** C's light reference has no source. Whether a "軽く参考" deserves even a one-line citation is for the person to decide. If so, it belongs in the person's own research template's expectation, or in a sentence of the guide backed by a trial that shows the gap matters.

## Records and cost

- Copies of the three run folders, the executor log and the attempt transcripts are in `agdevworld/.local/wfe-checks/p7ex1-trial/` (ignored). The scratch area was session-only.
- Attempt cost (reported by the backend): A1 $0.59 (19 turns), B1 $0.29 (14), C1 $0.24 (11), A2 $1.21 (53). Total $2.33.
- Afterwards all three trial runs were cancelled ("isolated p7/ex1 trial ended after evaluation"). Cancellation stopped A's presentation, verified gone, and no trial port was left listening.

## Deus ex machina notes

- did the review-stage cleanup (stopping trial A's presentation by cancelling the run) for agent autolab (wfrun) — handoff candidate. In a real run the agent's own `cleanup` node does this after the person's decision.
