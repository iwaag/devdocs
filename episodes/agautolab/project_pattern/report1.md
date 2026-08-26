# Step 1 — the `autolab` CLI

The guide already said "the command `autolab doc patterns`" and every role's
grant already carried `Bash(autolab:*)`; there was no command. There is one
now: `[project.scripts] autolab = "agautolab.cli:main"` in
`agautolab/pyproject.toml`, `src/agautolab/cli.py`, `tests/test_cli.py`.

## How a serving reaches it

Nothing new had to be wired. `agag.agent.chat_environment` already puts the
directory of the interpreter running the listener on every role run's PATH —
in a `uv` project that is `.venv/bin`, which is exactly where the new console
script installs. This is the same mechanism that gives `agentchat` its name,
so the deployment path stays unwritten. Verified: `.venv/bin/autolab` exists
after `uv run pytest`.

## The surface

```
$ autolab --help
usage: autolab [-h] {doc,project} ...

autolab's own tools for the project workspace you are working in.

positional arguments:
  {doc,project}
    doc          print one of autolab's documents
    project      work on the project workspace

options:
  -h, --help     show this help message and exit

examples:
  autolab doc patterns            print how project folders are
                                  organised by pattern
  autolab project init-repo publish
                                  create autodev/<project>-publish on
                                  the local Gitea and clone it into
                                  ./publish
```

```
$ autolab project init-repo --help
usage: autolab project init-repo [-h] [--project PROJECT] [folder]

Create the standard repository for one workspace folder on the local Gitea and
clone it into the workspace. The repository is named <project> for main/ and
<project>-<folder> for anything else. Nothing is pushed. A folder that already
is that clone is reported and left alone; a folder that is anything else is
refused.

positional arguments:
  folder             workspace folder, for example publish

options:
  -h, --help         show this help message and exit
  --project PROJECT  project slug; by default the workspace the working
                     directory is in
```

`autolab` with no arguments prints the same top-level help and exits 0 — an
agent that types the name alone gets the documentation, not an error. `autolab
doc` with no name and `autolab project` with no subcommand say what is
missing. `autolab doc <unknown>` lists the known documents and exits 1.

The help *is* the documentation the agent gets (Tool Giving): what the command
does, how repositories are named, and — the part an agent otherwise has to
guess — that nothing is pushed and that an occupied folder is refused rather
than written into.

## Project resolution

`project_settings.project_name_from_workspace(cwd)` is new beside the existing
`project_name_from_direction`: the workspace root itself counts, because that
is the cwd of both the superdirector and the supercoder runs
(`zulip_listener.project_directory`). Outside `.local/projects/` it returns
`None` and the command says to pass `--project <slug>`.

Repository naming reuses `project_init`'s convention exactly — `<project>` for
`main/`, `<project>-<folder>` for anything else — and the plumbing is
`project_init.load_gitea_config`, `ensure_gitea_repo`, `ensure_clone`, i.e.
the askpass path with `AUTOLAB_GITEA_TOKEN_VALUE`, unchanged.

## Token handling

The token is read inside the process by `load_gitea_config` and handed to
`git` through the existing askpass environment variable. It is never in argv,
never in a printed URL (the clone URL is plain `http://…/autodev/<repo>.git`),
and never in an error message — errors quote the folder, the remote it was
found to be, and the remote that was wanted. `test_the_gitea_token_never_reaches_the_output`
asserts this across four command lines plus one failure path.

## Tests

`tests/test_cli.py`, 16 cases: the bare command, `--help`, `doc patterns`
byte-identical to `agent/project_pattern.md`, the unknown-document exit,
repository naming, `init-repo` on a fixture workspace with the Gitea API
mocked the way `test_project_init.py` mocks it, `main/`'s bare name, the
wrong-remote refusal, the already-correct clone left alone, a plain folder in
the way, running outside a workspace, four unsafe folder names, and the token
assertion.

```
$ uv run pytest -q
........................................................................ [ 36%]
........................................................................ [ 73%]
....................................................                     [100%]
196 passed in 0.83s
```

(180 before this step, 196 after.)

## Working from a real workspace

Run inside `.local/projects/papers/` — the `scheduled_routine` p4 digest
project, whose `main/` is a clone of `autodev/papers`:

```
$ autolab doc patterns | head -8
# project patterns

Each folder in the project workspace is either a cloned repository or a
plain local folder, depending on the developer's request.

- When the developer gives a repository URL for a folder, clone it there.
- When the developer asks for a new repository without naming a URL, use
  `autolab project init-repo <folder>`: it creates the repository on the

$ autolab project init-repo main
path: /Users/eiji/projects/pj-agdev/agautolab/.local/projects/papers/main
remote: http://agstudio.local:3000/autodev/papers.git
$ echo $?
0

$ autolab project init-repo devlog
autolab: /Users/eiji/projects/pj-agdev/agautolab/.local/projects/papers/devlog already exists and is not a git clone; refusing to touch it
$ echo $?
1
```

The project slug came from the cwd in both cases. Neither call touched Gitea:
the first found `main/` already a clone of exactly the repository it would
have created and reported it; the second found the main-only project's plain
local `devlog/` and refused. `autodev/papers` history is untouched and no new
Gitea repository was created by this step.

## Notes for later steps

- The pattern document tells the agent to clone a developer-given URL itself;
  `Bash(git:*)` is in every role's grant, so no CLI support is needed for
  that, and a GitHub URL never involves the Gitea token at all.
- `init-repo` deliberately does not push and does not write `.gitignore`. That
  differs from `init_project`, which seeds `.gitignore` to establish `main`.
  A repository created by `init-repo` is therefore empty, and its clone has no
  branch until something is committed. Left as is: what enters a
  pattern-managed project's history is the agent's decision, and Step 4 will
  show whether the empty clone is awkward in practice.
