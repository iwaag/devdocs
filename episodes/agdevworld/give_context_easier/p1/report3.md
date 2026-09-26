# give_context_easier p1 — step 3 report: every agent can discover and read

## Setup performed

- **Gitea token.** At the Developer's suggestion (2026-09-26) the Omni Agent
  created an access token for `developer` through the Gitea API with the
  account password in `pj-agdev/.local/gitea/developer.env`. It is named
  `agdevworld-contexts`, has scopes `write:repository` and `write:user`, and
  is stored in `pj-agdev/.local/gitea/developer.token` (0600). Neither value
  was printed. The relay's ignored `agdevworld/.local/contexts.toml` names it.
- **Catalog.** `agentroom-contexts init --import agfront/.local/refs.toml`
  created `developer/context-catalog` and its first `catalog.toml`
  (`1b9be53`), with the `protoprey-refs` registration imported.
- **Host setting.** `~/.config/agag/refs.toml` on agstudio names the catalog
  (`[catalog] url`), once for every consumer on the host.
- **Per-agent registrations retired.** The four `refs.toml` files (agfront,
  agautolab, agforge, archsage) were renamed to `refs.toml.pre-catalog`. No
  consumer on the host now has a per-source entry.

## Code (already committed in step 2/3 commits, plus one grant)

- pyagag `c462f73`: `agag init` grants `Bash(agrefs:*)` to every generated
  role, and the generated guide points at `agrefs list`.
- Every consumer is locked to pyagag `c462f73`. agfront `70b7f3f` → `8c4ddc4`
  (the `routine_run` grant added today), agautolab `ae0ee6b`, agforge
  `978d859`, archsage `817ec07`, agobserver (pj-agdev), cagent
  (pj-clusterintent `47e0273`), relay (agdevworld `50e3428`, then `93c1a50`).
- Guides: the existing "Human-authored references" sections (Front ×3,
  autolab ×3, forge ×4, archsage) now say `agrefs list` shows every source the
  developer published, with what each is for. Front's three also say a
  reference inserted from the context panel already carries the full commit
  and is passed on as is. cagent's front/operator/argue guides, its window and
  authenticated AGENTS.md, and Observer's argue and intake guides gained a
  short paragraph. Nothing injects repository contents into prompts; names
  and descriptions are one `agrefs list` away.
- cagent: `contexts` tool on the in-process window and authenticated doors
  (the window has no shell); `agrefs` on PATH with `AGREFS_HOME=cagent/.local`
  for the topic roles, granted to front/operator/argue.
- Observer: argue role granted `Bash(agrefs:*)`; its agcode roles already had
  a shell and `agrefs` in their venv.
- ansible `366d7ed`: `agag_agent` writes `~/.config/agag/refs.toml` from
  `AGREFS_CATALOG_URL` on the controller, for remote nodes.

## Deployment

All six listeners, cagent-api and the relay were kickstarted at
2026-09-26T08:34:49Z (checked first: no harness grandchild under any
listener). Front was restarted again at 08:37:58Z for the `routine_run` grant.
Every listener's startup recovery queued nothing. The web image on :8090 was
rebuilt (HTTP 200). `/healthz` answers ok, mirror live. `nctl drift` earlier
today: converged=46 (observation dumps 22 h old; launchd shows every job
running).

## Verification per instance and role

Each role was resolved the way its listener resolves it (`resolve_spec_role`
for agag agents, `resolve_cagent_role` for cagent). `agrefs show
protoprey-refs@latest` then ran in that exact run environment (PATH,
`AGREFS_HOME`); all of these instances have no per-source config. Every row
printed `protoprey-refs@34ae3f9/ (revision 34ae3f9e98…)`.

| instance | roles with a working read and a grant | roles left without a grant (by design) |
|---|---|---|
| front-agstudio1 | front, desk, argue, routine_run | present (no shell: renders recorded speech only) |
| autolab-agstudio1 | front, director, superdirector, coding, supercoder, mediator, argue | summarizer |
| agforge-agstudio1 | front, generator, argue | — |
| archsage-agstudio1 | archsage, front | sage (bounded `sagetree` reader) |
| agobserver-agstudio1 | front, observe, triage (agcode, shell), argue | — |
| cagent-agstudio1 | front (agcode), operator, argue; in-process `contexts` tool on window/authenticated doors | — |

**New instance.** `agag init ctxprobe` into a scratch directory produced a
project with the grant on every role and nothing in `.local/` but
`instance.toml`. `agrefs list` there reads the catalog (`current at 1b9be53`,
`protoprey-refs`). It was generated only, not provisioned in Zulip.

This is environment-level evidence: the tool is reachable, granted and
answering in each role's environment. A model actually reading through the
tool (Front and a delegate) is step 6's live trial.

## Not done

- **agautolab1 (remote).** It runs only the autolab gateway, on agautolab
  `b38a6af` from 2026-08-16. It has no listener and is not on the `#agents`
  board. Redeploying it is ansible against a real cluster node, which the
  local notes put behind explicit approval. The role change is ready
  (`AGREFS_CATALOG_URL=<catalog URL reachable from the VM> ansible-playbook …
  setup_autolab_node.yml --limit agautolab1`). The path is unverified until
  the Developer approves.
