# Challenge p3 — Step 1: repository reading and revision semantics

Code: agdevworld `73eb720` (`workflow_editor/server/challenges.ts`,
`docs/challenges.md`). Steps 1 and 2 share one commit, because both live in
the same store; this report covers reading, [report2](report2.md) covers
writing and access.

## What a challenge is now

```text
challenges/<id>/
  summary.md          optional standard statement (frontmatter + free body)
  <any files>         source material, any names and folders
  wedo/<trial>/…      one directory per trial; summary.md optional
```

- **Discovery by directory.** Every `challenges/<name>/` with at least one
  tracked file is a challenge, with or without `summary.md`. WFE allocates
  `ch-NNNN` for what it creates; a directory made by hand with another plain
  name (letters, digits, `. _ -`) is read too (`steam-game` in the tests).
  Run inputs and message attachments accept such ids.
- **The summary is an optional reading.** Without it the title is the id,
  the state is null, and the files are still listed. A summary that cannot
  be parsed is still shown as written; `problem` names why.
- **States.**
  - The standard vocabulary is `open`, `resolved` and `withdrawn`.
  - Any other value is kept as written (`stateKind: custom`).
  - An absent state is `unspecified`. Nothing defaults to `open` any more;
    p1 did default to `open`.
  - Counts are `open/resolved/withdrawn/custom/unspecified`, plus each
    custom state's count.
  - `state=` filters by a kind or by one state exactly as written.
- **Trials.** Each direct child of `wedo/` holds its files, an optional
  summary, and its own revision, creator and last change. A trial summary's
  title is its frontmatter `title` or first heading. Nothing else is read
  from it.
- **Files** have two identifiers:
  - `name` is the path inside the challenge, and it is stable across the
    layout change;
  - `path` is the path in the repository at that commit.

## Revisions, history, pinned reads

- A challenge's revision is the newest commit touching anything under its
  directory, including ordinary Git commits made outside WFE. Before p3 it
  was the newest commit touching its file and evidence. History spans both
  layouts (`git log --no-renames` over all three pathspecs). Created stays
  with the first author across the move.
- **Pinned reads** take `at(id, commit)` from that commit's tree only:
  summary, files, trials, trial summaries, and created/updated from that
  commit's history. A test checks that a pinned read does not list a trial
  added later.
- **Old layout at old commits.** A commit that has `challenges/<id>.md`
  reads as `layout: legacy`, with `evidence/<f>` names. `text()` returns
  the old file byte for byte, so an attachment pinned before p3 verifies
  with the same sha256 (tested).
- A run receives the summary as written. A challenge without a summary is
  given a note that lists its files and does not invent a summary.

## Derived metadata and search

- The journal schema is v2. Operations gain `paths`. `records` is rebuilt
  and keeps the whole read challenge plus search text. The new `trials`
  table holds trial names and summaries. Both are rebuilt from Git whenever
  the head moves, and `reindex` rebuilds them from a fresh clone (tested).
  The journal (`operations`, `blobs`) is unchanged in meaning and separate
  from the index.
- Text search covers the challenge's id, title, summary and projects, plus
  trial names and summaries. A hit in a trial returns `matches: [{trial,
  title, excerpt}]`. Files, binary or not, are never searched, and nothing
  is read as an outcome.

## Checks

`test/challenges.test.ts` (12) passed. Reading is covered by these tests:

- direct Git writes: a trial without a summary, a custom state with extra
  frontmatter, an absent state, a directory without a summary, and
  unreadable YAML. The test also checks counts and filters;
- the old layout at its commit, then a move by a plain commit, with history,
  creator, pin hash and file names;
- a fresh-clone rebuild;
- trial search.

`npm run check` passed: tsc, 538/538 tests and the build.
