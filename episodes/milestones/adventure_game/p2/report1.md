# Step 1 report — the existing paths and the reference contract

Plan: [plan.md](plan.md) step 1. 2026-09-22, by the Omni Agent. Nothing was changed
in code or configuration during this step; it is a reading and a decision.

## Environment as found

| Check | Result |
|---|---|
| `nctl status` | Nautobot 3.1.3 reachable and authenticated, 1 celery worker, 0 pending jobs, all five submodules clean |
| Observation age | agstudio, agautolab1, aghub, agpc, agdnsmasq collected ~4.6 h before this step; agbach 16.8 h; agfixture stale (known) |
| Local Gitea | 1.27.1 on `autodev-gitea` (`:3000`, ssh `:2222`); **one user**, `autolab-agent` (site admin); one org, `autodev`; every repository public (anonymous read on the LAN) |
| agautolab1 | reachable through the ansible key; its checkout is at `b38a6af` (older deployment, as explicit_reply p1 left it) |
| SwarmUI | answers `GetNewSession` (0.9.8.3); forge's image route is live |
| ProtoPrey | `autodev/protoprey` at `v0.1.0` (`205037d`); the Developer's own clone at `~/projects/protoprey` is at the same commit, unplayed as far as any record says; `direction/` holds only `.gitignore` |
| Zulip | the listeners were not touched; `agentchat` from this shell needs an agent's credentials file, so reads in this step went through the repositories and records instead |

## Where references are lost today

Read: `project_pattern.md`, `project_init.py`, forge `knowledge.py` and its four guides,
autolab's workplan and workrun guides, Front's three guides, archsage's `sages.py`,
`sagetree.py`, `roles.py`, the p1 reports and the study_import report.

1. **A human-accepted asset has no home.** forge delivers a presigned link (60 min
   default); the p1 meadow (a7560 B) was judged from that link and never imported, and
   forge does not re-serve its own results. The project carries a remake.
2. **The only carrier between agents is prose.** Front writes `GOAL.md` from message
   ids; autolab writes `task[N].md` from the workplan; forge plans from
   `required_items.md`, which *quotes* the requester. Nothing in this chain names a file
   in a revisioned tree, so a worker can only read another agent's summary.
3. **Front is told not to open repositories** ("Do not open project repositories or
   nctl yourself"), and has no reader for anything outside its own workspace. It could
   not check a reference if the human named one.
4. **forge's `knowledge show` reports binaries without content**, and its two sources
   (`mediagen`, `localize`) are autolab's live checkouts: a mission that pulls them moves
   the revision forge cites. Precedence is written for technical guidance (chat >
   required_items > local > general); creative direction has no place in it.
5. **archsage's sage is bounded to one study tree** (`SAGETREE_ROOT`); the council role
   has `Read`. `sync_sage` fast-forwards; nothing pins a revision.
6. **autolab's project workspace** may add folders (pattern doc) and has a Gitea route
   for *its* repositories (`init-repo`), but no route to a repository it does not own and
   no notion of a pinned foreign revision. Every autolab role already carries `git`.
7. **Ownership**: the single Gitea account is autolab's, and it is a site admin. A
   human-owned repository does not exist yet; a human edit today would be a push with
   the agent's token, indistinguishable from an agent's.

## The reference contract (decided)

**Identity.** A reference is `<source>@<revision>:<relative path>` — for example
`protoprey-refs@3f2a1b0:scenes/meadow/composition.png`. `<source>` is a short name;
`<revision>` a git commit (short form allowed, the full one is what a snapshot is stored
under); the path is relative to the repository root, so the human's folder structure
*is* the structure agents see. A source without a path names the whole tree at that
revision. `<source>@<revision>` with no path is what a production unit *adopts*.

**Where the name resolves.** Each consumer maps `<source>` to a repository URL in an
ignored local file (`.local/refs.toml`, the shape forge's `knowledge.toml` already has:
name, url, about). URLs are host facts and stay out of committed files; the *name* and
the *revision* are what travel in conversations, plans, tasks and reports.

**Authoring → publication.** One human-owned repository per reference set, on the
existing local Gitea under a human account (to be created in step 2), cloned into the
human's own working folder outside every agent workspace. Publication is `git push`.
Drafts stay in an ignored `drafts/` folder. No metadata format: ordinary files and one
short README. Human-selected generated assets go into the same repository *by the
human's own commit* — authorship is the commit's author, adoption is the file being
there.

**Consumption.** A small shared resolver, `agrefs`, in pyagag (the package every
agent already carries for `agentchat`), reached under a `Bash(agrefs:*)` grant on any
harness:

    agrefs list                          every source: url name, cached revisions, latest
    agrefs sync <source> [--rev <r>]     fetch, and lay out an immutable snapshot of that revision
    agrefs show <source>@<rev>:<path>    text, or a directory listing, or "binary: <type>, <bytes>, at <path>"
    agrefs path <source>@<rev>[:<path>]  the absolute path of the snapshot — what a Read tool or a script takes
    agrefs search <source>@<rev> <terms> lines matching every term
    agrefs revision <source>             latest published revision (fetches)

A snapshot is `git archive` of one commit into `.local/refs/<source>/<full sha>/` beside a
mirror clone; several revisions coexist, so a running task keeps its revision while a
new one is adopted elsewhere. Image bytes are in the snapshot, so nothing depends on a
delivery link. Visual access is the harness's own image reading over `agrefs path`
(claude_code's `Read` renders images); `show` never pretends to have read a binary.

**Roles that get it.** Front (`front`, `desk`, `argue`), autolab (every working role),
forge (`front`, `generator`, `argue`), archsage (`archsage`). The sages keep their one
bounded reader over their study tree: creative references are a project's input, not a
study's publication, and the council is the one that reads across.

**Carrying it.** The project request names the adopted `<source>@<rev>`; `GOAL.md`
carries it; the workplan's tasks name the paths each task works from; forge's
`required_items.md` names the reference images and `agforge image generate` gets an
`--init-image` so a reference composition can steer SwarmUI (its API takes `initimage`);
a delivery names the reference it used. Interpretation and decisions go to
`direction/` (including `direction/REFERENCES.md`: source, adopted revision, why, since
which unit); derivatives to `main/`; evidence to the ordinary work records.

**Ownership boundary.** Agents read the human repository anonymously (it is public on
the LAN, like every repository there) and never hold its push credential; they write
derivatives into their own workspaces and project repositories. That autolab's token is
a site admin is noted and left: repository-scoped read is what is practical here, and
the plan says not to make an isolation project of it.

## Not adopted

- Copying references into `direction/` or `main/` of the generated project: it would
  make the human edit an agent's repository and lose the originals' identity.
- A shared absolute path or network mount: agautolab1 is another machine.
- Per-file metadata or a manifest format: the README and the folder structure suffice
  for a few examples; a manifest can come when the trial shows it is needed.
- A third forge knowledge kind: forge's knowledge is *how to make media*; a creative
  reference is *what to make*, and one resolver for all four roles beats four readers.

## Ready to build (step 2)

1. Gitea: user for the human, repository `protoprey-refs`, human clone outside the agent
   workspaces, README, `.gitignore` (`drafts/`, `.local/`).
2. pyagag: `agrefs` with tests; console script; `.local/refs.toml` example.
3. Pin the new pyagag in agfront, agautolab, agforge, archsage; grants; guides (step 3).
4. Independent retrieval check from agautolab1 through the ansible key.

## Cost of the step

No agent run was bought. Omni Agent only.
