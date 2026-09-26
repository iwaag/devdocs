# sage p2 — step 3 report: sage attachment, refresh and persistence

## What changed (archsage `b57f71a`, `a623066`)

- **A sage records its study:** `sage.toml` now carries `project` (the
  study's slug), `source` and `study` (the repository its tree is cloned
  from). `source` is one of:
  - `main`: the study's internal knowledge repository, current as soon as
    research is integrated;
  - `publish`: a public repository the developer supplied and pushes after
    review.
- **Commands** (all explained in `archsage --help`):
  - `archsage sage attach <name> --project <slug> [--source main|publish]
    [--repository <url>]` gives an existing sage a study. `main` is the
    default and is resolved from the study's internal repository on the
    host's Gitea (`agag.project.gitea_head`). If that repository does not
    exist, the command points at `agproject status`. `publish` needs the
    repository given explicitly. The repository is checked with
    `git ls-remote` **before** the definition changes, so an unreadable one
    is refused and the sage keeps its source. The tree is synced at once,
    and the revision and whether the study has findings yet are reported.
  - `sage add … [--project <slug>]` defines a sage that is already
    attached.
  - `sage update <name> [--about] [--guide-file]` changes an existing sage.
  - `sage show`, `sage remove` (the directory is moved aside and nothing is
    deleted) and `sage list` now include study, state and queue size.
- **Sync checks the origin.** A tree cloned from another repository than
  the definition names is replaced. The new repository is cloned beside
  the tree and swapped in **only once the clone succeeded**, and the old
  tree is kept under `.local/replaced-trees/`. The old `sync_sage` just
  pulled the old clone. A failed refresh leaves the tree where it was and
  says so (`the tree stays at <rev>`).
- **Findings vs scaffolding.** A tree holding only a plan, READMEs and
  indexes reports "no findings yet". The sage is told this, as is archsage
  in its placement, and so is the introduction. The plan is what the study
  intends to find, not a finding.
- **Durable definitions.**
  - `sages/` is no longer part of the public archsage repository. It is a
    checkout of the private internal repository `autodev/archsage-sages`,
    configured in the ignored `.local/sages-store.toml` (`url`,
    `token_file`, `username`, `author`).
  - `add`, `update`, `attach` and `remove` commit and push there.
  - Trees and queues are ignored by the store's own `.gitignore`.
  - `archsage store status | restore` shows the store and rebuilds a
    checkout.
  - The four existing definitions are its first commit, including the two
    that were untracked (`protoprey`, `worldtrend`). `arxiv` is recorded as
    `studyarxiv`/`publish` and `protoprey` as `protoprey-research`/`main`.
- **The introduction is refreshed after every definition change** (unless
  `--no-intro` is given), and `archsage intro` posts it on request. Both
  are reachable through the existing `Bash(archsage:*)` grant.
- **Queues.** `archsage queue list [<sage>] | show | resolve <sage> <note>
  --answered-by <path>…`. `resolve` accepts only files that exist inside
  the sage's **current** tree. It logs the note text, the revision and the
  paths to `.local/queue-resolved.jsonl`, and only then removes the note.
  Asking for research settles nothing.
- The prompts and docs that called every tree "published knowledge" were
  updated: the shared sage guide, the sage and council context, the README
  (with a section on which repository a sage reads, where definitions live
  and how to rebuild a machine), `sagetree`'s empty-tree line and
  `sync_knowledge.sh`. archsage's own guide and introduction are rewritten
  in step 4.

## Verification

- **Tests:** archsage `tests/test_studies.py` covers:
  - an existing sage attached, synced ("no findings yet") and refreshed
    after a pushed finding, with the new revision and findings count, and
    the store and intro hooks called;
  - reattachment replacing the tree;
  - a refresh failing after the source went away, with the tree kept;
  - a new source that cannot be cloned never costing the old tree;
  - an unreadable repository refused at `attach`;
  - `main` resolved from the study and `publish` needing a repository;
  - a queued question refused while the tree lacks the answer (and for a
    path outside the tree), then resolved and logged after a refresh;
  - definitions pushed to a store (trees excluded), an unchanged persist,
    and a restore into a fresh directory;
  - `remove`.

  A test fixture keeps every CLI test away from the real store and the
  real board. Suite: 37 passed.
- **Live** (throwaway sage `p2probe`, removed afterwards; store commits
  `6eeca46` … `b3dc04d`):
  - It was defined with no study.
  - It was attached to `protoprey-research` main: `empty → 89c52402f296;
    4 knowledge file(s)`.
  - It was reattached to `worldtrend` main: `89c52402f296 → 50e841c913a4;
    no findings yet; replaced the tree from …protoprey-research.git`.
  - **Failure found:** reattaching to an unreadable repository first
    reported the refresh failure but left the sage with no tree, because
    the old tree had been moved aside before the clone. The attach was also
    accepted. After the fix, the same command exits 1 with "cannot be
    read; p2probe stays attached to study `worldtrend` …" and nothing
    changes.
  - `archsage store restore` into a scratch directory rebuilt all four
    definitions with their project and source.
- **Deferred to step 5:** intro discovery of a new sage, and a sage
  answering with files from the expected revision. Both need the rewritten
  introduction and a study with findings.

## Decisions and handoff candidates

- **The store's push credential is the existing agents' Gitea identity**
  (`autolab-agent`, whose token file is named in the ignored config). A
  Gitea account of archsage's own would make its commits' pusher honest
  (the commits are authored `archsage`). Creating one is account work left
  to the Developer — **handoff candidate**.
- The Omni Agent created `autodev/archsage-sages` (private) and made its
  first commit by hand. That was a one-time bootstrap; afterwards archsage
  persists by itself — **did X for agent archsage (store bootstrap)**.
