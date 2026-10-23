# Challenge p3 — Step 2: publication, CLI and HTTP

Code: agdevworld `73eb720` (`server/challenges.ts`, `server/challengeApi.ts`,
`cli/challenge.ts`, `cli/remote.ts`, `nginx.conf`); contract
`workflow_editor/docs/challenges.md`; help: `wfe challenge help`.

## Commands and routes

| Need | CLI | HTTP |
| --- | --- | --- |
| list / search | `list [--text] [--project] [--state <kind\|state\|all>]` | `GET api/challenges?…` |
| a challenge with files and trials | `show <id> [--at]` | `GET api/challenges/<id>[?at=]` |
| file listing | `files <id> [--trial] [--at]` | `GET …/<id>/files[?at=]` |
| one file | `file <id> <name> [--trial] [--at] [--out]` | `GET …/<id>/files/<name>[?at=][&download=1]` |
| history | `history <id> [--trial]` | `GET …/<id>/history[?trial=]` |
| publish a trial (or challenge files) | `publish <id> [--trial <name>] <path>[=<name>]… [--base] [--drop]…` | `POST …/<id> {trial, files, drop, base?}` |
| create with source material | `create … --add <path>[=<name>]` | `POST api/challenges {files}` |
| edit summary fields | `edit <id> --base … [--title/--body/--state/--project]` | `POST …/<id>` |
| a draft's files | — | `GET …/operations/<n>/files/<name>` |

`evidence` (the command and the route) is gone. Its files are now named
`evidence/<f>` inside the challenge and read with `file`.

## Publication

- One write is one journal operation and one commit. It keeps the actor,
  `--for`, run, attempt, source, via and a new `Challenge-Trial` trailer.
  It keeps request deduplication, durable drafts (journal blobs until
  published), retry/reconcile and mirror pushes.
- Attribution over HTTP now carries `onBehalfOf`, `run` and `attempt`. In
  p1, `--service` dropped them.
- **Conflicts are per path.**
  - An edit names `base`. It is refused when a path it changes appears in
    `git log base..head`.
  - Writes that only add new files need no base. An existing file is then
    refused.
  - Publication re-checks against the clone's HEAD and keeps the draft as
    `conflict`.
  - `pending-edit` now applies only to overlapping paths.
  - Tested: two trials written on the same stale revision both publish. The
    same `summary.md` refuses the stale write. A conflict found only at
    publication keeps the draft, and its file can be read from the journal.
    A new trial can be added while another operation is in conflict.
- **Summary edits change only the named fields.** They use YAML Document
  editing. Comments, the order and other keys stay unchanged, and the body
  is kept byte for byte unless it is given. An unreadable frontmatter
  refuses field edits (`unreadable-summary`); replacing the file is the
  route for that case.
- **Path safeguards.**
  - Names are free relative paths of any depth. They must not contain
    empty, `.`, `..` or `.git` segments, control characters, quotes or
    backslashes.
  - The writer refuses to write through any symlink inside the clone (a
    committed `wedo/escape -> /tmp` was tested).
  - The CLI walks a directory and refuses a link that points outside it. It
    skips links to folders and `.git`.
- **Rendering safeguards.** Files are served with their type taken from the
  extension, and HTML and JS are served as text. Every file response has
  `nosniff` and `content-security-policy: sandbox`. `download=1` serves the
  file as an attachment, and a UTF-8 filename is encoded with `filename*`.

## Limits and body size

- A file can be up to 32 MiB (Gitea has no LFS). One write can add up to
  64 MiB in up to 500 files.
- `MAX_REQUEST` is about 86.3 MiB, which is 64 MiB as base64 plus 1 MiB.
  The service refuses a larger body from `content-length` before reading it
  (tested with a declared oversize).
- nginx `/wfe/` gets `client_max_body_size 100m`. p1's limit was nginx's
  default 1 MiB. That change is deployed in step 6.
- Help and the contract say that a linked external artifact is only as
  durable as its store, and that it is not part of the Git backup.

## Execution nodes

On a worker (`WFE_WORKER_AREA`), `wfe challenge …` used to be forwarded as a
whole command. Files named outside the run copy were then resolved on the
coordinator, and `--out` wrote into the coordinator's record copy. Now
`remoteMain` runs `challenge` locally with
`WFE_SERVICE=<coordinator>`:

- files are read on the node;
- `--out` writes on the node;
- writes go to the HTTP routes with the attempt (`WFE_ATTEMPT`) and the
  actor.

This needs the agpc worker re-pinned (step 6).

## Checks

- `test/challengeApi.test.ts` (3):
  - a trial published from a local directory with nested files over
    `--service`, with run and trial trailers;
  - a later trial that does not appear in the pinned read, while `latest`
    shows it;
  - audio type, download disposition, `file --at --out`, `files`, `show`
    and trial history;
  - the 413 for an oversize body.
- `test/challenges.test.ts`: interrupted publication (failed push, crash
  after push), duplicate requests, conflicts, symlinks and limits.
- `npm run check` passed: 538/538.
