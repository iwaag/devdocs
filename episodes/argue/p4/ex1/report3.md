# argue p4 ex1 — Step 3: archives migrated, destination assumptions removed

## Migration evidence

- **Import.** pj-ponderer `bbbe548` merges pj-argue `main` (`4fd89fd`)
  with `--allow-unrelated-histories` (Step 2). It includes 11 discussions:
  8 closed and 3 open. Three of them have a `summary.md`.
- **Byte identity.** `git diff argue/main 2b0be53 -- discussions episodes`
  is empty. Message ids, authors, timestamps, runs, annotations and
  horizons are argue's bytes.
- **Manifests against transcripts.** `tools/discussions validate` reports
  `ok` for all 11: format, every record once, transcript extent equal to
  the manifest horizon, every reference resolvable. `tools/discussions
  episodes validate` reports `ok (3)`.
- **Citations.** Every distinct `argue@<commit>` found in tracked files of
  the 15 workspaces and in devdocs (14 commits) is a commit of pj-ponderer.
  None is unresolved (`pj-agdev/.local/argue-p4-ex1/step1/argue-citations.txt`).
  The `last commit that changed discussions/<id>/manifest.json` in
  pj-ponderer is the argue archive commit itself (for example `04a807d`
  for argue-p4-trial), so links built from it point at real content.
- **Run history.** The run records stay at their argue commits. The 17 run
  branches are tags `history/argue/wfe/argue/…` in pj-ponderer.
- **Current links.**
  - aghangout `ASSETS.md` keeps its literal historical citation, now
    explained: it resolves in pj-ponderer, and the current conclusion is
    forge's.
  - archsage, forge, Front and ponderer guides cite ponderer and forge
    locations.
  - Historical episode reports and run records are left as written.

## Publication behaviour (WFE agdevworld `7893d75`, not yet deployed)

`server/discussionArchive.ts` and store **v8**
(`archives.destination`):

- A publication record names the repository its commit and horizon were
  verified in: the origin URL without credentials, `.git` or a final slash
  (`destinationOf`).
- When `archive.workspace` points elsewhere, a record of the old
  repository becomes **`unverified`**, never `current`.
  - Its error says where it was published and that it has not been checked
    in the new repository.
  - Its link still names the repository it was verified in.
  - A record written before v8 has no destination: it is unverified and
    gets no made-up link.
  - It is **due**, so the reconcile handles it within a minute. The export
    reads the new repository's own manifest and appends only beyond that
    horizon. After an import that appends nothing; it records the commit
    found there.
- The writer's clone is disposable. A clone whose origin is another
  repository is **deleted and cloned again**, not re-pointed, so nothing
  from the old repository travels with it. The lock, the retry/backoff and
  the single writer are unchanged.
- No parallel publisher and no new service: the configurable
  `archive.workspace` is the only switch.

The summary card on Home shows `archive: unverified` instead of a link.
`wfe discussion archive --status` prints the configured repository and the
state. aghangout says "Archived through #n (commit) in <repository>; not
yet checked in the current archive repository" (pj-aghangout `545d9f8`).

## argue-specific assumptions removed

- **WFE** (`7893d75`):
  - module header, comments and the config example name ponderer or the
    configured archive project;
  - `wfe discussion help`: "the archive project's", "verified in";
  - the Home link title;
  - `docs/discussions.md`: configuration, participant rule (one discussion
    workflow per workspace; other workflows by delegation), archive layout,
    the *Destination* paragraph, status states, hand-off and reuse;
  - the format name `ag.argue.discussion.v1` is documented as a kept name.
- **aghangout** (`545d9f8`):
  - the `argue` character (described as "the facilitator") is replaced by
    `ponderer` (the conductor). argue's historical lines play as a plain
    speaker under their recorded participant;
  - the fixture's facilitator is `ponderer/conduct`;
  - the browser check expects `ponderer-agent`/`Ponderer`;
  - README; the `unverified` wording.

## Focused tests

New `test/archiveDestination.test.ts` (4 tests, all pass):

| Test | Covers |
| --- | --- |
| switch to an imported history | closed and open archives. A record is `unverified` until read in ponderer. The export appends nothing and records commits that exist in ponderer. The clone origin is now ponderer. An old argue commit resolves in ponderer. The closed transcript is byte-identical. A later message lands only in ponderer, once, and argue's main does not move |
| switch to an empty destination | the whole record is exported there once. An interruption after the push leaves `failed`, and the replay adopts the push and appends nothing. A concurrent `summary.md` integration by another writer is kept |
| stale clone, pre-v8 record | a record with no destination is unverified with no link, then verified. A stray file in the old clone does not reach the new repository |
| `destinationOf` | credentials, `.git` and a final slash do not make another repository |

- WFE full suite: **480/480 pass** (`npm test`, 201 s), the 475 earlier
  tests and the new ones.
- aghangout: `npm run check` 13/13, including the new `unverified` case.
  `npm run browser-check` (WebKit, fixture with ponderer as facilitator):
  all checks passed.
