# scheduled_routine p6 — Plan

Braindump: `braindump.md`. Add a routine for study-pattern projects that
turns an existing arXiv `summary.md` and `manual.md` into an actual local
reproduction attempt. For each selected paper, autolab creates a new Gitea
repository cloned as `localtest-<paper-id>/` inside the project workspace,
runs what can reasonably run on the local machines, and leaves a useful
result whether the experiment succeeds, fails, or needs an upper development
actor.

This is a private experimental environment and a destructive phase. Backward
compatibility is not required. Prefer a small live-tested contract over a
general workflow framework, and let findings from the first runs determine
the exact implementation.

## Decisions and useful context

- The existing study pattern already permits extra folders and
  `autolab project init-repo <folder>` already creates the standard local
  Gitea repository. For `studyarxiv`, `localtest-2608.23283/` should normally
  map to `autodev/studyarxiv-localtest-2608.23283`. Replace `/` in old-style
  arXiv IDs with `-`; otherwise preserve the paper ID.
- Pattern-managed projects now have Plane projects and can execute ordinary
  missions (`scheduled_routine` p5). Use that path; do not invent a second
  task runner for local tests.
- A local test may need several bounded tasks. Model downloads, installation,
  service startup, and evaluation do not have to fit in one autolab run.
  Persist enough state in the localtest repository for a later routine fire
  to resume it. The precise file format is the implementer's choice; a small
  `localtest.yaml` plus `report.md` is a useful starting point.
- Treat `waiting_external` as a normal result of a run, not as a process that
  stays alive. Record the requested action and the read-only evidence that
  will show it has happened, then end the mission. A later routine fire reads
  the state and continues.
- Starting a project-owned Docker Compose stack is encouraged when it is the
  shortest honest test. A successful experiment is not automatically desired
  cluster state: after the test, decide whether to remove it, retain it as a
  manually managed service, or propose a managed deployment.
- Exact hostnames, addresses, local paths, credentials, and cluster-specific
  handoff data belong in ignored `.local/` material or the local cagent /
  Nautobot surfaces. The committed report should contain portable reproduction
  facts and sanitized evidence.
- Current relevant workspace:
  `pj-agdev/agautolab/.local/projects/studyarxiv/`. Its `main/papers/INDEX.md`
  has two papers with manuals and `runnable: yes` at planning time. Re-read
  live state rather than assuming those rows remain unchanged.
- Current cluster reads work: `nctl status` is healthy. The planning-time
  drift summary was `converged=42, drifting=2`; the two existing drifts are
  unrelated to this episode, so use scoped before/after evidence rather than
  requiring global convergence.
- nodeutils already collects a sanitized list of all Docker containers,
  Compose projects, and published ports. Only known or hinted services become
  normalized observed services today. Use this fact before adding another
  collector.
- If the permission classifier stops an in-system run, stop that run and
  report it as required by `localrule.md`; do not work around it.

The only hard prohibitions for this phase are: never commit credentials or
private machine/cluster facts, and never convert an experimental runtime into
Desired State automatically merely because it was observed.

## Step 1 — define and prove the localtest workspace contract

Extend the study-pattern documentation with the convention that a paper may
have a repository-backed `localtest-<paper-id>/` folder. Keep it a convention,
not a mandatory third folder in every study project.

Exercise the existing `autolab project init-repo` path in tests with a
`localtest-<paper-id>` folder, including one old-style ID normalization case.
Decide where the minimal resumable state lives and document the few states the
routine actually needs; suggested states are `prepared`, `waiting_external`,
`running`, `verified`, `failed`, `adoption_pending`, and `complete`. Do not add
a database or a new service for this state.

The repository should begin with a useful `.gitignore`, a README naming the
paper and upstream sources, and whatever small state/report skeleton the
implementation chooses. Update the study project's `README_PROJECT.md` after
adding the folder, as the pattern contract already requires.

Report `report1.md`: documentation/code changes, tests, the chosen state
contract, and the exact standard repository naming demonstrated by the test.

## Step 2 — add the `localtest` routine

Create standing text in `#front › routine-localtest`. It should tell Front to
ask autolab for one mission in the study project which:

- resumes an existing `waiting_external` or `adoption_pending` local test
  before starting another one;
- otherwise selects one paper with `runnable: yes`, an existing `manual.md`,
  and no completed local test;
- creates its `localtest-<paper-id>` Gitea repository and workspace folder;
- reads the summary, manual, official code/README, and current local/cluster
  facts before choosing the smallest meaningful experiment;
- records commands, upstream revisions, expected evidence, actual evidence,
  and the result;
- splits work into further Plane tasks when downloads or execution exceed one
  task-sized run;
- records and reports an upper-actor request instead of waiting indefinitely.

The standing text should describe the desired outcome, not prescribe one
installer, container runtime, model server, or report template. Let autolab
choose those from the paper and live environment.

Add a one-shot schedule event through Front's existing `rtschedule` path and
let the production dispatcher fire it. No dispatcher schema or trigger change
is expected; change them only if the live run supplies a concrete reason.

Report `report2.md`: standing text, schedule request/event IDs, dispatch
timeline, Front/autolab topics, and any first-run questions or routing failure.

## Step 3 — run one local reproduction end to end

Use one currently eligible paper, preferring the smallest credible experiment
rather than the most ambitious model configuration. Let the mission create the
repository, prepare the upstream code, install project-local dependencies,
and attempt one documented minimal command. Docker is acceptable and
preferred when it makes cleanup or repeatability easier.

The test succeeds when it produces evidence that answers a concrete claim
from the manual; merely reaching an installer exit code is not enough. Failure
is also a valid result when the report identifies the failed boundary and
retains enough evidence for the next attempt.

Verify:

- the localtest repository is committed and pushed to local Gitea;
- rerun instructions do not depend on chat history;
- local/private details and large model artifacts are ignored;
- the paper index or another obvious project-level view points to the local
  test and its state;
- a second routine fire resumes or recognizes the finished state instead of
  creating a duplicate repository.

Report `report3.md`: selected paper, repository, task timeline, installed or
downloaded components, the minimal command, evidence, result, cleanup state,
and every human intervention.

## Step 4 — prove the upper-actor wait and resume path

Use a real requirement from the first experiment if one appears: a host-level
tool, large model download, GPU/runtime choice, service port, or nctl mutation.
If the first paper needs none, select another eligible paper or create a small
controlled wait at a genuine host-change boundary.

Have the mission finish in `waiting_external` with a local-only handoff that
states:

- the requested operation and why it is needed;
- expected disk/GPU/network/runtime impact;
- processes, containers, ports, and files it expects to create;
- a read-only check that proves the operation is ready;
- cleanup or rollback advice where useful.

The Developer or Omni Agent performs or rejects the operation. Fire the same
routine again and verify that it reads the persisted state, observes the new
reality through cagent or ordinary read tools, and continues without the human
having to restate the paper or plan. Do not require the old mission process to
remain alive.

Report `report4.md`: the handoff as seen by the upper actor, who performed or
rejected it, the before/after evidence, and the resumed mission's result.

## Step 5 — account for experimental services in cluster intent

For every container or persistent process created in Steps 3–4, make one
explicit disposition:

1. ephemeral experiment — capture evidence and stop/remove it;
2. retained manual service — propose a `desired_service` and
   `desired_service_placement` using an existing manual profile where it fits;
3. retained managed service — describe the missing deployment profile,
   observation, and reconciler work for a later episode.

Send cagent a report for a retained runtime or for any cluster explanation
made false by the experiment. cagent remains read-oriented; it records or
explains the finding and does not repair it. Put any exact proposed nctl batch
in ignored local material for Developer/Omni review. Apply it only with the
authority available during execution, then verify with scoped `nctl drift`.

Test whether existing nodeutils observations make an unregistered localtest
Compose project discoverable. If the report path proves sufficient, leave the
collector alone. If a live retained runtime is invisible or easy to forget,
implement the smallest read-only discovery surface that compares sanitized
observed containers/Compose projects with desired placements. It must report
unmanaged candidates; it must not create Desired State.

Report `report5.md`: runtime inventory, each disposition, cagent exchange,
any reviewed/applied desired-state change, final scoped drift, and whether a
discovery code change was justified by evidence.

## Step 6 — repeat once and write the phase report

Run `localtest` for one more eligible paper, or rerun the same paper from a
clean checkout if there is no second credible candidate. Avoid manually
guiding the agent around findings fixed in earlier steps. Record whether the
routine independently selects/resumes the right work, whether its repository
is useful, and whether cleanup/intent disposition happens without prompting.

Write `report6.md` for the second run, then `report.md` with:

- papers attempted, verified, failed, waiting, and completed;
- repositories created and pushed;
- Front runs, autolab tasks, elapsed time, and human/Omni interventions;
- downloads, installs, containers, services, and their final dispositions;
- whether a later fire resumed correctly;
- every implementation change demanded by live evidence;
- whether the result is useful enough to schedule regularly;
- the next feature, if any, supported by evidence rather than anticipation.

Leave a real recurring schedule only if the Developer wants continuing local
experiments. Otherwise remove or expire the p6 test request/events and leave
the standing routine available for manual fires.

## Out of scope unless a live run makes it necessary

A general experiment orchestration service; automatic model-cache management;
automatic Desired State creation; a universal paper reproduction schema;
remote/cloud GPU provisioning; publishing to the study project's `publish/`
repository; fixing unrelated existing cluster drift.
