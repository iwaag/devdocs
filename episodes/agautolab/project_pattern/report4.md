# Step 4 — the live test

Four servings, all `superdirector` / `claude_code` / `anthropic/claude-sonnet-5`,
all `"outcome": "done"`:

| run | topic | turns | wall | cost |
|---|---|---|---|---|
| `run-0082` | `#pj-studyarxiv` › `workplan-create` | 11 | 28.2 s | $0.134 |
| `run-0083` | `#pj-studyarxiv` › `workplan-zine` | 10 | 37.8 s | $0.118 |
| `run-0084` | `#pj-studyarxiv` › `workplan-nourl` | 8 | 25.7 s | $0.095 |
| `run-0085` | `#pj-studynourl` › `workplan-create` | 8 | 20.2 s | $0.119 |

`.local/out/zulip-listener.log`:

```
2026-08-26T07:38:22Z serving 'pj-studyarxiv'/'workplan-create'
2026-08-26T07:38:22Z workplan topic 'pj-studyarxiv'/'workplan-create'
2026-08-26T07:39:26Z serving 'pj-studyarxiv'/'workplan-zine'
2026-08-26T07:39:26Z workplan topic 'pj-studyarxiv'/'workplan-zine'
2026-08-26T07:40:32Z serving 'pj-studyarxiv'/'workplan-nourl'
2026-08-26T07:40:32Z workplan topic 'pj-studyarxiv'/'workplan-nourl'
2026-08-26T07:41:24Z serving 'pj-studynourl'/'workplan-create'
2026-08-26T07:41:24Z workplan topic 'pj-studynourl'/'workplan-create'
```

Two lines per serving throughout: no `init_project` step, no Plane read-back,
for either pattern-managed project.

## Setup

`#pj-studyarxiv` and the marker file are from Step 3 (report3 has the
`create_channel` call and the probe serving). The `publish` URL was confirmed
with the Developer before posting: the braindump's
`https://github.com/iwaag/study-arxiv-torend` is a typo for
**`study-arxiv-trend`**, which already exists on GitHub and is not empty
(`gh repo view iwaag/study-arxiv-trend` → `"isEmpty":false`), so nothing had
to be created by hand. The Developer also corrected the braindump itself.

## The request (message 2410, `#pj-studyarxiv` › `workplan-create`)

Posted as the Developer, the braindump's text verbatim with the confirmed URL:

> studyパターンプロジェクトを、publishリポジトリをhttps://github.com/iwaag/study-arxiv-trend、mainリポジトリをhttp://agstudio:3000/autodev/papers.gitにして作って。

## The reply (message 2412)

> Project set up as the "study" pattern: `main/` cloned from
> `http://agstudio:3000/autodev/papers.git` and `publish/` cloned from
> `https://github.com/iwaag/study-arxiv-trend`, with `README_PROJECT.md`
> updated to document both. No plan/task files were created since this was
> project preparation, not a new mission.

## Success checklist

### 1. Both folders exist as clones of exactly the given URLs — **pass**

```
$ git -C .local/projects/studyarxiv/main remote -v
origin	http://agstudio:3000/autodev/papers.git (fetch)
origin	http://agstudio:3000/autodev/papers.git (push)
$ git -C .local/projects/studyarxiv/publish remote -v
origin	https://github.com/iwaag/study-arxiv-trend.git (fetch)
origin	https://github.com/iwaag/study-arxiv-trend.git (push)
```

`main/`'s remote is the given URL character for character, `agstudio:3000`
host included — the agent did not "helpfully" normalise it to
`agstudio.local:3000`, which is the form `project_init` uses everywhere else
and which the mDNS-quirk note in the local memo would have made tempting.
`publish/`'s remote has a `.git` suffix the request did not have; that is
`git clone` recording what it was given plus the conventional suffix, resolves
to the same repository, and is the only deviation.

### 2. No new Gitea repository was created — **pass**

```
$ …/api/v1/repos/autodev/studyarxiv          → HTTP 404
$ …/api/v1/repos/autodev/studyarxiv-publish  → HTTP 404
$ …/api/v1/repos/autodev/study-arxiv-trend   → HTTP 404
```

Both URLs were given, so nothing needed creating and nothing was created. The
`autodev` org has the same repositories it had before this step (the two
`studynourl*` repositories below come from the negative test, not from this
request).

### 3. `README_PROJECT.md` — **partial pass**

Written by the run (409 bytes, replacing the 80-byte hand-written marker):

```
This project follows the "study" pattern.

- `main/` ... workspace where summaries of knowledge are stored.
  Repository: http://agstudio:3000/autodev/papers.git
- `publish/` ... only reviewed summaries of knowledge, moved here from
  `main/` on the developer's explicit approval. Never push `publish/`;
  the developer pushes it by hand after review.
  Repository: https://github.com/iwaag/study-arxiv-trend
```

- describes both folders — **yes**
- names the study pattern — **yes**
- carries the publish/no-push rule — **yes**, and in the pattern document's own
  words
- reflects `main/`'s **existing** content — **no.** `main/` is the p4 digest
  repository: a `README.md` explaining the `papers` and `manual` routines, and
  `papers/` holding `INDEX.md` plus four paper directories (`2608.20430`,
  `2608.21156`, `2608.23283`, `2608.23552`). None of that reaches
  README_PROJECT.md. "workspace where summaries of knowledge are stored" is
  the pattern document's sentence with the second person removed, not an
  observation.

  The run *did* read the clone — the `workplan-nourl` serving an hour later
  described `main/` as "already populated with paper summaries" without being
  told — so this is not a blind write. It is the pattern text winning over what
  was on disk when both were available. That is the sharpest finding of this
  step and is carried into Step 5: the guide says "create it to explain how
  each folder works", the pattern document supplies a ready-made sentence per
  folder, and the agent takes it. If the intended README is a description of
  *this* project rather than a copy of the pattern, something has to say so.

### 4. Nothing pushed anywhere; `publish/` clone clean — **pass**

```
$ git -C .local/projects/studyarxiv/main status --short --branch
## main...origin/main
$ git -C .local/projects/studyarxiv/publish status --short --branch
## main...origin/main
```

No local commits ahead of either origin, nothing uncommitted, nothing pushed.
`publish/` holds exactly what GitHub holds (`LICENSE`, one commit
`fbf6517 Initial commit`). `README_PROJECT.md` sits in the workspace root,
outside both clones, so it is not staged anywhere either.

`autodev/papers` is untouched — `.local/projects/papers/main` (the routine's
own clone, a different checkout) is still level with `origin/main`, and
`studyarxiv/main`'s log still ends at `d2a5a25`, the same head the routine
last pushed. The `papers` routine keeps running against its own clone,
undisturbed.

### 5. The reply reports what was made and stops — **pass**

No mission invented, no `plan.md`, no `task[N].md`, no `start.flag`. The reply
says what it cloned, says it updated README_PROJECT.md, and explicitly says
why it wrote no plan: "this was project preparation, not a new mission" —
which is the guide's own split, in the agent's words. The pattern-managed
Plane notice from Step 3 never fired, because there was nothing to notice.

**Verdict: 4 of 5 clean, one partial** — the workspace is exactly right, the
README is right about structure and rules and silent about content.

## Workspace tree after the request

```
.local/projects/studyarxiv/
├── README_PROJECT.md          (written by the run)
├── main/                      → http://agstudio:3000/autodev/papers.git
│   ├── .gitignore
│   ├── README.md
│   └── papers/
│       ├── INDEX.md
│       ├── 2608.20430/
│       ├── 2608.21156/
│       ├── 2608.23283/
│       └── 2608.23552/
└── publish/                   → https://github.com/iwaag/study-arxiv-trend.git
    └── LICENSE
```

Unchanged by the two negative tests that followed.

## Negative test 1 — a nonexistent pattern

`#pj-studyarxiv` › `workplan-zine`, message 2413:

> zineパターンのプロジェクトを作って。

Reply (message 2415):

> This request is about project structure, not a new mission, and
> `autolab doc patterns` doesn't define a "zine" pattern — only "study" and
> "game" exist. This project (studyarxiv) is already set up as "study" pattern.
>
> Could you clarify what you mean by "zineパターン"?
>
> - Add a folder to studyarxiv's existing "study" pattern (e.g. a place for
>   zine-style output)?
> - Define a brand-new "zine" pattern — if so, what folders should it have and
>   what's each for?
> - Start an entirely separate project using this new pattern?
>
> Once I know the folder structure, I'll update `README_PROJECT.md` and set
> things up accordingly.

**Asked back, created nothing** — the workspace was byte-identical before and
after. It named the command it consulted and the two patterns that do exist,
and its three readings of the request are all real ones. Exactly the guide's
"specified nonexistent pattern, just ask back in your reply".

## Negative test 2 — a study request with no repository URLs

Run twice, because the first run was confounded.

**First attempt (confounded).** `#pj-studyarxiv` › `workplan-nourl`, message
2416, "studyパターンのプロジェクトを作って。" — but `#pj-studyarxiv` was
*already* a fully set-up study project, so the branch under test never came
up. Reply (message 2418):

> The project already exists and is fully set up with the "study" pattern:
>
> - `main/` — a cloned repo (`autodev/papers.git` on local Gitea) already
>   populated with paper summaries.
> - `publish/` — a cloned repo (`github.com/iwaag/study-arxiv-trend`) for
>   reviewed summaries, currently just the initial commit.
> - `README_PROJECT.md` documents both folders per the pattern.
>
> Nothing needs to change — the workspace is already pattern-managed and
> untouched … I won't edit any files since no new repos or folders are being
> added and the management approach hasn't changed.

Created nothing, correctly, and quoted the guide's edit-only-when rule almost
verbatim. Also — worth noting — it read `main/`'s contents here ("already
populated with paper summaries"), which is how we know the earlier
README_PROJECT.md finding is about writing, not reading. It also read
`agautolab`'s git log and cited the Step 3 commit message; the workspace sits
inside that (gitignored) checkout, so its history is reachable. Harmless here,
but a pattern-managed workspace is not as isolated as it looks.

**Clean re-run.** A fresh pattern-managed project, declared the same way as
studyarxiv — `#pj-studynourl` channel, marker file, nothing else:

```
$ mkdir -p .local/projects/studynourl
$ cat .local/projects/studynourl/README_PROJECT.md
pattern-managed; folders are created by the workplan on the developer's request
```

`#pj-studynourl` › `workplan-create`, message 2419, the same text. Reply
(message 2421):

> studyパターンでプロジェクトを準備しました。`main/`(`autodev/studynourl`)と
> `publish/`(`autodev/studynourl-publish`)の2つのリポジトリを作成・clone し、
> `README_PROJECT.md` を更新しました。

**It used `autolab project init-repo` with the standard names** — the second of
the two acceptable outcomes, and it matches the pattern document's "if not
specified" line exactly:

```
$ git -C .local/projects/studynourl/main remote -v
origin	http://agstudio.local:3000/autodev/studynourl.git (fetch/push)
$ git -C .local/projects/studynourl/publish remote -v
origin	http://agstudio.local:3000/autodev/studynourl-publish.git (fetch/push)
$ …/api/v1/repos/autodev/studynourl          → HTTP 200, empty: true
$ …/api/v1/repos/autodev/studynourl-publish  → HTTP 200, empty: true
```

`autodev/<project>` for `main/`, `autodev/<project>-<folder>` for the other —
the naming Step 1 took from `project_init`, reached through the CLI, with the
Gitea token never leaving the subcommand. It did not ask back, and did not
need to: the pattern document answers the question the request left open,
which is what that line is for.

Its `README_PROJECT.md`:

```
pattern-managed; folders are created by the workplan on the developer's request

This project follows the "study" pattern (see `autolab doc patterns`).

- `main/` ... workspace where you store summaries of knowledge.
  Repository: `autodev/studynourl` on the local Gitea.
- `publish/` ... only reviewed summaries of knowledge, moved here from
  `main/` on the developer's explicit approval. Never push `publish/`;
  the developer pushes it by hand after review.
  Repository: `autodev/studynourl-publish` on the local Gitea.
```

It **appended** to the hand-written marker line rather than replacing it (the
studyarxiv run replaced it). So the marker sentence, which is a declaration
to `init_project` and not documentation, is now the first line of the
project's README — a small ugliness that belongs in Step 5's "what should the
declaration become".

Both clones are on an unborn branch: `## No commits yet on main...origin/main
[gone]`. That is report1's noted consequence arriving on schedule —
`init-repo` deliberately neither pushes nor seeds `.gitignore`, unlike
`init_project`, so a repository it creates is empty and its clone has no
branch until something is committed. Nothing broke, and the agent said nothing
about it; whether an empty clone is a problem will show the first time
something is committed into one.

## Push credentials

Nothing tried to push, so the by-design absence of GitHub credentials for
agents was never exercised. The checklist's "nothing pushed" holds by the
agent's own choice, not by a failed attempt — a weaker result than a refused
push would have been, and worth remembering when the review workflow that
moves summaries into `publish/` is built.

## Artifacts

- `#pj-studyarxiv`: messages 2407–2418 across `workplan-setup` (Step 3 probe),
  `workplan-create`, `workplan-zine`, `workplan-nourl`.
- `#pj-studynourl`: messages 2419–2421, `workplan-create`.
- Run records `.local/agent/superdirector/run-0081.json` … `run-0085.json`.
- Serving workspaces under `.local/topics/pj-studyarxiv/` and
  `.local/topics/pj-studynourl/`; none of them holds a `current/`.
