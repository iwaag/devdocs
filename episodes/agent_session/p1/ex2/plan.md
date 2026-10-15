# Agent session p1 ex2 — optimize continuation context and repeat the comparison

## Objective and scope

Optimize the information supplied to a retained workflow session, then repeat
the RTS comparison against fresh attempts. Determine whether p1's cost result
persists when a continuation receives current state and new information without
being asked to reconstruct work already present in its conversation.

Read [p1's report](../report.md), [the measured comparison](../report5.md) and
[ex1's fixes](../ex1/report.md). P1 already omitted the common contract and
project guides from resumed prompts; it did not append the entire bootstrap
again. However, both prompt paths used the same generated brief, and both told
the agent to begin with `wfe run show <run> --history`. The follow-up must measure
that duplication rather than attribute all carried context to an implementation
mistake: previous tool results and implementation discussion also accumulate
naturally, and cached input still has a cost.

Keep `fresh` as the default until new evidence supports a change. Preserve
explicit session identity, attempt fencing, single-writer ownership, ex1's
launch/mismatch fixes, human holds and authoritative records. Creating this plan
does not start implementation, paid trials or service changes.

## 1. Audit what the model actually receives and reads

Inspect the executor's `prompt()` / `continuation()`, WFE's `brief()` and its
helpers, the wfrun and project guides, CLI response shapes and shared harness
invocation. Inspect representative p1 transcripts and attempt streams in the
ignored trial notes, not just generated prompt strings.

Build an inventory covering these sources:

| Source | What to establish |
| --- | --- |
| Bootstrap, system/tools and harness-injected files | What is supplied once, reused as history, or injected again by the harness |
| Continuation brief | Stable identity/paths, catalog, documentation policy, ready/waiting state, pending inputs and recovery facts |
| Agent reads after resumption | Repeated history, braindump, plan, definition, guides, reports, CLI help and file scans |
| Tool responses | Full records, event history, repeated metadata and oversized output when only a small part is needed |
| Session maintenance | Compaction, context loss, cache expiry and configuration replacement |

For each repeated element, record its purpose, whether it can change externally,
how the agent learns changes, and whether it should be omitted, delivered as a
delta, summarized as current state, or fetched on demand. Fix other material
session-specific waste discovered here; do not limit implementation to removing
the literal `--history` flag. Avoid speculative infrastructure with no measured
source of waste.

**Evidence:** representative before-context breakdowns, repeated-read examples,
and a concrete list of optimizations and retained correctness checks. Distinguish
newly appended duplication from old history the model necessarily still reads.

## 2. Implement a compact, reliable continuation contract

Separate new-session reconstruction from retained-session continuation in WFE
and the executor. A replacement or retired/missing session gets the complete
bootstrap regardless of the run's selected policy.

- For a normal continuation, supply the new attempt identity, run/session
  identity, prior attempt outcome, a compact current-state snapshot, and new
  external inputs or relevant changes. Preserve the fact that previous attempt
  IDs are ended and their processes are gone.
- Do not append unchanged contracts, guides, catalog descriptions, documentation
  policy, workspace explanations or historical answers again. Keep stable
  information accessible through records and targeted commands. Send changes
  explicitly; a configuration change still follows the existing replacement
  rules rather than pretending the retained session knows a new guide.
- Remove the unconditional full-history read from the resume prompt and any
  guides that effectively reintroduce it. Prefer authoritative state already
  supplied by claim, or a compact current-state/delta command if a later refresh
  is needed. Merely dropping `--history` is insufficient if the remaining
  response still dumps the entire run record.
- Deliver new answers and instructions with exact text, stable IDs, authors and
  references needed for take-up/acknowledgement. Include holds, cancellation,
  agreement/acceptance changes, graph revisions and other external changes that
  affect the next action. The session transcript does not automatically contain
  events recorded while its process was absent.
- Define an explicit record revision/event boundary for the snapshot and delta.
  Reuse existing sequence/receipt mechanisms where possible. A claimed or sent
  prompt is not proof that an input was read: pending, unacknowledged inputs
  must remain available after launch failure, interruption or retry. Prefer
  bounded repetition of pending inputs to silent omission. Never infer a safe
  delivery cursor solely from timestamps or a normal process exit.
- Preserve checks for instructions arriving after claim and at node boundaries.
  A compact startup snapshot cannot authorize ignoring later input; existing
  write-time refusals and fencing remain in force.
- Do not routinely reread an unchanged accepted plan, original request, help or
  definition merely because a new attempt began. Give the agent references and
  targeted refresh paths when a revision, uncertainty or missing context makes
  a read necessary. Retain meaningful file/diff verification before changing
  work; removing duplicate context must not remove verification of actual state.

For interrupted attempts, supply recovery findings and inspect files/records
needed to establish uncertain side effects before repeating commands. Full
history remains an available diagnostic, not a mandatory prelude. After
compaction, detect what the harness exposes and provide a compact recovery/read
path when necessary; if its completeness cannot be established, fall back to
authoritative reconstruction explicitly. Cache expiry alone does not mean the
conversation was lost and does not require a bootstrap.

Records, plans and reports must remain sufficient for `fresh` or replacement
sessions. Do not suppress durable reporting to make retained sessions cheaper.
Do not add automatic compaction, change models or split sessions in this main
optimization comparison: each would introduce a separate experimental factor.

**Evidence:** captured new/resume/replacement prompts and actual tool outputs
demonstrate that stable material is not appended or routinely fetched again,
while new input and state remain discoverable and actionable.

## 3. Verify the changed correctness boundaries

Use focused tests and short disposable harness probes where real transcript
behavior matters. Avoid broad repeated suites and long timing-based waits.

| Case | Required result |
| --- | --- |
| Ordinary resume | Compact state and pending/new inputs; no mandatory history or unchanged bootstrap |
| New or replacement session | Complete reconstruction, including original requirements and current plan |
| Multiple answers/instructions during a wait | All pending input is discoverable, correctly taken up and acted on once |
| Input after snapshot or during execution | Boundary refresh/write-time checks expose it; no stale completion |
| Prompt/launch failure or interrupted delivery | No input disappears because it was only claimed or sent |
| Hold, cancellation, acceptance and graph changes | Current authoritative state controls action; old memory cannot override it |
| Interrupted command | Effects are checked before retry; no blind replay |
| Guide/config change, missing store or mismatch | Existing replacement/refusal and ex1 ownership rules remain intact |
| Compacted conversation | New input and requirements survive, or explicit reconstruction occurs |

Test delta boundaries and retries deterministically with fixtures. Include a
compact-state output-size check against a long synthetic history, and inspect
a real resumed transcript to catch behavior that prompt assertions miss. Run
relevant existing executor/WFE regressions and required release checks once;
expand testing only for affected code or unresolved failures.

## 4. Run a bounded, controlled RTS follow-up

Record versions, baseline commits, exact prompts/guides, execution settings,
trial inputs and a spending cap before starting. Inspect the actual installed
harness/model and observed cache behavior; do not assume p1's version or TTL is
still applicable. Use Nautobot/nctl before depending on local services, following
the workspace operating instructions. Keep raw transcripts and machine-specific
details in ignored local notes.

Use three arms on the same current RTS baseline and model/profile:

1. `fresh`: the existing fresh-attempt behavior, reconstructing from records.
2. `run-before`: retained sessions with the pre-optimization continuation and
   read guidance, pinned to a reproducible revision.
3. `run-optimized`: retained sessions with step 2's compact continuation.

Hold task, workflow graph, documentation requirements, accepted plan constraints
and trial inputs constant. Keep any shared correctness fix common to all arms;
document unavoidable differences. Use isolated run copies and session stores as
needed to prevent experimental cross-talk. Do not edit a held person's run or
integrate trial implementations into the shared project.

- Run one immediate-answer triplet and one delayed-answer triplet, reversing
  arm order in the second. Six task runs maximum and a $30 cap on reported
  list-price cost, including probes and repairs. If a valid comparison cannot
  fit, stop and report incomplete evidence; do not silently enlarge the budget.
- Use an actual useful task whose feature is absent from the baseline. Include
  plan agreement, implementation, a bounded correction and explicit result
  acceptance. Preserve original requirements through the correction. Use the
  same clearly labeled scripted-person inputs in all arms; recognize re-checks
  by their meaning/recorded question, not a hard-coded node name. Ambiguous
  input must not be counted as acceptance.
- For the delayed triplet, wait beyond the observed cache TTL without keeping
  an attempt or slot alive. Record actual wait duration and hit/write evidence;
  do not infer expiry solely from elapsed time. Avoid cache-clearing operations
  that affect unrelated work, and record shared-prefix warming/order effects.
- P1's figures remain historical reference, not a substitute for the concurrent
  `run-before` control. Six runs are a small-sample diagnostic, not a definitive
  estimate of average savings.

Measure both actual total outcome and the mechanism of any change:

| Measurement | Required detail |
| --- | --- |
| Context | Appended prompt size, first-request context, cached reads/writes, ordinary input, repeated tool-output volume and history/plan reread counts |
| Work | Tool/model steps, active time, output tokens, extra work and repair attempts |
| Lifecycle | Session/attempt identity, compaction, replacement, wait and cache conditions |
| Cost | Per-attempt and whole-run reported cost, cumulative-session deltas, missing totals and any separately estimated amounts |
| Quality | Original requirement retention, plan adherence, correction uptake, tests, actual UI result, explicit acceptance and stale-state failures |

Use a small paired context replay/probe, within the same budget, if needed to
separate reduced input from stochastic differences in work. Do not present
token savings as measured monetary savings without actual usage/cost evidence.
Include all failed and repaired runs; label subscription list-price estimates
as estimates rather than bills or quota consumption.

## 5. Evaluate phase boundaries separately

The person's normal practice is a planning session followed by a fresh
implementation session receiving the approved plan. Neither p1 nor the main
three-arm comparison tests this policy. Explain that limitation explicitly.

Use the optimized trial's phase-by-phase context measurements to propose a
specific plan-to-implementation boundary, with original requirements, accepted
plan revision, answers/constraints, artifact references and current state as the
handoff. A plan-only handoff must not accidentally omit constraints that the plan
failed to restate. Implementation and its short correction loop may then share
a session.

Document the smallest explicit policy/UI design and a separate matched trial
needed to test it. Do not silently add session splitting to `run-optimized`, or
claim savings for it from these trials. Implementing a new phase policy is a
follow-up scope, rather than a prerequisite for measuring input optimization.

## 6. Deploy the verified optimization and report

Commit/push code changes and required dependency/submodule pointers. Check
current WFE runs and service ownership before activation; deploy matching
versions without interrupting active writers or lifting holds. Newly started
services follow localrule.md's LAN-access requirement. Execute the agent-operable
deployment checks and one short continuation smoke check before asking for any
human verification; count paid smoke usage within the cap.

Update session documentation and guides to describe ordinary compact
continuation, targeted refresh, uncertain recovery and full reconstruction.
Write `report.md` with the audit inventory, implemented optimizations, exact
tests, source/deployed revisions, before/after examples, trial tables, quality
findings and remaining limits. Account separately for total carried history,
avoidable duplication, cache expiry and extra work. Keep p1's measured results
intact and link this follow-up rather than rewriting the historical result.

Recommend a default only from the resulting evidence; a negative or inconclusive
result is valid. Commit/push the final report and documentation as required by
`localrule.md`.

## Definition of done

- A normal retained-session continuation avoids redundant startup context and
  blanket history reads, with other observed material waste addressed.
- Authoritative new input, current state, race handling and safe reconstruction
  remain correct across ordinary resumes and the changed failure boundaries.
- Controlled RTS evidence separates the effect of input optimization from work
  variation and cache expiry, including whole-run cost and quality.
- Plan/implementation session splitting is clearly distinguished as untested.
- The optimization is verified, deployed, documented and committed/pushed, with
  a reproducible comparison and an evidence-based recommendation.
