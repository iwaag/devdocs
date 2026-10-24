# fix5 step 2 — recoverable, consistent warning responses

agdevworld `caaca29`.

## One response, nothing recorded

- A reminder is one `RequestError(409, text, {code: 'reminder', boundaries, reminders})`.
  If one operation triggers several reminders, they come back together in
  that single response, merged by code. For example, a conversation post
  that is a `result` and also completes its node is evaluated as both
  boundaries and answered once.
- The check runs against the same record the write would replace, inside
  the existing run lock:
  - `runOpLocked` evaluates the reducer's result before `writeRecord`.
  - `adjustRun` evaluates the trial revision before it writes the
    revision file. A refused combined operation leaves no
    `definition/r<n>/`.
  - `registerPresentation` evaluates before any file is copied.
- A warned operation records no entry, revision or presentation, posts
  nothing and queues nothing.
- A retransmission with the same receipt is a duplicate before evaluation,
  so it records nothing new and is not warned again.
- A conversation post skips evaluation when its key was already posted.

## Paths

| Path | Continue | Refusal |
| --- | --- | --- |
| CLI | `--yes` | the text on stderr, exit 1; JSON detail under `--json` |
| HTTP `POST …/ops` | `"yes": true` | 409 `{error, detail: {code: 'reminder', reminders}}` |
| HTTP `POST …/adjust` | `"yes": true`; `completing: {node, outcome, artifacts?}` is now accepted, so the combined operation is the CLI's | same |
| Remote worker | `--yes` travels with the forwarded argv | the coordinator's own CLI output and exit code |
| Run view (browser) | the person confirms explicitly after reading the whole reminder | otherwise shown as a warning beside the form; nothing is set silently |

The browser itself only sends answers, decisions, stops, resumes,
cancellations and instructions. None of these is a boundary, and a person's
answer is never evaluated. The confirmation path exists so that any later
form that does trigger a reminder behaves correctly.

A related fix: on a remote worker, `--source out` (a directory name without
a slash) was not uploaded. That made `wfe present register` and
`wfe run publish` from a PC fail with "not a directory". `source` and
`review` are now path options.

## Repetition

- Explicit operations are evaluated every time. `--yes` is per command;
  nothing persists from it.
- The advisory text that goes with an answer about a presentation appears
  only while that answer is pending: in the brief, the continuation and
  `wfe run state`. Take-up shows it once. It does not repeat after that.

## Checks

Added to `test/reminders.test.ts`:

- the HTTP ops route returns 409 with the reminder codes and records
  nothing, and `yes` records the operation;
- the HTTP combined route refuses without writing a revision;
- an outdated `expectSeq` is refused even with `--yes`;
- of two concurrent completions with `--yes`, one records and the other
  gets `state`.

Added to `test/workers.test.ts`: a forwarded `wfe present register` gets
`cleanup-unplanned`, registers nothing on the coordinator, and `--yes`
then registers it.

Results: reminders 10/10, workers 3/3, tsc clean.
