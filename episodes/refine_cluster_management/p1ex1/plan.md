# Phase 1 Extra 1 Plan — Close out the Phase 1 leftovers

Source: [p1/report.md](../p1/report.md) "Left for phase 2", minus the cagent
responsibility work (that is phase 2 proper, planned separately in `p2/`).

Goal: resolve the three housekeeping leftovers so the intent/evidence layer is
noise-free before phase 2 builds on it:

1. Decide the fate of the catalog-only `haos` / `grafana` / `nomad` /
   `prometheus` / `prometheus-node-exporter` rows and of `agbach`.
2. Decide whether the agpc IPv6-RDNSS override should be generalized.
3. Remove preserved relic volumes whose recovery value is no longer wanted.

Ground rules (same spirit as p1):

- Experimental, non-production environment; no backward compatibility owed.
- Structured desired-state changes go through `nctl desired apply`
  (preview → `--yes`), not hand-edits.
- Everything else is implementer's discretion.

User decisions (2026-08-12): the catalog-only rows are leftovers of old
experiments — retire them. agbach is actually up — bring it back into
observation via nodeutils instead of retiring it. Relic volume deletion is
approved up front; no further checkpoint needed.

Write `report1.md` … `report3.md` (one per step) and a short closing
`report.md` in this folder.

## Step 1 — Catalog noise: catalog-only rows and agbach

Current state (p1/report5.md final drift): `converged=39 unknown=1 drifting=0`,
with 5 warnings from catalog-only services with no active placement and 1 error
`agbach: stale_actual_data` (no active placement, node currently unreachable —
a known state per pj-clusterintent local memos).

- Retire the five catalog-only rows — user confirmed they are old-experiment
  leftovers with no consumer. Go through the supported nctl path
  (`nctl desired apply` / catalog edit). If the enabled host services on agpc
  (Nomad, node-exporter) bother anyone they can be disabled too, but that is
  optional cleanup, not required for a clean drift.
- Bring `agbach` back into observation: the node is up. Run a nodeutils
  collection against it (the observation-refresh path / `nctl reconcile
  --refresh-observation` host-scoped run is the precedent from p1 Step 5),
  confirm `stale_actual_data` clears, and add it to the six-hour
  `com.clusterintent.observation-refresh` schedule if it carries (or should
  carry) any placement worth watching; if it hosts nothing, a fresh dump plus
  catalog presence is enough.

Deliverable: `report1.md` with the resulting `nctl drift` summary. Target:
zero warnings/errors that do not describe a real, actionable problem.

## Step 2 — IPv6-RDNSS override: generalize or document as one-off

p1 Step 5 fixed agpc only: the router's IPv6 RDNSS won over `192.168.0.2`, so
`home.arpa` names leaked to public DNS; the active NetworkManager profile now
ignores automatic IPv6 DNS and routes `~home.arpa` to the DHCP IPv4 DNS.

- Inventory which other nodes are dual-stack clients on this network and could
  hit the same failure (agautolab1, aghub, agdnsmasq itself, future VMs).
  Check per node how DNS is actually resolved (systemd-resolved /
  NetworkManager / static) using existing read-only channels — nodeutils dumps
  and ansible ad-hoc; no passwordless sudo on aghub, read-only there.
- If at least one other node is exposed: fold the fix into the ansible layer
  (e.g. a small role or an extension of the node setup playbooks in
  `pj-clusterintent/ansible_agdev`) so node setup produces the correct DNS
  posture instead of relying on a hand-applied per-host tweak; apply it to the
  exposed nodes and verify with a `home.arpa` resolution + binding probe.
- If agpc is genuinely the only exposed client: do not build machinery
  (Easier Next Time cuts both ways) — record the symptom, the fix, and the
  "how to recognize it" in the ansible/README or node docs so the next
  occurrence is a lookup, not a re-diagnosis.

Deliverable: `report2.md` with the exposure inventory and either the
generalized implementation + verification, or the documented one-off decision.

## Step 3 — Relic volumes: delete (approved)

p1 retired the relic containers but deliberately retained data: Keycloak,
RabbitMQ/Hatchet (+ its PostgreSQL), legacy MinIO (`my_minio`), both Portainer
volumes, duplicate legacy Nautobot volumes, and the pre-migration PostgreSQL
dump under the ignored local backup directory.

Deletion is pre-approved (see user decisions above) — no checkpoint.

- Enumerate the actual leftovers (`docker volume ls` + the legacy
  `~/services/service_scripts` bind mounts), verify each is one of the
  retired-relic remnants and not in use by a running container, then delete.
- One sanity check before deleting legacy MinIO data: confirm nothing in it is
  absent from the live Nautobot MinIO at :9100 (agforge presigns against
  :9100, so nothing should reference the old store).
- Keep the pre-migration PostgreSQL dump under `.local/backups/` — it is a
  backup file, not a relic volume, and it is the designated recovery artifact.
- Afterward, confirm the legacy `service_scripts` compose directory anchors
  nothing live and retire the directory itself if so.

Deliverable: `report3.md` with what was deleted and reclaimed space.

## Close

`report.md`: two or three paragraphs — final drift summary (should be
all-green modulo genuinely out-of-scope items), what was generalized vs
documented, what data was removed vs kept. Hand nothing further to phase 2.
