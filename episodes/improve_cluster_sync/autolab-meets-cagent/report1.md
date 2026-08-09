# Step 1 report — agautolab1 enters production scope

Status: complete (2026-08-09).

`agautolab1` now has a fresh nodeutils observation, an explicit guest-OS
`Device` realization, and is included by production composition. The final
host-scoped drift contains exactly two converged targets (node and compute
instance); the node has only `intent_effect_summary`, whose production result
is `state: included` with no reasons.

## Execution

- Initial apply operation `01KZHZTCDN8YG3E6YSES62B8ZV` stopped safely at the
  SSH trust gate. The existing `agautolab1.local` entries in the operator's
  ordinary `known_hosts` were promoted with `nctl ssh enroll
  --from-known-hosts` (apply operation `01KZHZTRT6VKZ06S35MCZCEJ8M`).
- Observation operation `01KZHZTWM0TDVB09QDYM2ZSNZ3` collected the nodeutils
  report and ingested it. The scratch Nautobot worker had stopped consuming
  queued jobs despite reporting healthy, so only that worker was restarted;
  the pending ingest then completed.
- The first observation exposed stale intent: `accepted_actual_types` was
  `[virtual_machine]`, while the current guest contract uses a Nautobot
  `Device` for guest-OS realization and a separate `VirtualMachine` for the
  Proxmox compute realization. A one-operation desired-state batch was
  previewed and atomically changed it to `[device]`. A pre-change canonical
  export is retained privately at
  `.local/autolab-meets-cagent-step1-before.yaml`.
- Dry operation `01KZJ05249DSX61X1PSED5D2TG` then selected the unique Device
  candidate. Apply operation `01KZJ057AHF4YRWRG4TTHZJPYW` linked
  `DesiredNode.realized_device`, refreshed observation, regenerated production
  inventory, and finished `state: converged`.

## Evidence

- Durable successful reconcile evidence:
  `~/.local/state/nctl/events/01KZJ057AHF4YRWRG4TTHZJPYW/`
- Final independent drift capture (private cluster payload):
  `pj-clusterintent/.local/autolab-meets-cagent-step1-drift.json`
- Final result: `scope_summary: {converged: 2}`; production
  `state: included`; no `waiting_for_manual_initial_access`,
  `actual_node_not_linked`, or `no_realized_device` finding.

The observed address remains `192.168.0.220` while the desired DHCP reservation
is `192.168.0.130`, as anticipated by the plan while `agdnsmasq` is stale/down.
That optional network repair is deferred and is not a Step 1 completion gap.
