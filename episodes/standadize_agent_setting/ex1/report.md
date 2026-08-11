# Final report — shared agent-setting package `pyagag`

## Outcome

The duplicated Python agent configuration and harness layer has been extracted
into the public `pyagag` repository. Both agautolab and agforge now consume it
through the `agag` import package.

The goal is met: a new Python harness is implemented at the shared config,
argv, compatibility, capability, and output-extraction seams in pyagag;
agautolab and agforge do not carry harness-specific configuration or process
implementations. agdevworld deliberately remains a JavaScript sibling of the
same language-neutral contract.

## Delivered changes

- pyagag: hatchling/src/uv package, shared config resolver, non-raising harness
  runner, output normalization, environment injection, run-record writer,
  ported tests, and the public contract document.
- agautolab: deleted duplicated config/harness modules, retained only its
  project-path resolver wrapper and adapter/orchestration policy, and adopted
  the shared run-record writer.
- agforge: converted from flat PEP 723 service scripts to a hatchling src-layout
  package, retained thin legacy launchers, removed `sys.path` glue, and adopted
  the shared resolver/harness result seam.
- Secret fix: Anthropic `_file`/`_env` references now reach agforge subprocesses
  through `ResolvedAgent.environment`, with deterministic integration coverage.
- Documentation: `ag.agent-config.v1`, `HarnessResult`, and
  `ag.agent-run.v1` are documented in pyagag; editable sibling path dependency
  expectations are documented in its README.

## Verification summary

- pyagag: 19 passed; sdist and wheel built.
- agautolab: 80 passed; fake `run_once` smoke passed; sdist and wheel built.
- agforge: 58 passed; fake HTTP request/environment smoke passed; sdist and
  wheel built.
- Local cluster status: `nctl.status.v1` returned `ok: true` with Nautobot and
  worker healthy.
- agforge `:8092`: not running, so the optional restart/live request was skipped.

## Commits

- pyagag: `253f752` (package), `6106eef` (contract docs)
- agautolab: `cc9d892` (consumer migration), `bff78f1` (shared record writer)
- agforge: `f6507f0` (package migration)
- Step reports: one devdocs commit per `report1.md` through `report4.md`; Step 5
  and this final report are committed together after this document is written.

No deployment to agautolab1, gitea push, Ansible execution, or external model
run was performed, matching the plan's scope.

Omni Agent completed the package extraction for the project agents — handoff candidate.
