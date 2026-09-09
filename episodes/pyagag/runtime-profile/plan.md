# Runtime profiles for agag agents

## Goal and approach

Let Front fulfill requests such as “run routine-study-realworld using agy until
its usage exceeds 70%” by discovering the recipient's published execution
options and selecting one for the work. Preserve conversational delegation:
agents decide whom to ask and what to request; code reliably applies the
selected option before launching a model.

This is a private experimental environment in a breaking-change phase. Backward
compatibility is unnecessary. Implementers may refactor existing paths, replace
obsolete conventions, and choose command syntax, module boundaries, and storage
details. Keep the contract small; avoid adding approval machinery or broad
security infrastructure for this episode. Keep machine facts and credentials in
ignored local files, following the existing repository policy.

Paths below are relative to the shared projects directory.

## step1 — Define the public execution contract

- Describe public execution options separately from internal profile names.
  Each option needs a public name, a short explanation, the usage pool it
  consumes, and the work it covers. The recipient owns its mapping to internal
  profiles and roles; a simple one-to-one mapping is a valid starting point.
- Define topic-local selection and reset commands targeting the topic's owner.
  Configuration-only posts update the setting without starting agent work.
  Distinguish actual commands from quoted examples and ordinary discussion.
- Freeze the effective selection at each serving's start. A later command
  affects subsequent servings, including callbacks, rather than changing an
  in-flight process. Use message ordering and retain the source message ID.
- Define precedence: explicit topic selection, inherited selection, then the
  existing project/instance/role defaults. Make reset behavior explicit; prefer
  returning to the next applicable default rather than introducing extra modes.
- Reject invalid selections visibly without silently running another profile.
  Missing capability information means unknown, not confirmed unsupported.

Deliverable: a short contract with selection, reset, rejection, and inheritance
examples. Choose the exact syntax during implementation.

## step2 — Implement selection and discovery in pyagag

- Add shared parsing, selection resolution, and run metadata support. Treat
  topic history as the durable source where practical, consistent with existing
  conversation notes. Reuse existing handling of resolved topic names.
- Connect selection to `agag.agent.resolve_spec_role` /
  `agag.agent_config.resolve_role`; `profile_override` already exists. Reuse
  role capability checks and perform availability validation at execution time.
- Extend introductions with support status and execution options. Keep short
  catalogs inline; use a linked detail topic if needed. Publish only the
  supported external options, not the entire local configuration.
- Give callers a convenient way to select/reset through the existing chat
  tools. Acknowledgment should not create extra agent servings; existing
  selfnote filtering and the notifier's reaction approach are useful references.
- Record the requested public option, selection source, and resolved
  profile/harness/model with each run. Existing harness records already carry
  execution identity; extend rather than duplicate them.

Check: selection, replacement, reset, invalid requests, restart recovery,
resolved topics, and configuration-only posts behave consistently without
launching paid models in unit tests.

## step3 — Carry selections through autolab work

- Wire topic selection into planning and task execution in
  `pj-agdev/agautolab/src/agautolab/zulip_listener.py`, especially
  `run_superdirector` and `workrun_supercoder`. The wrapper in `role_run.py`
  already accepts `profile` and prioritizes it over project settings.
- Cover all internal agent runs needed for the advertised work. If an option
  uses different profiles for auxiliary roles, keep that mapping inside autolab
  and explain any usage-pool exceptions in the public option.
- Carry the accepted plan selection into newly created `workrun-` topics before
  their first execution. Prefer a snapshot at child creation: later changes to
  a plan affect new children, while existing children keep their selection until
  explicitly changed. Preserve provenance and allow a child override.
- Reuse the task's selection after callbacks and listener restarts. Inspect
  `anchor.py`, `agag.selfnote`, and the existing home/served-note routing;
  a callback's remote topic is not the task's execution context.
- Publish autolab's options from its actual configured mappings. Verify an agy
  option covers planning and task work, not merely the entrance response.

Check: two simultaneous missions can use different options; children and
callback continuations retain the correct selection; normal defaults still
work when no selection is made.

## step4 — Teach Front to request execution options

- Extend Front's tool guidance to discover options through introductions,
  interpret the developer's preference, and select an advertised option when
  delegating. Keep recipient names, local profile names, and role mappings out
  of Front's routing code.
- Preserve the execution preference in the routine run's opening context so
  later servings and new delegations continue to honor it. Selecting autolab's
  option need not change Front's own execution profile.
- At a delegation to another agent, discover that recipient's options again;
  translate the intent rather than blindly forwarding a profile name. Report
  unsupported or unavailable requirements to the requester.
- Keep execution selection separate from the stopping condition. Reuse
  `pj-agdev/agfront/src/agfront/budget.py` and `agbudget`: usage is the current
  shared account window, not this routine's cost. Identify the intended window
  and match the observation to the execution pool. Failed/stale reads remain
  unknown; preserve the requested threshold semantics, including “exceeds.”

Check: with fixture introductions and budget observations, Front can discover
an agy option, request it, continue across callbacks, and explain why a run
ended using the observed usage. Include already-reached and unavailable cases.

## step5 — Verify the complete workflow in the experiment environment

- Read `pj-clusterintent/nctl/README.md` and use Nautobot or read-only nctl
  queries to establish current service state before deployment or live checks.
  Local environment notes provide startup, dependency-update, and reload hints.
- Run focused shared-library, listener, and Front tests, then each changed
  repository's required checks. Exercise command handling with deterministic
  fixtures, including a command arriving during a serving and rejection without
  an accidental model launch.
- Update consuming dependency locks, reload affected services, and refresh
  introductions so the running agents expose the implemented contract.
- Demonstrate Front discovering the published option and autolab using it for
  a small plan, a child task, and a callback continuation. Inspect actual run
  records and prove an unrelated topic retains its own setting.
- Use controlled budget observations for threshold/reset/failure cases; a
  useful live proof need not consume a real account up to 70%. Distinguish
  fixture evidence from actual harness execution in the report.

## step6 — Document and finish

- Update public usage documentation and relevant agent introductions/guides;
  put machine-specific deployment notes in ignored local documentation.
- Write `report.md` with the final contract, commits, test results, live
  evidence, and remaining limitations. Explain any differences from this plan.
- Commit and push the episode's changes in each affected repository, including
  dependency/submodule updates where applicable, following `localrule.md`.
