# Step 4 report — Intent registration and DNS/DHCP sync

Status: complete.

## Intent registration

- Re-exported the current 70-operation desired state to the ignored
  `.local/desired-state.yaml` and appended every remaining keeper.
- Preview: 25 creates, 70 unchanged, zero conflicts/errors, not committed.
- Apply: the same 25 creates committed. The cluster now has 27 placements.
- Added manual placements for Gitea, Plane, agdevworld web/assistant, agforge,
  cagent OpenCode/API, ACE-Step and music-gen, with public endpoints where the
  service is LAN-facing.

## agautolab1 address transition

- Confirmed the primary endpoint reservation `192.168.0.130` for MAC
  `bc:24:11:7a:b1:09`.
- `nctl render dnsmasq` rendered the host record and DHCP reservation at `.130`;
  content SHA-256 was
  `17f4168f11809cc12d58c96975911b1e51503d5bf1ca5bf7c678392805fa5b89`.
- Previewed and applied the dnsmasq deployment. The first application used the
  bootstrap inventory, whose defaults omitted the placement's LAN-listen and
  DHCP variables. That left dnsmasq listening for DNS on loopback only. This was
  corrected by recording `enable_dhcp: true` in desired placement config and
  rerunning `setup_dnsmasq.yml` with both authoritative settings:
  `dnsmasq_listen_addresses=[192.168.0.2]` and `dnsmasq_enable_dhcp=true`.
- During the transition the replacement router at `192.168.0.1` was also found
  serving DHCP, and assigned `.83`. The operator disabled that unintended DHCP
  server. No address conflict or stale second DHCP authority remains.
- After reboot, agdnsmasq leased `192.168.0.130` to the expected MAC. Verified:
  `dig @192.168.0.2 agautolab1.home.arpa` returned `.130`, mDNS resolved `.130`,
  ping succeeded, the node reported `192.168.0.130/24`, and its gateway health
  endpoint returned `{"ok":true}`.

## Observation and deployment path

- Forced a fresh agautolab1 observation and ingestion. The resulting production
  inventory selects `.130` and retains the command-node Gitea URL.
- Ran `setup_autolab_node.yml --limit agautolab1` using that generated inventory.
  Result: `ok=23`, `changed=0`, `unreachable=0`, `failed=0`; the Gitea checkout,
  pinned agent tools, overlay, cagent client, systemd user unit and gateway
  health verification were all successful.

The intended authority split is now explicit: `192.168.0.2` is the LAN DHCP/DNS
service and `192.168.0.1` remains only the upstream router/default gateway.
