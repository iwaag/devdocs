# give_context_easier p1 — report

## Outcome

The Developer can create and manage shared context repositories from
agdevworld, find them in a collapsible panel in the Front Room, and add an
exact, pinned reference to a draft with one click. Every agent discovers and
reads the same published material with no per-repository configuration: the
set of sources is one catalog repository, configured once per host.

Proven together on 2026-09-26:

1. A repository was created through the UI, and text and an image were
   published to it.
2. Its pinned reference was inserted into a Front Desk draft.
3. Front read the text and looked at the image.
4. Front delegated the same reference to forge, which read the same originals
   from its own cache without a new configuration entry. An independently
   configured consumer read identical bytes.
5. A second revision was published: a new selection gets it, and the earlier
   reference still reads the earlier bytes.

Step reports: [report1](report1.md) (integration points, contract) ·
[report2](report2.md) (catalog and resolution) · [report3](report3.md)
(every agent) · [report4](report4.md) (picker) · [report5](report5.md)
(creation and management) · [report6](report6.md) (verification, deployment,
live workflow).

## The contract

- **Reference**: `<source>@<commit>[:<path>]`. The panel inserts the full
  commit resolved at the moment of selection and shows the short one.
  Publication changes future selections only.
- **Catalog**: `developer/context-catalog` › `catalog.toml`
  (`agag.refs-catalog.v1`). Per entry: `id` (stable, the `<source>`), `name`,
  `description`, `repository` (host-relative) or `url`, `branch`, `status`
  (`active` | `archived`). Archived entries resolve but are not listed.
- **Location**: `~/.config/agag/refs.toml` `[catalog] url` per host. An
  instance `refs.toml` may override or add explicit sources.
  `AGREFS_CATALOG` overrides for one process.
- **Freshness**: re-read after 60 s and at once for an unknown name. An outage
  keeps the last good catalog (`last-known`) and every pinned reference
  already fetched.
- **Writes**: relay operations on the Developer's Gitea token, no model run.
  A publication is a commit on the editor's base revision, pushed without
  force; a moved branch is a recoverable 409. Creation is idempotent
  (repository, then first commit, then catalog entry).

## Deployed revisions

| repository | revision |
|---|---|
| pyagag | `c462f73` (catalog `06ba708`) |
| agdevworld (relay + frontend) | `5eba6f4` |
| agfront | `8c4ddc4` |
| agautolab | `ae0ee6b` |
| agforge | `978d859` |
| archsage | `817ec07` |
| pj-agdev (agobserver, submodule pointers) | `77bcce6` |
| pj-clusterintent (cagent) | `47e0273`, ansible_agdev `366d7ed` |
| Gitea | `developer/context-catalog`, `developer/context-trial` (trial) |

Every listener, cagent-api and the relay run these revisions (kickstarted
08:34–08:38Z). The web image on :8090 was rebuilt. `nctl drift`:
converged=46.

## Omni Agent work performed for the system

- Created the Developer's Gitea access token `agdevworld-contexts` with the
  account password, at the Developer's suggestion. This was human-side setup,
  not an agent's job.
- Created the catalog and imported the `protoprey-refs` registration
  (`agentroom-contexts init`). Renamed the four per-agent `refs.toml` files
  aside.
- Created the trial context and drove the Front Desk as the Developer (through
  the UI).
- Nothing was done in place of an in-system agent: Front and forge read,
  delegated and reported on their own. (No "did X for agent Y" note is
  owed.)

## Remaining gaps

- **agautolab1** (remote autolab gateway, `b38a6af`, no listener) was not
  redeployed. That is ansible against a cluster node and needs approval. The
  role now writes the host catalog file when `AGREFS_CATALOG_URL` is set.
- **Trial leftovers**, left for the Developer: `context-trial` (active; archive
  it from the panel when no longer wanted), the Front Desk conversation
  `front-desk-20260926-ctx-trial` (open, with Front's relayed question), and
  forge's plan a11459 (never started). Front is waiting on the Developer's
  answer in that conversation; ignoring it costs nothing.
- The panel is an overlay on narrow screens, as the history panel is. On wide
  screens the dialogue moves aside.
- No automatic rewrap of old references and no repository deletion, by
  design (p1 scope).
- Token scopes: `write:repository`, `write:user`. The relay itself is
  unauthenticated on loopback, as before, so anything on this Mac that can
  reach :8094 can publish as the Developer. That was accepted by the plan
  (no broad hardening).
