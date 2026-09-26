# give_context_easier p1 — step 1 report: integration points and the reference contract

Date: 2026-09-26 (UTC 07:30). Read: the prerequisite documents, the braindump,
`preresearch.md`, pyagag `agag.refs` (`7f6095b`), the relay (agdevworld
`592dee6`), cagent's runners (pj-clusterintent), agobserver's config, and the
live Gitea.

## Environment as observed

- `nctl drift` (Nautobot, just now): **converged=46**, error 0. Agent liveness:
  `agfront`, `autolab-agstudio`, `agforge`, `agobserver-agstudio1`,
  `archsage-agstudio1`, `cagent` all `polling`; `autolab-agautolab1` `stale`.
- `nctl status`: node observation dumps are **22.1 h old** for every active
  host (agstudio, agautolab1, agpc, aghub, agdnsmasq). launchd on agstudio
  shows every listener, the relay, the autolab gateway, the forge service,
  cagent-api and the notifier running, so the stale dumps are age, not an
  outage.
- agautolab1 (checked read-only through ansible): only the autolab
  **gateway** runs there (`agent/gateway.py`, up 36 days) on agautolab
  `b38a6af`; there is no listener since 2026-08-16 and no `intro-` post on the
  `#agents` board. It is a remote execution node, not an active conversational
  instance.
- Gitea `http://<git host>:3000` answers; `developer` owns exactly one
  repository, `developer/protoprey-refs` (default branch `main`, non-empty).

## Active agent instances and roles

| instance | project | roles that read references | reaches `agrefs` today |
|---|---|---|---|
| front-agstudio1 | agfront | front, desk, argue, routine_run (present has no shell, by design) | `Bash(agrefs:*)` on front/desk/argue; `.local/refs.toml` |
| autolab-agstudio1 | agautolab | director/planning and task roles, argue | grant on every harness role; `.local/refs.toml` |
| agforge-agstudio1 | agforge | assetplan front, generators, argue | grant; `.local/refs.toml` |
| archsage-agstudio1 | archsage | archsage, sage | grant; `.local/refs.toml` |
| agobserver-agstudio1 | agobserver | front, observe (agcode, full shell), argue (Read/Glob/Grep) | **no** `refs.toml`; `agrefs` is in its venv but nothing names a source |
| cagent-agstudio1 | pj-clusterintent/cagent | Zulip `front` (agcode), `operator`, `argue` (claude_code, `Bash(cagent:*)`), in-process window `front` (curated agcode tools, no shell) | **none**: no grant, no tool, no `refs.toml`, and the window has no shell to run one |
| agautolab1 gateway | agautolab (remote) | autolab task execution through the gateway | none: old code, no `refs.toml` |

The relay (agentroom) is a seventh pyagag consumer; it reads no references
today. comfynotify is not an agent and reads nothing from references.

Provisioning: `agag init <agent> --yes --provision --like <sibling>` renders
the project from `pyagag/src/agag/templates/` and copies only
`agents.local.toml` from the sibling. The templates grant no `agrefs` and
their guide says nothing about references, so a new instance today starts
without any reference access. Remote nodes are deployed by
`ansible_agdev/playbooks/agent/setup_autolab_node.yml` (autolab) and
`setup_agag_agent.yml` (agag agents).

## Decisions: the catalog and the reference contract

**One catalog repository** holds registration metadata; content stays in its
own repositories.

- Repository: `developer/context-catalog` on Gitea (human-owned, public
  read). One file, `catalog.toml`, schema `agag.refs-catalog.v1`.
- Entry:

  ```toml
  [[source]]
  id = "protoprey-refs"            # stable; the <source> in every reference; never renamed
  name = "ProtoPrey references"    # display name, freely edited
  description = "…"                # what an agent reads to decide relevance
  repository = "developer/protoprey-refs"   # relative to the catalog's own Git host
  # url = "https://…/x.git"        # absolute alternative for a repository elsewhere
  branch = "main"
  status = "active"                # or "archived"
  ```

  `repository` is relative on purpose: agstudio and a remote node may reach
  the same Gitea under different host names (`.local` is slow on some hosts
  here), so each consumer resolves it against the URL *it* read the catalog
  from. The web link is derived the same way.
- `id` follows the existing source name rule (`^[a-z0-9][a-z0-9._-]{0,63}$`).
  A display-name or description edit changes nothing a reference carries.
  A duplicate `id` in the catalog is reported and the later entry ignored.
- **Archived** entries stay resolvable (old references keep working) and are
  left out of the default list and the picker.

**Where the catalog is configured: once per host**, not per instance.
`~/.config/agag/refs.toml` holds `[catalog] url = …`; an instance's own
`.local/refs.toml` may override it or add explicit `[[source]]` entries
(explicit entries win over a catalog entry with the same id), and
`AGREFS_CATALOG` overrides both for a single process. Every consumer on
agstudio — the six listeners, their runs, the relay — and any instance
provisioned later on the same host inherit it with no file of their own. A
remote node gets the same one-line file from its deployment role.

**The reference stays `<source>@<commit>[:<path>]`.**

- The picker resolves the selected version to the **full 40-hex commit** and
  inserts that; the panel shows the short form. `agrefs` already accepts both.
- Publishing a new revision changes what the *next* selection resolves to.
  A reference already in a post, a GOAL.md or a task keeps its commit, and the
  per-revision snapshots already make that hold on disk.
- Each consumer keeps its own cache (`<instance>/.local/refs/`): the catalog
  as a mirror clone plus its last good parse, and content as the existing
  per-commit snapshots, fetched on demand.

**Publication and credentials.**

- Destination: the `developer` account on Gitea. New context repositories are
  created there by the relay as the Developer's explicit action from the
  panel; the catalog commit follows the first publication.
- **Missing setup (human action required):** there is still no Gitea API token
  for `developer` (`pj-agdev/.local/gitea/developer.env` holds only login and
  password; checked by key names). The relay needs a token with
  `write:repository` and `write:user` scopes, stored as
  `pj-agdev/.local/gitea/developer.token` (0600) and named by the relay's
  ignored config (`agdevworld/.local/contexts.toml`). Until it exists the
  relay's write routes answer 503 naming that file, and every read, the
  catalog library and the picker are built and tested independently.
  Creating the token and the `context-catalog` repository itself is where
  this episode will stop and ask.

## Code entry points

- **pyagag `src/agag/refs.py`** — catalog loading, host config lookup,
  catalog cache and state (current / last-known / unavailable), archived
  handling, empty-repository and duplicate handling; `list --all`; a
  `catalog` verb and a small read API (`entries()`, `tree()`, `read()`) the
  relay imports. Tests: `tests/test_refs.py`.
- **pyagag `src/agag/templates/`** (`agents.toml.in`, `guide.md.in`) — new
  instances get the `agrefs` grant and a short usage section.
- **pyagag `src/agag/agent.py` `chat_environment`** — already hands
  `AGREFS_HOME`; unchanged.
- **agentroom `src/agentroom/`** — a new `contexts.py` (read model over the
  same pyagag functions, Gitea writes, publication log) and routes in
  `server.py` (`GET /contexts…`, `POST /contexts…`), config
  `agdevworld/.local/contexts.toml` with a tracked example.
- **agdevworld `src/frontDeskInput.ts`** — `insert()` at the saved caret or
  selection; **`src/scenes/FrontDeskScene.ts`** — a `context` toggle
  button and a DOM panel (new `src/contextPanel.ts`, `src/contextState.ts`)
  shared by the Front Desk and the Arguing Room adapters.
- **cagent** — `agent_runner.py` `window_tools()` gets an in-process
  `refs` tool over `agag.refs`; `role_run.py` grants `Bash(agrefs:*)` to
  front/operator/argue and puts the venv's `agrefs` on PATH with
  `AGREFS_HOME`.
- **agobserver** — `agents.toml` argue role gains `Bash(agrefs:*)`; guides
  mention it.
- Guides already carrying the reference section (agfront ×3, agautolab,
  agforge, archsage) get the discovery sentence (`agrefs list` reads the
  shared catalog).
- **ansible_agdev** — the autolab node role writes `~/.config/agag/refs.toml`.

Done criterion check: ownership (Developer's Gitea account, relay acting on
the Developer's explicit action), discovery (host-level catalog location,
one catalog repository), publication (relay operations with a token the
human creates) and revision selection (full commit at selection time,
immutable afterwards) are decided, with the files above as entry points.
