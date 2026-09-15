# Step 2 — the `studyindustry` project

Date: 2026-09-15 JST (timestamps UTC where they come from Zulip or run
records). Plan: [plan.md](plan.md) section 2. Previous:
[report1.md](report1.md).

The project exists as three surfaces — the Zulip channel, one internal Gitea
repository, and the workspace with its two clones — plus the developer's
pre-existing GitHub publication repository. One autolab serving built the
workspace; the two acts that cannot be done from inside the system were
done as the Developer.

## Manual setup, and the handoff note it owes

1. **The project channel.** `#pj-studyindustry`, stream **163**, created
   from the Developer credential with the four participants every other
   study project channel carries: Developer (8), autolab-agstudio1 (11),
   Front (15), Opsroom Observer (22). No folder was passed, so that the
   documented self-filing could be observed.
2. **The pattern marker.** `README_PROJECT.md` written into the new
   workspace root, labelled as setup rather than knowledge, declaring the
   study pattern, the subject and the two requested folders. This is what
   makes project initialization return `pattern-managed, folders untouched`
   on the first serving instead of scaffolding the fixed layout before the
   request is read.

> **Deus Ex Machina note:** *did the workspace pattern marker and the
> project channel creation for agent autolab — handoff candidate.* (A
> `pj-` channel is opened by a human by design; the marker is the part
> that could move.)

## The setup request and what autolab did with it

`#pj-studyindustry` › `workplan-setup-studyindustry-workspace`, message
**7012** (10:09:14Z), as the Developer with no `AGENTCHAT_HOME`, so no root
note. It asked for the layout in the plan's table, the standard internal
route for `main/`, a clone of the supplied GitHub URL for `publish/` with
its history inspected and preserved, the `.local/` rule stated and
actually working, the publication handling recorded, and no research.

autolab answered in **one serving** (message 7014, 10:10:07Z) and did the
setup directly rather than planning a mission — its own reading, stated in
the reply, was that a workspace-preparation request is not a mission, the
same call it made for `studyrealworld` and the opposite of the one it made
for `studyuspolitics` (one bounded task there). Both are defensible; the
result is what matters and it is verified below.

| Role | Record | Duration | Turns | Cost |
|---|---|---|---|---|
| `superdirector` | `run-0201` | 52.7 s | 19 | **$0.3011** |

`claude_code` / `sonnet` / `anthropic/claude-sonnet-5`, `exec_source:
default`, `outcome: done`.

## Verified against the tree, not the report

| Claim | Verification |
|---|---|
| `main/` identity | `origin` = the internal Gitea `autodev/studyindustry.git` — the standard route, not the GitHub URL. |
| `main/` initial revision | `787c1c0` "Initialize main working knowledge repository"; `git ls-remote origin` returns `787c1c0c…` for `HEAD` and `refs/heads/main`, so it is pushed. Working tree clean, level with `origin/main`. |
| `main/` contents | Exactly four files: `.gitignore`, `README.md`, `methods/.gitkeep`, `reports/.gitkeep`. |
| Ignore rule works | `.gitignore` is the one line `.local/`; `git check-ignore -v` on a probe file under `main/.local/` matches it. |
| No imported layout | No `sources/`, no `INDEX.md`, no report template; `main/README.md` says file names, subdirectories, index format and report units are the researcher's to choose. |
| `publish/` identity and state | `origin` = `https://github.com/iwaag/study-industry.git`; single commit `a91105c` on `main`, `LICENSE` only; clean and level with its origin; **no new commit, nothing pushed**. |
| Separation | Two origins, two independent histories; `publish/` is not `main/`'s origin. |
| Marker survives | `README_PROJECT.md` is present at the workspace root, the condition `project_init` checks. |
| Channel filed itself | After the serving, `pj-studyindustry` sits in a new channel folder `pj-studyindustry` (id **18**), minted by the agent; the realm now has folders 3, 5, 6, 7, 10, 12, 13, 17, 18. |
| No stray `devlog/` | The workspace holds only `README_PROJECT.md`, `main/`, `publish/`. |

`README_PROJECT.md` as rewritten by autolab records: the subject (first
subject the video game industry, not limited to it); `main/`'s repository,
initial revision and the `.local/` rule ("private host facts, credentials,
downloads, database files, caches and execution logs"); `publish/`'s
repository, what its history held at clone time, that only the `publish`
routine populates it by fixing in `main/` first and copying, and that
agents commit it locally and **never push** it; and that the two are
independent repositories. It ends with "No research has been done yet".
That is the standing publication contract the plan asks for in step 6,
already in place; step 6 re-checks it rather than writing it.

`main/README.md` restates the two divisions and the publish-ready rule, so
a reader with only the repository has the conventions. That matters because
`README_PROJECT.md` sits outside any repository — tracked nowhere, as with
every study project.

## Observations

- **The setup topic is unresolved.** autolab does not tidy on its own; the
  setup conversation is not a mission and has no `[mission]` note, so there
  is nothing to mark done. Left as is, like `workplan-create` and
  `workplan-setup-studyuspolitics-workspace` before it.
- The reply names the internal Gitea host; that is fine in Zulip and in
  the ignored `README_PROJECT.md`, and nothing of it is in `main/`.

## Step 2 conclusions

1. The workspace exists at the requested layout; `methods/` and `reports/`
   are the only research-content directories in `main/`, and the ignore
   rule for `.local/` is present and effective.
2. `main/` is an independent internal repository, committed and pushed;
   `publish/` is the supplied GitHub clone with its one-commit CC0 history
   intact and untouched.
3. Both origins and the publication handling are recorded in
   `README_PROJECT.md`.
4. Nothing about the research has been asked for or done. One paid run,
   $0.30.
