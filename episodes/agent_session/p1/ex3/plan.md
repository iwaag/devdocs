# Agent session p1 ex3 — three larger RTS implementation comparisons

## Objective and scope

Compare `fresh` attempts with ex2's optimized retained `run` sessions on three
distinct RTS improvements that require more implementation than the previous
display-only tasks. Each task is one matched pair, once per policy: six task
runs in total, not three repetitions of the same task.

Build on [ex2's report](../ex2/report.md). Its comparable implementation and
close-out attempts cost almost the same; its whole-run difference was largely
additional correction work. Test whether that pattern holds for somewhat larger
work without claiming that one pair per task establishes average performance.

No deliberate cache-expiry waits, repeated conditions, order-reversal reruns,
extra paid probes, or automatic trial retries. Answers are immediate once the
previous process has ended. Do not implement another session policy or change
models, prompt optimization, compaction settings or the default during the
comparison. Planning and implementation remain in one conversation under `run`;
the person's plan-to-implementation split remains untested.

Creating this plan does not launch experiments or modify the RTS game/services.

## 1. Freeze the baseline, task contracts and budget

Read localrule.md and the relevant workspace operating instructions. Inspect
Nautobot/nctl and WFE state before relying on services. Record the current
deployed WFE/executor, harness version, model/profile, effective environment,
guides, catalog and documentation policy. Confirm ex2's compact continuation is
deployed and there are no unrelated open attempts to disturb.

Use a private RTS clone with isolated run copies and trial input routing. All
six runs start from the same recorded baseline commit; task 2 and task 3 do not
inherit either arm's previous changes. Keep task outputs on their own branches.
Do not integrate competing implementations or change a person's held run.

Initial inspection found production queues, cancellation, attack-move and simple
group-movement spacing already present in the game. The following candidates
add distinct behavior rather than repeat those features. Recheck absence at the
frozen baseline and freeze exact requests and acceptance criteria before the
first model call. If a candidate is already implemented, choose and document a
similar-sized absent feature before starting; never quietly change a task after
seeing one arm's performance.

| Set | Improvement | Work and objective acceptance checks |
| --- | --- | --- |
| A | Production-building rally points | Add an owner-validated simulation command and per-building rally target; newly produced units move toward it. Add a discoverable UI interaction and marker. Existing units remain unaffected, changing a point affects later production, and restart clears it. Test command validation and actual spawn-to-movement behavior. |
| B | Shift-queued movement waypoints | Add ordered movement destinations per unit, Shift-right-click to append, and visible selected-unit routes. Plain movement replaces the queue, Stop clears it, dead units are removed safely, and units advance through destinations. Keep existing attack/gather/build commands working; state their replacement semantics. No attack/build/gather queue expansion or pathfinding rewrite. |
| C | Local save and load of a match | Add one local save slot, visible Save/Load controls and a versioned format. Restore simulation time, resources, entities/IDs, active orders, production and research state, and bot state needed to continue. Reject incompatible/corrupt saves without damaging the live match. Rebuild derived references and reset transient UI/timing state. Test round-trip and continued simulation plus browser persistence. No cloud save, replay system or migration support. |

These are bounded medium-sized features spanning behavior, UI and tests, rather
than cosmetic changes. Do not require an invented line count or extra refactoring
to make them expensive. Save/load is likely the largest; constrain it to one
slot and the current game schema.

Freeze a $45 cap on reported list-price cost for all six runs, including failed
attempts and repairs, with a $9 ceiling per run and a 45-minute active-time
ceiling per run. These are ceilings, not spending or runtime targets. Stop
cleanly at a ceiling, record incomplete work and preserve the evidence. Do not
automatically replace the run or increase the budget. If the actual model/profile
cannot plausibly fit this scope, revise the task contracts before any trial
starts rather than consuming budget on known-unbounded work.

## 2. Prepare one consistent trial protocol

Use the same workflow shape in both arms: plan → implement → verify → present →
result check → one bounded follow-up → verify/present again → acceptance → close.
Only `execution.session` differs (`fresh` versus optimized `run`), apart from
necessary trial identifiers. Keep model effort, guide hashes, tool access,
budgets, output requirements and evaluation criteria identical.

- Both arms receive the same original request and task constraints. Each
  creates its own plan; record plan/design differences rather than pretending
  their work is identical. Use the same scripted plan agreement. Additional
  scope questions receive the same prewritten decisions where applicable.
- Freeze one small follow-up request per task before starting. Disclose that
  it is a staged requirement used to exercise continuation; do not choose it
  after observing an implementation or intentionally reject correct work.
  Suggested additions: A, provide a clear-rally action that affects future
  spawns only; B, add a visible numbered count of remaining destinations;
  C, display the saved match time and disable Load when no valid slot exists.
  Keep these additions out of the initial request in both arms.
- If an arm already satisfies its follow-up, acknowledge that and verify it;
  do not invent repair work to equalize cost. Classify the asymmetry explicitly.
- Use labeled scripted-person inputs, recognize questions by recorded content
  and plan/presentation references, and record explicit agreement and acceptance.
  Ambiguous permission to proceed is not acceptance. Unexpected questions are
  handled from the frozen task contract, with exact replies recorded.
- Deliver boundary answers promptly **after attempt completion and process
  cleanup**, so an immediate answer cannot let one arm continue in its current
  process while the other incurs a new attempt (as happened in ex2). Use the
  existing ended/waiting state; no minute-long sleeps or artificial TTL waits.
  Preserve real questions and record semantics rather than forge lifecycle events.
- Allow at most one ordinary defect-repair loop per arm if objective evaluation
  fails; include its full cost. If still failing or blocked at the budget limit,
  report it as incomplete rather than repeat or substitute the trial.

Prepare a small driver with fixture checks for question matching, lifecycle
gating, budget accounting and acceptance recording. Reuse ex2's working driver
where possible. Do not run the entire workflow or call a model just to test the
driver. Protect the ex1 launch/single-writer behavior.

## 3. Execute exactly three pairs

Run A fresh-first, B run-first, C fresh-first, with one execution slot. Record
order and shared-cache conditions. This balances order somewhat without an extra
repetition; it does not eliminate cache warming or stochastic differences.

Complete both arms of each task before starting the next set. Use unchanged
contracts and the same starting commit throughout. Do not feed another arm's
code, plan or evaluation details into the agent. No coaching an expensive arm
into matching the cheaper design. Do not selectively omit failed runs.

No separate paid smoke check is needed: the first pair supplies real-harness
evidence of the frozen deployed behavior. If the harness or services fail,
inspect existing logs and preserve partial results. Necessary infrastructure
repairs are reported explicitly; do not silently restart a condition on a new
revision. Do not make unrelated service/deployment changes during the trial.

## 4. Evaluate behavior and account for work differences

For each arm, run the existing headless checks and focused tests that exercise
the new feature. Independently inspect the resulting diff and perform a short
browser check against the frozen acceptance criteria, rather than accepting
only the agent's test claims. Use accelerated deterministic simulation instead
of waiting through full real-time matches.

For A, verify a trained unit actually follows a changed/cleared rally point.
For B, exercise appended versus replacing orders, queue completion and Stop.
For C, save a nontrivial in-progress game, reload it, compare key state, continue
simulation, and verify corrupt-save handling and persisted browser reload.
Check the staged addition and preservation of original requirements in all sets.

Record screenshots or compact state assertions sufficient to support the UI
and behavior judgments. Follow the project's existing presentation/review
contract equally in both arms. Do not demand identical designs, test counts or
diff sizes. Keep independent reviewer activity separate from executor cost and
record any extra model usage against the overall cap; prefer deterministic
review tools and no extra model calls.

Collect the following per attempt and aggregate per task/policy:

| Area | Required evidence |
| --- | --- |
| Identity | Baseline, source/deployed revisions, task/arm, session ID, attempt ID and outcome |
| Context | First/last request context, ordinary input, cache reads/writes, appended prompt size, duplicate reads and compaction/reconstruction |
| Work | Steps, output tokens, active time, implementation scope, staged changes and defect repairs |
| Cost | Reported per-attempt deltas and whole-run totals; cumulative session fields deduplicated; interrupted/missing totals explicitly labeled |
| Quality | Acceptance-criterion results, original constraints retained, defects, completed/accepted/incomplete status |
| Conditions | Arm order, natural wait time, observed cache hits, environment and any infrastructure incident |

Report list-price estimates separately from bills or subscription quota. If
usage is missing, provide a labeled estimate or unknown amount, never a false
zero. Do not force cache expiry or compaction; record either if it occurs
naturally.

## 5. Compare, report and preserve useful results

Write `report.md` and short per-set reports with raw-evidence references. Keep
bulky logs, session stores, machine paths and credentials in ignored notes.

- Lead with a three-row task table: fresh cost, retained cost, ratio, quality
  and major work differences. Also show combined cost and active time across
  all three tasks, including unsuccessful work and repairs.
- Separate planning, initial implementation, staged follow-up, defect repair
  and close-out. Show comparable phases where possible without subtracting
  inconvenient work from the primary whole-run total.
- Explain whether larger histories were offset by fewer reconstruction/tool
  steps. Distinguish actual cost from token counts and from offline estimates.
- Do not conclude a policy caused the entire difference when plans, designs
  or repairs diverged. Three distinct pairs provide broader evidence than ex2,
  but remain a small sample with one model. Accept mixed or inconclusive results;
  do not add runs to obtain a preferred result.
- Keep `fresh` as the default during the experiment. Make any recommendation
  proportional to the results; do not silently switch production settings.
  Planning/implementation session splitting is outside the measured comparison.

Preserve each arm's implementation and evaluation artifacts in committed trial
branches so useful RTS improvements can be reviewed later. Do not auto-integrate
either arm, deploy game changes, or make the second task depend on experimental
output. Integration of selected improvements is a separate decision.

Commit/push the plan, reports and required trial-source changes according to
localrule.md. If human review is requested, finish all agent-operable checks and
prepare directly usable evidence first. No new service is required; if one is
needed for previews, follow the LAN-access rule and stop trial-only processes
after collecting evidence.

## Definition of done

- Three distinct, moderately larger RTS tasks each have one fresh/retained pair,
  or a precise explanation of incomplete runs within the fixed limits.
- The same baseline, frozen task contracts and consistent attempt boundaries
  make the comparison reviewable without pretending the work was identical.
- No deliberate long waits, repeated conditions or extra paid probes occurred.
- Whole-run and phase costs, objective quality and work variation are reported,
  with useful outputs preserved and no unrequested game integration.
- Documentation and trial-source changes are committed/pushed.
