# fix5 step 3 — atomic retrospective recording

agdevworld `d06e3ac`.

## `wfe run record-done`

```
wfe run record-done <run> (--template <t> | --type do|study|talk --name <text> --id <node>)
    [--id <node>] [--after <node>]... [--before <node>]...
    --outcome <text> --reason <text> [--artifact <path>]... [--commit-id <sha>[@<repository>]]...
    [--entry <seq>]... [--question <q>]... [--output <o>]... [--presentation <p>]
    [--started-at <time>] [--ended-at <time>] [--expect-revision r<n>]
```

`wfe run help record-done` owns the full usage. HTTP:
`POST …/runs/<wf>/<run>/record-done` with the same fields.

## What one entry does

`recordDone` (server/runs.ts) works inside the run lock. It builds the
revision from the active definition with the editor's YAML round trip, as
`instantiate` does, so a node made from a template keeps its provenance.
It then validates and compiles the revision and checks the evidence on
disk:

- each file must exist, and is recorded with its hash;
- each commit must resolve to a full id in the run's copy;
- each entry must be an earlier seq of this run.

The reducer checks questions, outputs and presentations against the record.

The history entry `run.record-done` carries the revision fields and the
node's outcome. The reducer then applies, in one step:

- the revision rules shared with `run.adjust`: parent, revision id and
  file, graph validity, attempt fencing (another attempt's run is refused);
- only the one node and its generated steps are new. Nothing is removed or
  changed, and the node may follow completed nodes only. During deployment
  this became: it may follow nodes that are completed or still under way,
  and edges of generated steps may move (see [report6.md](report6.md));
- the node goes straight to `completed`. Its `retrospective` records
  `{seq, at, by, reason, reported: {started, ended}, evidence}`, and
  `started` and `ended` stay `null`. The entry's change is
  `{node: [null, 'completed']}`, read as "added as completed (recorded
  afterwards)";
- `--presentation p` adds `links: [{node, seq, at, by}]` to that
  presentation. Its registration entry and `node` stay as they were.

The revision file is written before run.json. A refusal removes it, so no
partial pending node can be scheduled. A retransmission with the same
receipt returns the existing result without writing.

## Honesty in the views

| Surface | Shown |
| --- | --- |
| Run view graph | dashed outline; foot "Done (recorded afterwards)" |
| Node details | a "Recorded afterwards" box: who, when, entry, "Nothing ran this node", the reason, reported work time (or "unknown"), the evidence |
| History (view, `wfe run show`, continuation) | `page added as completed (recorded afterwards)` |
| `wfe run show` node line | `RECORDED AFTERWARDS by … at … (nothing ran it then: …)` |
| Revisions | source "work recorded afterwards (page)"; the node exists from that revision on and not in r1 |
| Notifications | one `adjusted` event, "recorded afterwards as done: page — reason". The run's `completed` event is not repeated |

## Interaction with the reminders

- A recorded `work` node is visible work, so `unrecorded-activity` no
  longer applies.
- `presentation-missing` still applies to that work unless a presentation
  covers it: one registered later, or one linked with `--presentation`. A
  missing presentation is not repaired by declaring it done.
- A linked presentation does not make the work node a presentation node.
  An early version treated it that way and a test caught it.

## Checks

Added to `test/reminders.test.ts` (14/14):

- **Atomic recording with full provenance:**
  - one entry, no start or end time, reported time kept, the talk's outcome
    unchanged;
  - `checkRun` replays the record;
  - one `completed` event;
  - a retransmission writes nothing.
- **Refusals that write nothing:** a missing file, an unknown commit or
  entry, an unknown presentation or template, a predecessor that is still
  running, or an existing id. Each leaves no revision file and no record.
- **Interaction with the reminders:** the self-check clears; an unlinked
  recorded work node still triggers `presentation-missing`; a linked
  presentation satisfies it.
- **CLI:** help, recording, and `wfe run show` output.
