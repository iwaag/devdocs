# cagent — removal of the self-dependency-host restriction

Developer instruction, 2026-10-16: remove the rule that makes changes to hosts carrying cagent/WFE/Nautobot/Zulip person-operated. This supersedes pre1 decision D9 and its corresponding design, plan and trial requirements. The earlier reports and run snapshots remain historical evidence, not current instructions.

## Result

- Removed the deterministic host checks from `cagentctl apply`, `reconcile` and `playbook`, the preview flag and the `self_dependency_hosts` configuration field.
- Updated the project intent, change workflow definition, CLI help, guides and example/ignored configuration. Dependency hosts now follow the same plan agreement, scoped execution, ledger and verification path as other hosts.
- Kept the independent destroy-class rule, plan agreement, prerequisite checks, effect tracking and recovery behavior.
- Updated the registered workspace and existing waiting change-run copies. Existing immutable definition snapshots, agreed plans and past person-handoff entries are retained as evidence; this amendment does not claim they ran or authorize an otherwise unagreed operation. The updated guide records the Developer's superseding instruction.

## Verification

`uv run --project tools/cagentctl pytest tools/cagentctl/tests -q`: **30 passed**. Fake-backend tests cover the command host as a reconcile target and as a contacted host, a batch naming it, and an allow-listed playbook on it. Tests still exercise destroy-class refusal, plan/attempt validation, scope checks and person confirmation.

The changed workflow validates without issues. Its definition approval records the Omni Agent applying this explicit Developer instruction. No live reconcile, batch apply or playbook was executed to test the removal; no waiting run was resumed on the Developer's behalf.

Implementation commit in private `pj-cagent`: `9cc0601`. Runtime paths and local synchronization evidence belong in ignored `pj-agdev/.local/project-migration-p5/` notes.
