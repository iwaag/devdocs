# give_context_easier p1 — pre-research

Facts gathered before planning, plus notes that should save the implementer
time. Everything under "Facts" was read from code, the running Gitea or the
local notes on 2026-09-26; "Advice" is opinion.

The braindump asks for two things:

1. A context repository the Developer wants every agent to share should be
   easy to create and manage from agdevworld.
2. The Front Room shows a list of those repositories that can be shown and
   hidden, and clicking an entry adds a reference to the chat input, as agent
   IDEs do.

## Facts

### What the Front Room can take today

- The composer is text only. `POST /frontdesk/<id>/post` reads a JSON body
  (`text`, `answers`, `not_answer`, a submit `token`) and posts it into
  `#front › front-desk-<id>` as the Developer
  (`agentroom/src/agentroom/server.py` `_body` / `_desk_post`,
  `agentroom/src/agentroom/frontdesk.py` `FrontDesk.post`). No route accepts
  a file, and no multipart parsing exists.
- The frontend composer (`src/frontDeskInput.ts`) is a DOM textarea over the
  Phaser bar, because a canvas cannot host an IME. Its handle exposes
  `value()`, `set(text)`, `focus()`, but nothing that inserts at the caret.
  It handles no drop and no paste of files.
- A file attached in Zulip directly (`/user_uploads/…`) is never fetched by
  any agent: nothing in pyagag, agfront, agautolab, agforge or agobserver
  reads uploads. An agent sees only the link text.
- The Front Desk and the Arguing Room are one scene
  (`src/scenes/FrontDeskScene.ts`, 1.5k lines) with two adapters
  (`src/roomState.ts`). Anything added to the scene appears in both rooms
  unless it is gated on the adapter.
- The scene already has a panel that can be shown and hidden, the history
  (`historyOpen`, `toggleHistory`, `historyPanel` container), and a row of
  Phaser text buttons (`history`, `settings ⟳`, `finish ✔`, …). Phaser buttons
  have no DOM.

### agrefs — the only way a file reaches an agent today

- `agrefs` is `agag.refs` in pyagag (`src/agag/refs.py`, since b227b78,
  `adventure_game` p2). A reference is
  `<source>@<revision>[:<path>]`, e.g.
  `protoprey-refs@3f2a1b0:scenes/meadow/composition.png`.
- Verbs: `list`, `sync <source>[@<rev>]`, `show`, `path`, `search`,
  `revision`, `changes`. `show` prints text, lists a directory, or describes a
  binary (kind, pixel size, bytes, path). The agent then opens an image with
  its own harness's image reader via `agrefs path`.
- A revision must be named. `latest` is allowed but every answer prints the
  sha it resolved to. Snapshots are `git archive` of one commit under
  `<home>/refs/<source>/<sha>/`, beside a `mirror.git`. Nothing is rewritten
  in place.
- Sources come from **one `refs.toml` per agent instance**
  (`load_sources`): `[[source]] name, url, about`. The name must match
  `^[a-z0-9][a-z0-9._-]{0,63}$` (no capitals). An unknown name is an error
  listing the known ones.
- Where that file is: `AGREFS_HOME`, which the listener sets to the instance's
  `.local/` for every run (`agag/agent.py` `chat_environment`); otherwise the
  nearest `.local/refs.toml` above the working directory. An autolab mission
  running in a `.local/missions/<slug>/m<id>/` worktree still gets its
  instance's `.local`.
- `refs.toml` exists today in four places, all naming only `protoprey-refs`:
  agfront, agautolab, agforge, archsage. The agautolab1 VM has none (it can
  reach Gitea, but was not redeployed after `adventure_game` p2).
- Tool grants: agfront's desk, front and argue roles all have
  `Bash(agrefs:*)`. agautolab and agforge also grant it and their guides
  (workplan/workrun, assetplan/assetrun, argue) describe it.
- Front's guide (`agfront/agent/guides/front/guide.md`, "Human-authored
  references"; the desk and argue guides carry the same section) tells Front
  to resolve a named reference with `agrefs`, and to carry the **resolved sha**,
  never `latest`, into whatever it writes for other agents.
- Every consumer takes pyagag as a git dependency on `main`, pinned by
  `uv.lock` (agfront currently at `7f6095b`). A pyagag change reaches an agent
  only after that agent's lock is bumped and its listener restarted.

### Gitea

- `http://agstudio.local:3000`, Gitea 1.27.1, local docker compose on this
  Mac (agstudio).
- Anonymous API reads work: `GET /api/v1/repos/search`,
  `/api/v1/users/<user>/repos`, `/api/v1/repos/<owner>/<repo>/topics`.
- About 100 public repositories. Every one except `developer/protoprey-refs`
  lives in the `autodev` org and is agent output: `<project>`,
  `<project>-direction`, `<project>-devlog`, smoke and e2e leftovers. So
  "every repository on Gitea" is almost entirely **not** human-authored.
- `developer` (id 3) is the human's account. `developer/protoprey-refs`
  already carries a description written for agents ("Human-authored
  references for ProtoPrey: … read by the agents at a pinned revision.").
  Its topics are empty.
- Credentials: the developer account has a password only
  (`pj-agdev/.local/gitea/developer.env`, 0600). **No API token exists**: the
  Omni Agent's auto-mode classifier refused to create one and refused to push
  under the human's credential. Publication (`git push`) has so far been the
  human's own act. autolab has its own token
  (`agautolab/.local/gitea/autolab-agent.token`) scoped to the `autodev` org
  and uses it in `agautolab/src/agautolab/project_init.py`
  (`ensure_gitea_repo`: `GET` then `POST /api/v1/orgs/<org>/repos`). That
  function is a working example of repository creation.

### The relay (agentroom)

- A stdlib `ThreadingHTTPServer`, unauthenticated, loopback, on :8094,
  supervised by launchd. `ROUTES` and `WRITE_ROUTES` at the top of
  `server.py` list every door, and `/healthz` returns both lists.
- Its module docstring states the stance: routes that change the realm are
  deliberate and few. The existing write doors post to Zulip with the
  Developer's credential. None writes to Gitea.
- **Precedent for a git repository served through the relay:**
  `agentroom/src/agentroom/settings.py` (the settings repository). It has an
  ignored config with a tracked example, an explicit `sync` that snapshots a
  commit and switches the active revision only when valid, retained
  `revisions/<sha>/`, and `GET /settings[/<sha>[/<path>]]` that serves only
  files the manifest names ("which is what keeps this from being a file
  server"). A context-repository list can follow the same shape.

## Advice

### Discovery instead of per-agent registration

The Developer's objection in the planning chat: with 20 agents, adding one
source means editing 20 `refs.toml` files. Suggested direction:

- Make agrefs discover sources from Gitea. Candidate rule: **every repository
  owned by `developer`** is a source, its Gitea name is the source name, and
  its Gitea description is the `about`. This keeps agents' `autodev` output
  out of the set of human originals, which the "all of Gitea" version would
  lose.
- Alternatives worth weighing in the plan: a dedicated Gitea org (e.g.
  `refs`) if `developer` will also own non-context repositories, or a Gitea
  topic (e.g. `agrefs`) as the marker. A topic is invisible in the repository
  name and easy to forget. An owner or org is visible in every URL.
- Keep `[[source]]` entries as explicit overrides for a repository outside
  Gitea. Put the discovery base (Gitea URL plus owner) in `refs.toml`, or give
  it a pyagag default so no per-agent file is needed at all. Remember that
  agautolab1 is a different host that currently has no file.
- Cache the discovered list with the snapshots. An agent should still be
  able to resolve an already-synced `name@sha` when Gitea is down, because
  pinned revisions are the whole point.
- Gitea names can contain capitals; `NAME_RE` does not allow them. Decide
  whether to lowercase, reject, or relax the pattern. Lowercasing makes two
  repositories collide.

### Creating repositories from agdevworld

- This is the first relay route that would write to Gitea, and it needs a
  credential the relay does not have. Options: a developer API token the
  **human** creates and stores in an ignored 0600 file (the Omni Agent cannot
  create or write it under auto mode), or the Developer keeps creating
  repositories in the Gitea UI and agdevworld only lists them. The second
  option satisfies "easy to manage" with no new write door and may be
  enough for p1.
- If creation is in scope, copy `ensure_gitea_repo`'s idempotent
  GET-then-POST, and use `POST /api/v1/user/repos` (the token owner's
  account) rather than an org route. Set the description at creation, since
  it becomes the `about`.
- An empty repository has no commit, so `agrefs sync` fails. Either create
  with `auto_init` (a README commit) or make agrefs say "empty" clearly.
- Uploading files through the browser into the repository is a larger step
  (commits made by the relay). It is not in the braindump. Keep it out
  unless the Developer asks.

### The list and the click-to-insert composer

- Serve the list from the relay (e.g. `GET /refs`) rather than from the
  browser calling Gitea directly. The relay already owns the loopback
  CORS stance, and one place decides what counts as a context repository.
  If agrefs gains discovery, the relay can call the same pyagag function, so
  the browser and the agents cannot disagree about the set.
- What a click inserts is a product decision for the plan:
  - `name` alone lets Front resolve it and pin the sha, as its guide already
    says.
  - `name@<sha>` pins at click time, which is precise but less readable.
  - A path inside the repository means the list needs a file tree
    (`GET /api/v1/repos/<o>/<r>/contents/<path>` is anonymous-readable), or
    a later step.
- The composer needs an insert-at-caret method in `frontDeskInput.ts`
  (`selectionStart`/`selectionEnd`, then fire the existing `onChange` so the
  counter and the Send button update). `set()` replaces the whole draft.
- Decide whether the panel appears in the Arguing Room too. It is the same
  scene, and archsage and the argue roles also have agrefs, so appearing
  there is probably right, but say so in the plan.
- Front already knows the reference syntax. A plain-text mention such as
  `agrefb:myfolder/request.md` needs no new relay post field or `ag-post`
  marker. Prefer that to a structured attachment field, which would need
  every consumer to learn a new shape.

### Verification and deployment notes

- UI checks: `&demo=1` exposes `window.__room`, and the ignored
  `.local/roomshot.mjs` drives Phaser buttons (`btn:<sceneField>`) and
  panels. Kill a lingering Chrome on its port first. Live pages have no hook.
  Keep `docker compose up --build -d web` (:8090) current after a UI change.
- Relay: `cd agentroom && uv run pytest -q`; it is restarted through launchd,
  as described in `pj-agdev/.local/devenv.md`.
- pyagag: `uv run pytest` in pyagag (`tests/test_refs.py` exists). Read the
  summary line; a `| tail` pipe hides a red run. Then bump `uv.lock` in each
  consumer (agfront, agautolab, agforge, archsage, agentroom) and restart
  their listeners. A consumer left on the old lock silently keeps
  per-file-only sources.
- End-to-end proof worth planning for: create a new context repository,
  push one text file and one image, click it into the Front Desk, and see Front
  read both and hand a pinned `name@sha:path` to another agent that also reads
  it. Test it on an agent whose `refs.toml` was never edited.
- The Developer's Gitea password is theirs. Pushing to `developer/*` stays
  the human's act unless a token they created says otherwise.
