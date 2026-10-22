# Challenge p1 — Step 2: WFE API and CLI

## Delivered (agdevworld `8d4cf58`)

Equivalent HTTP and CLI operations over the one store of step 1
(contract: `workflow_editor/docs/challenges.md`).

| Operation | HTTP | CLI |
| --- | --- | --- |
| list / search | `GET api/challenges?text=&project=&state=&limit=&offset=` | `wfe challenge list [--text] [--project] [--state]` |
| show (current) | `GET api/challenges/<id>` | `wfe challenge show <id>` |
| show (pinned) | `GET api/challenges/<id>?at=<commit>` | `wfe challenge show <id> --at <commit>` |
| create | `POST api/challenges` | `wfe challenge create --title … --body/--file … --by …` |
| edit | `POST api/challenges/<id>` with `base` | `wfe challenge edit <id> --base <commit> …` |
| resolve / withdraw / reopen | the same edit with `state` | `wfe challenge resolve|withdraw|reopen <id> --base … [--note]` |
| history | `GET api/challenges/<id>/history` | `wfe challenge history <id>` |
| evidence | `GET api/challenges/<id>/evidence/<name>[?at=]` | `wfe challenge evidence <id> [<name>] [--at] [--out]` |
| journal | `GET/POST api/challenges/operations/<n>[/retry|/discard]` | `wfe challenge ops [--all] | retry [<op>] | discard <op>` |
| status | (in the list answer) | `wfe challenge status`, `reindex` |

- **Reads** return the record with its revision (`commit`), creator, last
  change and evidence. A pinned read answers `pinned: true` and the
  `latest` revision beside it, so a client can show "attached version vs
  latest". The list carries counts per state, every project named, and the
  journal's unpublished operations.
- **Writes** answer `{operation, duplicate, record, publication}`; only
  `publication.state = published` (with its commit) means durable. A stale
  base answers 409 `conflict` with `current`; an open edit of the same record
  409 `pending-edit`.
- **Actors**: a page writes as the configured person
  (`conversations.json` `person`); an automated trial names `author`; the
  CLI requires `--by` (an agent writes as itself, `--for` names the person
  it writes for) and records the executing attempt (`WFE_ATTEMPT`) and
  `--run`. `--source-message <conv>:<id>` records where it was said. The
  CLI's request identity is derived from the command's content, writer and
  attempt, so repeating a command is one write; `--request` names one.
- **Other hosts**: on an execution node, `wfe` forwards every command to the
  coordinator (files named by `--file`/`--evidence` are carried), so remote
  runs use the same store and identities. `--service <url>` / `WFE_SERVICE`
  goes through a service's HTTP routes instead.
- **Retry**: the executor's reconcile (every minute) publishes unsynced
  operations — each first looked up in the remote history — and catches up
  mirrors.
- Status changes are ordinary versioned edits; `--note` becomes the commit
  message. No assignment, review step or automatic interpretation of
  conversation text exists; nothing here starts a run.

## Checks

- `test/challengeApi.test.ts` (2): the page writes as the person and a trial
  as its author; another origin cannot write (403); stale base → 409 with
  the current record; pinned vs current read; evidence by revision; history
  with actors and the source trailer; list counts and projects; the CLI
  writes as `--by` with on-behalf-of and attempt trailers, a repeated command
  is the same write, `--service` show and resolve, a stale CLI edit refused,
  missing `--by` is a usage error.
- Whole WFE suite: 527/527 passing; `tsc` clean.

## Known limits

- `wfe challenge evidence … --out <file>` on an execution node writes on the
  coordinator; on a node, read evidence through its HTTP route (the run
  brief gives it, step 3).

## Next

Step 3: challenge attachments on conversation and discussion messages,
carried into the receiving run's input.
