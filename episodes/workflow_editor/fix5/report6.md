# fix5 step 6 — deployment, trials

## State read before changing services

- `nctl status`: ok (Nautobot, worker, dumps).
- WFE served `36588fd8`. The executor's heartbeat was current.
- `wfe observe`: no run was executing. The only held run was jevlab
  experiment/run-001, held by its person.

## Releases

| Component | Revision | How |
| --- | --- | --- |
| WFE (agdevworld) | `506a116` (fix5 releases `3d569db` → `f6d8c71` → `cb2f2a9` → `67de4f5` → `506a116`; previous `36588fd8`) | `scripts/wfe-release.sh deploy` each time: built, checked (558/558), service and `bin/wfe` healthy on the new commit |
| agautolab (wfrun guide) | `8f341ed` (previous `881ca9f`) | the executor reads the guide from its checkout |
| agpc worker | wfe `506a116`, executor `8f341ed` | nctl batch on placement `agautolab-agpc`, `render production`, `setup_wfe_worker.yml -l agpc` (last run ok=22 changed=3); the unit is active |

A store backup was taken before the first release. No store schema
changed. The run schema gained `run.record-done`, `retrospective` and
`links`, and a release before fix5 cannot replay a record that contains
them. The recovery procedure (activating the previous release, restoring
affected run.json files) is in the ignored `pj-agdev/.local/fix5/notes.md`.
No registered project holds such a record.

Access from another LAN host (agpc):

- the coordinator's `/api/version` names `506a116`;
- the browser path `/wfe/` answers 200;
- the worker's own `wfe run help record-done` shows the new command;
- the run view of the retrospective fixture, served by the deployed
  release on a temporary port, answered 200. Its API showed the node
  completed with `started: null` and the evidence kinds.

## The four follow-up releases: what the deployed checks found

1. **`f6d8c71`: the self-check's repair could not be applied.** The check
   fires when a talk completes, but `record-done --after talk` was refused
   because the talk was not yet completed. A recorded node may now follow
   nodes that are completed or under way (running, waiting), never future
   work.
2. **`cb2f2a9`: generated steps.** In freework (final report), adding a
   node moves `_final-report`'s edges. The revision was refused as
   "changes other edges". Edges of generated steps may now move, as may
   the direct edge a placement between two nodes bypasses.
3. **`67de4f5`: `--commit` was dropped.** `--commit` is a boolean flag of
   other commands, so the parser lost its value. The option is now
   `--commit-id`.
4. **`506a116`: reported times.** A reported work time after the recording
   is refused. (Two scratch fixture runs made under `67de4f5` with such
   times no longer replay. They are scratch only.)

Each fix has a test (16 reminder tests, 558 in all).

## Trial: gimmickimg freework/run-012 (a real run)

The request came from the check actor `fix5-check` and said so in its
text. It asked for a still strip comparing the sway clips and to "tell
fix5-check when it is done". It did not mention presentation or cleanup.

| Seq | What happened |
| --- | --- |
| 5–6 | The agent instantiated the catalog's `work` template as `strip` and adjusted the run **before** doing the work, then started `strip` (fix4's improvement held) |
| — | It built `experiments/sway-strip/sway_strip.png` with ffmpeg and Pillow |
| (refused) | `wfe run complete … strip` returned `presentation-missing`: "strip is work this run completed and the run has no presentation…", with the instantiate and present commands |
| 7 | **The agent repeated the command with `--yes` at once.** The log shows no deliberation and no attempt to present |
| 9–11 | It gave `--yes` pre-emptively to `_final-report` and to `talk`, whose completion completed the run |

Result: the run is "Done · every step completed". There is no presentation,
nothing is served, and the result was integrated. The image is a result
the person can look at, and the reminder said so, but it was not
presented. This is the limitation the plan named and accepted: an agent
may learn to use `--yes` immediately. Nothing was added to hide the flag
or to enforce reflection. The agent also told fix5-check nothing beyond
the completion outcome and the report.

No agent-authored feedback was needed, because the run asked nothing. The
Developer was not impersonated.

## Retrospective recording with known fixture work (deployed release)

This used a scratch area and a project registered only there, with
fixture work already performed: a file written and committed while the
talk was running. The actor was `fix5-check`, and the CLI was the
deployed release's.

| Step | Result |
| --- | --- |
| `wfe run complete … talk` | `unrecorded-activity`: "the graph shows no work node, yet artifacts outside the run folder are recorded … WFE cannot tell whether work was done …" |
| `wfe run record-done … --template work --id count-words --after talk --artifact … --commit-id <sha> --entry 3 --started-at …` | "Recorded afterwards: count-words is completed in r2 (nothing was run). Evidence: experiments/fixture/result3.txt, commit 63f5e3c418, entry 3" |
| the same command, same receipt | "Already recorded (same receipt)" |
| `wfe run complete … talk` again | `presentation-missing` for `count-words`: declaring the work done did not satisfy the presentation reminder |
| the same with `--yes` (the check actor's judgement: a one-line text file needs no page) | recorded, the run completed |
| record | `count-words` completed with `started`/`ended` `null`, `retrospective` holding the reason, reported start, artifact by hash, full commit id and entry; change `[null, 'completed']`; `wfe run check` usable |
| run view (run-004) | dashed card "Done · recorded afterwards" and a details box "Recorded afterwards: fix5-check … Nothing ran this node … Work time as reported … Evidence …" ([screenshot](run-view-retro.png)) |
