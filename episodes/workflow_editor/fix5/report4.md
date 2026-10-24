# fix5 step 4 — guidance and documentation

agdevworld `c00d64e`, agautolab `8f341ed`.

## WFE documentation (agdevworld `workflow_editor/docs/`)

- **runs.md**, new section *Reminders at operation boundaries (workflow_editor
  fix5)*. It covers:
  - the boundaries and codes;
  - role identification from structured records;
  - what "planned" and "completes the run" mean;
  - the three responses and how `--yes` behaves;
  - answers about presentations;
  - the CLI, HTTP, remote and browser paths;
  - the evidence the change rests on: run-010 and run-011, and run-001…008
    via fix4's report.

  The *Operations* table has a row for `record-done`.
- **adjustments.md**, new section *Recording work already done*. It covers:
  - the one-entry revision;
  - what is and is not recorded (no start or end; reported times only as
    reported);
  - evidence checks;
  - presentation links;
  - honest views;
  - when to use `adjust` instead;
  - that a missing presentation cannot be declared done.
- **presentations.md**: the reminders around presentations. They are
  reminders, not gates, and nothing stops a page automatically.
- **conversations.md**: fix3's "completed elsewhere" warnings are now
  reminders of the shared evaluator, and `reply --kind result|synthesis`
  and `--completing` are evaluated before the post.

## Command help

- `wfe run help complete` and `wfe run help ask` state once that WFE may
  answer with reminders, that they record nothing, and that the refusal
  explains how to continue. The synopsis no longer shows `[--yes]`, so
  ordinary use is taught without it, as the plan asks.
- `wfe run help take-up` says that an answer about a presentation comes
  with its reminder and that nothing is stopped by it.
- `wfe run help adjust` says that `--completing` is checked on the revised
  graph and that work already done uses `record-done`.
- `wfe run help record-done` owns the full usage (step 3).
- `wfe present help` (register) and `wfe conversation help` (reply) name
  their reminders.

## wfrun guide (agautolab `agent/guides/wfrun/guide.md`)

Two facts were added, in the guide's existing style:

- **Reminders**, under *The steps the person expects to see*: what is
  checked, that a reminder is not proof, the three responses (with tool
  usage left to `wfe run help record-done`), and that work not yet done
  goes through `adjust` and that a presentation cannot be declared done.
  It cites gimmickimg/freework/run-010 and run-011 and fix4's report.
- Under *Presentations*: an answer may ask for changes or for the page to
  stay up. Whether more work or cleanup follows is read from the answer.

No project's guide and no frozen catalog snapshot was changed. The plan
asked that no blanket instruction be added to every project.

## Code change in this step

The serving-page reminder was renamed to `presentation-serving`.
presentations.md records that p11 retired a gate called
`presentation-open`, and reusing that name would have suggested that the
gate was back.

agautolab tests: 47 passed, 1 skipped.
