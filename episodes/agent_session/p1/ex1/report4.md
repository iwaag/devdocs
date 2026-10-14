# Agent session p1 ex1 — step 4: deployment

Date: 2026-10-14, 18:12–18:15 JST.

## Before activation

- `nctl status`: Nautobot 3.1.3 authenticated, worker idle, submodules
  clean.
- `wfe observe`: executor available and idle, nothing queued, no open
  attempt. The only held run was jevlab `experiment/run-001`, held
  `stopped` by iwaag at seq 11.
- Backup: exec, notify, intake and present stores via `.backup`, plus the
  registry, system and zulip JSON files. The location is in the ignored
  host notes.

## Activation

1. `wfe-release.sh deploy` built agdevworld `75cdae7`. It ran the mandatory
   `npm ci && npm run check` once (**308/308**, type check, build), switched
   `service/current` from `dab1e54`, restarted the editor service, and
   checked that the service and `bin/wfe` both report `75cdae7`.
2. The executor was restarted on agautolab `043c082`. It logged its usual
   start-up lines and no failed tick. Its first reconcile was clean.
3. Observer was restarted on pj-agdev `a9e71e2`. Its first look observed
   1 run and 0 open incidents.

## After activation

| Check | Result |
| --- | --- |
| Release identity | `GET /wfe/api/version` and `bin/wfe version`: `75cdae7`; `/wfe/` HTTP 200 over the LAN name |
| Executor | `available`, idle, heartbeat 2 s old, pid of the new process |
| Observer and watchdog | watchdog `observer: ok`, `executor: available`; health has 0 open incidents |
| Human hold | jevlab `experiment/run-001` still `stopped` by iwaag, seq 11, unchanged |
| `wfe observe` problems | none |

There was no store migration. No live workflow trial was run (the plan does
not require one after the focused regressions). The final report is
[report.md](report.md).
