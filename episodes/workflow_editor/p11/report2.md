# p11 Step 2 — WFE implementation progress

Status: **in progress; Step 2 is not complete**. This report covers the tested
publication and dotfile changes only. The remaining WFE changes and Steps 3–6
are not implemented or deployed by these commits.

## Implemented

- Removed both checkpoint file-size vetoes from `server/publish.ts`: intended
  working files above 50 MiB are committed and pushed, and an agent's own
  unpublished commits containing such files are pushed. There is no replacement
  override flag or permissive mode. Shared storage remains available.
- Replaced blanket dot-entry exclusion in `server/shares.ts` with explicit
  transport/Git/OS metadata names. User dotfiles and nested dot directories now
  pass through input sets, output publication, manifest verification, directory
  listings and HTTP downloads. Exclusions are `.wfe-share.json`,
  `.wfe-output.json`, `.partial-*`, `.staging-*`, `.git` and `.DS_Store`.
  The output manifest itself is metadata, avoiding a self-referential file set.
- Updated the maintained shared-area documentation and replaced the two
  size-refusal tests with successful publication tests using real 51 MiB files.
  Added a dotfile input → output → verification → HTTP regression.

## Validation

The relevant publication/shared-area suites passed **16/16**. The large-file
cases exercise both automatic checkpoint commits and an agent-created commit,
then inspect the remote Git object size. The dotfile case verifies both
`.gitignore` and `.config/settings.json`, including downloaded bytes and a
folder listing that excludes transport metadata. Existing tests for interrupted
publication, retries, tampered manifests and byte-range downloads also pass.

Full `npm run check` passed: **380/380 tests**, no failures or skips,
TypeScript checking and the production Vite build successful. No installed release,
executor setting, registered workflow, existing run copy or infrastructure
resource has changed. The Developer-stopped p6 runs have not been resumed.

## Remaining work

The WFE portion of S2/S5 is implemented in source; playprey's dotfile-dropping
workaround and large-file instructions remain open. S3 (integration without
acceptance), optional documentation/plan records, graph revision and reopening,
path/binding policy removal, delegation changes, UI/help and existing-state
migration still require implementation and regression coverage.

A broader reducer/integration experiment was evaluated and set aside in ignored
working evidence. It is not part of the committed implementation: the changed
record semantics need a complete migration and cross-layer validation. A focused
passing test of that draft is not evidence that the retained source supports it.

Inventory entries remain open until all their consumers and deployment proofs
are covered. In particular, this report is not a claim that the six-step plan or
its completion criteria have been fulfilled.

## Published source revisions

| Repository | Revision |
| --- | --- |
| agdevworld | `4497e5b` |
| pj-agdev gitlink | `6248733` |

Both source revisions are pushed. The installed WFE remains on its prior
release; this partial source change has not been deployed as the complete p11
change. Raw validation logs and the broader draft are in ignored
`pj-agdev/.local/workflow-editor-p11/`.
