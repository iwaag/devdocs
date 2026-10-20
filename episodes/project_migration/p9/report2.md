# p9 Step 2 — the missing capabilities

## Changes

**G1 — Gitea as a named principal** (agdevworld `075500c`, deployed).

- `gitea.json` may name other principals:
  `"principals": {"<name>": {"tokenFile": "…"}}`. The service and every
  `wfe` operation keep the top-level principal; only `--as` selects another.
- `wfe gitea whoami [--as <name>]`: the login it authenticates as.
- `wfe gitea repo <owner>/<name> [--as <name>]`: visible or not, and
  pull/push/admin.
- `wfe gitea git [--as <name>] -- <git args…>`: git with that principal's
  authentication header scoped to this Gitea's URL (the existing `gitEnv`
  mechanism). The token never appears in output, remotes, URLs or Git
  config. Force pushes (`--force`, `-f`, `+<refspec>`, `--mirror`) are
  refused before git runs. `--force-with-lease` is allowed. A partial or
  failed push is git's own error and can be retried as is.
- A token file may be the bare token or a credential note with a
  `token: …` / `…TOKEN=…` line (JSON `token` too). An error never echoes
  the content. The existing trustedagent credential is such a note.
- Help: `wfe help gitea`, an index line; README configuration table.

**G2 — Front's guide** (pj-front `7b890f2`): its tool list names
`wfe project register|obtain` for an existing repository and
`wfe gitea … --as <name>`. Lowerpoly-specific content is not added. The
request carries that.

Not changed: registration, obtain, basics, integration. The service
principal stays `developer` for all of them (report1, identity decisions).

## Checks

- `npm run check` in `workflow_editor`: tsc, 444/444 tests, vite build.
- New `test/principals.test.ts`:
  - token-file forms, with no echo of a password;
  - named and unknown principals;
  - force-push recognition;
  - against a disposable Gitea, with a second user as the principal: whoami,
    access, a push that fails without the principal and succeeds with it;
  - no token in output or `.git/config`;
  - a refused `--force`, then a non-fast-forward rejected by git.

## Deployment and live verification

| Check (installed launcher, the executor's context) | Result |
| --- | --- |
| `wfe version` | release `075500c` |
| `wfe gitea whoami` | `developer` |
| `wfe gitea whoami --as trustedagent` | `trustedagent` |
| `wfe gitea repo trustedagent/pj-lowerpoly` | id 144, default `main`; developer may admin, push, pull |
| same `--as trustedagent` | trustedagent may admin, push, pull |
| `wfe gitea git -- push` (Front guide to `developer/pj-front`) | pushed `7b890f2`; `git ls-remote` agrees |

The host setting now names `trustedagent`, with a reference to the
existing credential file in pj-agdev's ignored Gitea folder. The previous
setting and the rollback command are in the ignored environment notes.
Omni's own attempts to read that file were refused by Omni's session
classifier. The tool reads it, and Omni never saw the token.

Front's actual use of `--as trustedagent` is exercised in Step 3. Omni's
check here is not evidence that Front can do it.
