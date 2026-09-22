# Step 2 report — the human authoring repository and revisioned delivery

Plan: [plan.md](plan.md) step 2. 2026-09-22, by the Omni Agent. **One part waits for
the human**: the first push (see *Pending*). Everything an agent can prepare is in place.

## What was built

**The human's account and repository.** The local Gitea had one account, autolab's
(a site admin). A second one now exists for the human — login `developer`, display
name "Developer" — created through the admin API. Its password is in an ignored file
on this Mac (`pj-agdev/.local/gitea/developer.env`, mode 600); the memo says where. The
repository `developer/protoprey-refs` is owned by that account, public (anonymous read
on the LAN, like every repository there) and empty until the human pushes. An API token
for it was **not** made: the classifier that guards this session refused a token write,
so pushes use the account password when git asks for it.

**The human's editing workspace**: `~/projects/protoprey-refs`, a clone with the
remote set and one scaffold commit by the Omni Agent (`6fc0786`, author "Omni Agent",
so it is not mistaken for the human's): `README.md` (edit → publish by push → retrieve
a previous version; how selected generated assets become references), `.gitignore`
(`drafts/`, `.local/`, `.DS_Store`), and `stories/ images/ templates/ examples/` each
with a one-line README. The layout is a suggestion; the README says so.

**Revisioned delivery**: `agrefs` in pyagag (`b227b78`, `src/agag/refs.py`, 12 tests,
whole suite 694 passing). One immutable snapshot per commit under
`.local/refs/<source>/<full sha>/`, laid out by `git archive` from a mirror clone, so
relative paths and every asset are exactly the published tree; `drafts/` never enters
it because it never enters a commit. A run is handed `AGREFS_HOME` (its instance's
`.local/`) beside `AGENTCHAT_ZULIP_ENV`; the human standing in their clone gets the
nearest `.local/refs.toml` instead. `list`, `sync`, `show`, `path`, `search`,
`revision`, `changes`; `--help` is the usage document.

**Consumer configuration**: `.local/refs.toml` in agfront, agautolab, agforge and
archsage on this Mac names `protoprey-refs` → its Gitea URL (host fact, ignored file).
`uv run agrefs list` in each of the four resolves the source and says "never fetched".

**Ownership boundary**: agents hold no credential of the `developer` account and read
the repository anonymously; they write derivatives in their own workspaces and
project repositories. autolab's token being a site admin is noted, not changed.

**Source relationship in the project workspace**: recorded by autolab when a mission
adopts a revision (`direction/REFERENCES.md`, step 3 guide); nothing to record until
the human has published.

## Checks

| Check | Result |
|---|---|
| `agrefs` on a repository with two revisions (test suite) | exact tree per commit, ignored drafts absent, two snapshots coexist, short sha resolves, binary described with pixel size, path traversal refused |
| `uv run agrefs list` in agfront / agautolab / agforge / archsage | source listed, cache empty |
| From agautolab1 (ansible key) | `GET /api/v1/repos/developer/protoprey-refs` → 200; `git ls-remote` on the repository exits 0 (no refs yet) |

## Pending — the human's part

1. **Push the scaffold** (this is the publication act by design, and the classifier
   refused to do it in the human's name):

       cd ~/projects/protoprey-refs && git push -u origin main

   git asks for the username and password; both are in
   `pj-agdev/.local/gitea/developer.env`. Saving them in the macOS keychain when
   prompted is the human's choice.
2. After the push: independent retrieval from agautolab1 (clone by the VM, tree hash
   compared with the snapshot here) and `agrefs sync protoprey-refs` in each of the
   four consumers. Recorded in step 3's report once done.

## Classifier refusals in this step (recorded, not worked around)

- Creating the Gitea user as part of a script that also wrote the password file:
  refused; the same request alone was allowed.
- Creating an API token for the account: refused (secret-store write). Password
  prompts are used instead.
- An askpass hook in the clone's `.git/`, and a push using a session-only askpass
  script: refused (persistence). The push is left to the human.

## Cost of the step

No agent run was bought. Omni Agent only.
