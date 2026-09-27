# progress_panel p1 — step 6: consolidation

Date: 2026-09-27 (UTC 11:10–11:30).

## Removed or retired

- Temporary trial aids: the vite server on `:5179` is stopped and the three
  CDP profiles (`/tmp/pg*-profile`) are deleted. The trial directories
  (`agdevworld/.local/pg/trial/`, `trialB/`: logger, watcher scenarios,
  `board.jsonl`, `ui.jsonl`, screenshots) stay as the ignored record; the
  drivers and scenarios under `.local/pg/` stay for the next check.
- Dead code from the phase: two unused constants in `agag.progress`, an
  unused import in the relay's `progress.py`. Nothing older was superseded:
  the panel adds a read; the trace changes replace rules in place (a
  self-served run's state, `silent`/`resolved_live`/holder/receipt rules)
  and their old behaviour has no other caller.
- Nothing was removed from Observer's state: its review topics from the
  trials carry their later outcomes and are left for the Developer's ✔.

## Documentation

- `devdocs/README_DEV.md`: a new section *The progress panel*; the Routines
  section says how a run completed from the request's conversation is ended
  (`agrunfinish`); the archsage studies section names the `sagesync` record.
- `pyagag/docs/progress-v1.md`: the read model's contract; the trace's
  module docstring names the facts it now carries for progress readers.
- `agdevworld/README_DEV.md`: *Progress panel* under the Front Desk
  (polling, drawing rules, the demo and snapshot hooks).
- `pj-agdev/.local/devenv.md` (ignored): route, configuration, version set,
  drivers, trial records, residue.

## State left

| repository | head |
|---|---|
| pyagag | `29f3253` |
| agdevworld | relay on pyagag `29f3253`; web image on `:8090` carries the panel |
| agfront | `agrunfinish`, its grant and the guides; pyagag `03f8fc9` |
| agautolab, agforge, archsage, cagent, agobserver | pyagag `03f8fc9` |
| pj-agdev, pj-clusterintent, devdocs | pushed, clean |

Every consumer's suite passes at its pin (pyagag 1018, agfront 185,
agautolab 331, agforge 265, agobserver 169, archsage 38, cagent 204, relay
362); `npm run build` is clean; `nctl drift` converged=46. All services are
running on the committed code (last restarts: listeners 10:23Z, Observer
10:09Z, relay 11:20Z).
