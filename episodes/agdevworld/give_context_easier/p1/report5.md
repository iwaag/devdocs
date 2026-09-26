# give_context_easier p1 — step 5 report: creation and basic management

## What exists

**Relay operations** (agdevworld `50e3428`, `agentroom/src/agentroom/contexts.py`),
all on the Developer's Gitea token and with no model run:

| route | what it does |
|---|---|
| `POST /contexts` `{id, name, description, readme}` | creates `developer/<id>` if missing (`POST /user/repos`, no auto-init), pushes a first README commit if the repository has none, then adds the catalog entry. Each step is checked before it is taken, so a repeat finishes a cut-short creation, and the repository name is the id, so no duplicate is possible. |
| `POST /contexts/register` `{id, repository \| url, name, description, branch}` | an existing repository with at least one commit into the catalog; a second registration of the same repository is a no-op; the same id for another repository is refused (409) |
| `POST /contexts/<id>` `{name?, description?, status?}` | display name and description (mirrored to Gitea's description for the Developer's own repositories), archive / reactivate. The id never changes. |
| `POST /contexts/<id>/publish` `{base, message, files[]}` | files as text or base64 (20 MB each, 40 MB per publication; deletions allowed) committed on `base` in a bare work clone with a scratch index, pushed **without force**. A moved branch is a 409 with the new head, the files changed since `base` and which of them the request touches. Nothing is written. |

Catalog edits are structured (read the newest `catalog.toml`, apply one change,
check that the result parses, push without force) and re-applied up to three
times on a race. Every write appends a line to
`agentroom/.local/contexts/publications.jsonl`, and every commit carries
`Published-from: agdevworld context panel`. Commits are authored by the token
owner's Gitea identity (`Developer`). Ordinary git authoring keeps working:
the relay fetches before each publication and treats a push made elsewhere as
a newer revision.

**Panel views** (agdevworld `93c1a50`, `a790fff`): "+ new" (name → suggested
id, description, README), "register existing…", "✎ name & description",
"+ add / upload files" (a Markdown file plus any uploaded files under a
chosen folder, one publication), "edit" on a Markdown file, "archive" /
"reactivate", and "open in Gitea ↗". A conflict shows what moved and offers
"publish mine on top of <new>" or "load <new>'s version (keeps mine below)".
The edit is kept in both cases. When writes are unavailable the create view
says why (the relay's reason).

## Live proof (deployed relay, web image on :8090, 2026-09-26 08:39–08:44Z)

Driven through the panel over CDP (`agdevworld/.local/ctx/s5-live.mjs`,
`s5b-reactivate.mjs`; screenshots `agdevworld/.local/shots/ctx/L01`–`L06`):

1. **Create.** "Context trial", id `context-trial` → `repository created,
   README published, registered in the catalog at 479c23a`.
2. **Text and image in one publication.** `brief.md` + uploaded
   `images/pond-dusk.png` (480×270 PNG) → `context-trial@cceca3b (2 files)`.
   The preview loaded the image back through the relay at that commit
   (`naturalWidth` 480).
3. **Edit later.** `brief.md` opened in the editor, one line added →
   `context-trial@3a72710 (1 file)`.
4. **Rename.** "Context trial — pond at dusk"; the id and every reference are
   unchanged.
5. **Archive** → gone from the default picker, listed under "show archived".
   **Reactivate** → back in the picker.
6. **Conflict**, with a stale base (`479c23a`) against the live relay:
   409 `context-trial has a newer publication (3a72710) than the one you edited
   (479c23a)`, `changed: [brief.md, images/pond-dusk.png]`, `touched:
   [brief.md]`. `git ls-remote` still shows `3a72710`, so nothing was written.
7. The Front Desk draft was never touched, and nothing was posted.

Records: Gitea `developer/context-trial` main = `3a72710` ← `cceca3b` ←
`479c23a`, all `Developer <developer@agstudio.local>`. The catalog entry
carries the new name, `status = "active"`, `registered_at 2026-09-26T08:39:23Z`.
`publications.jsonl`: create, register, publish ×2, update ×3 (name,
archived, active).

Found and fixed during the live run: long file paths in
`protoprey-refs` pushed the size and buttons out of the panel (horizontal
scroll). They now truncate with an ellipsis, with the full path in the
tooltip (`a790fff`); body overflow measured 0 px afterwards.

Automated coverage of the same operations (step 2's suite, real git):
creation, retry after an interrupted registration with no second repository
or entry, registration rules, rename/archive/reactivate with old references
still resolving, publication and both conflict kinds (the panel's own
publication, and an ordinary `git push` in between), unchanged content
refused, path and id validation, the HTTP routes.

Out of scope, as planned: deleting a repository, a general browser IDE.
