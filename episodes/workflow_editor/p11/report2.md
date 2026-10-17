# p11 Step 2 — WFE execution and integration

Implemented in source. Deployment and application to existing records belong to
Step 5; this report does not claim an installed release changed.

## Behavior

- Definition approval remains informational. Starting work needs no plan
  agreement, completed predecessor or acknowledged instruction. Completed work
  can reopen. Pending generated documentation no longer prevents authored work
  completing; available document evidence is recorded without inventing agreement.
- Graph revisions can change running/completed work, dependencies, bindings,
  execution settings and documentation. The CLI defaults to an agent-sourced
  revision. Explicit request citations identify actual input bytes rather than
  granting authority. Stale writes, malformed graphs and actual attempt ownership
  retain concrete diagnostics and agent-operated stop/resume/retry tools.
- Protected/retired workspace metadata and readonly/if-unchanged binding declarations
  no longer veto work or integration. Missing binding directories are informational;
  actual file/Git operations report concrete failures.
- Integration checkpoints actual working changes and merges the saved candidates
  into target branches without an accepted revision or completed execution. Later
  deliverable, plan and final-report edits require no renewed acceptance. Clean
  changes to different parts of the same file merge. Real conflicts name files;
  an agent can merge/resolve in its run copy and retry. Partial pushes retain
  their journal and actual commits. A retry refreshes changed candidate commits.
- Delegated answers/decisions need no person-only acceptance scope or prior
  take-up. Delegate completion does not require a prescribed judgment sequence.
  Recovery has no person-reset budget. Actual explicit stops/cancellations remain.
- CLI help, generated step instructions, dynamic attempt briefs, browser integration
  controls/status and maintained integration/revision documentation describe the
  new semantics. Historical decisions remain records, not manufactured approvals.

The existing integration-journal column called `decision` identifies the source
history sequence for new requests; runtime code no longer looks it up as an
acceptance index. Historical rows keep their original values. Pending requests
share an unfinished integration so retries retain partial progress.

## Publication and transport

The preceding source commit `4497e5b` removed both 50 MiB checkpoint/push vetoes.
Real 51 MiB working files and agent-created commits publish to temporary bare
remotes. User `.gitignore` and nested dot directories pass input selection,
publication, verification, listings and HTTP download. Named Git/OS/transport
metadata remains excluded, including the output manifest itself. Playprey's
producer-side workaround is handled in Step 4.

## Validation and migration support

`npm run check` passed **380/380 tests**, no failures or skips, with TypeScript
checking and production Vite build successful. Coverage includes no-acceptance
integration, final reports after acceptance in both devdocs storage modes,
advanced targets, clean same-file merges, real conflicts/rejected pushes,
reopening, concurrent/running-node revisions, no-agreement operations, delegation,
large files and dotfile transport.

An explicit projection migration script backs up original run.json bytes, replays
actual events and rewrites derived state/change projections. Its regression proves
actor/event preservation, no fabricated decisions, idempotency and preservation of
a Developer stop while obsolete pending-document completion behavior disappears.
Malformed histories remain errors. Frozen guides and installed sources need the
separate deployment migration; source tests do not establish it.

Raw logs are in ignored `pj-agdev/.local/workflow-editor-p11/`. No stopped p6 run
was resumed. Source: agdevworld `b9886d5` (in addition to `4497e5b`).
