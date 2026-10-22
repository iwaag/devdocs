# workflow_editor fix2 — Step 1: unify parsing and serialization

agdevworld `4be9732` (not yet released; see Step 4).

## What was inconsistent

- `emptyWorkflow` (empty template, freework template, basic workflows,
  project creation) set `intake: {zulip: requests}`.
- `workflowFromData` read an absent `intake` or `zulip` as `''` (off).
- The writer skipped empty values for keys the file did not have, so an
  opt-out made in the model of a file without `intake` was never written.
  Reload turned it back into the loader's reading.
- The canonical definition projected `intake` only when it was non-empty.
  A file without intake and a file with `zulip: ""` had the same digest.

## What it is now

| File says | Model | Canonical `intake` | Written back |
| --- | --- | --- | --- |
| nothing | `requests` | `{zulip: requests}` | unchanged (no key added) |
| `intake: {}` / `intake:` / `{zulip: null}` | `requests` | `{zulip: requests}` | unchanged |
| `{zulip: requests}` | `requests` | `{zulip: requests}` | unchanged |
| `{zulip: ""}` | `''` | `{zulip: ""}` | unchanged |
| `zulip: 1`, `true`, a list, a mapping; `intake: requests`, a list, `""` | — | — | shape problem: the file is unreadable, as for any wrong type |
| `{zulip: everything}` | `everything` | — | validation error `intake-mode` |

- **Loader** (`shared/model.ts`): only an absent or null `zulip` takes
  the default. Otherwise the value is read as written.
- **Writer** (`server/yamlDoc.ts`): a scalar shape may say what its
  absent key reads as (`absent: 'requests'`). It is empty only at that
  value, so `''` is written even where the file had no `intake`. A new
  file (`newWorkflowText`: templates, basics, project creation) declares
  intake either way. Byte-for-byte round trip of unchanged documents is
  kept.
- **Canonical definition** (`shared/canonical.ts`): intake is always
  projected. A file without intake has the same digest as one with
  `{zulip: requests}`, and the opt-out has its own.
- **Editor API** (`server/api.ts`): a client model must carry
  `intake: {zulip: text}`. A model without intake would have deleted the
  file's key and turned an opt-out into the default.
- No legacy mode. The editor UI has no intake control. Inspector and
  validation messages now name the opt-out (`intake: {zulip: ""}`) as the
  reason a target refuses delegation.

## Tests

`test/intakedefault.test.ts` (new) covers:
- omitted intake, `{}`, an empty value, null `zulip` and explicit
  `requests`;
- the explicit opt-out;
- seven invalid shapes, and an unsupported mode;
- edit/save/reload: unrelated edits, off where no key existed, back on, an
  empty mapping kept, new files for both values;
- canonical equality and difference.

`settings.test.ts` changed: its "legacy definitions keep `''`" assertion
now checks that the explicit opt-out round-trips, and the projection
always has intake. Runtime tests (Step 2) are in `intakestore.test.ts` and
`delegateflow.test.ts`.

`npm run check`: 515/515, tsc and both builds pass.
