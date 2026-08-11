# Step 4 report — Intent registration and DNS/DHCP sync

Status: in progress; waiting for one operator-authenticated DHCP renewal or
reboot on agautolab1.

## Completed

- Re-exported the current 70-operation desired state to the ignored
  `.local/desired-state.yaml` and appended every remaining keeper.
- Preview: 25 creates, 70 unchanged, zero conflicts/errors, not committed.
- Apply: the same 25 creates committed. The cluster now has 27 placements.
- Added manual placements for Gitea, Plane, agdevworld web/assistant, agforge,
  cagent OpenCode/API, ACE-Step and music-gen, with public endpoints where the
  service is LAN-facing.
- Confirmed the existing agautolab1 primary endpoint is DHCP-reserved
  `192.168.0.130` for MAC `bc:24:11:7a:b1:09`.
- `nctl render dnsmasq` rendered both the host record and DHCP reservation at
  `.130`, content SHA-256
  `17f4168f11809cc12d58c96975911b1e51503d5bf1ca5bf7c678392805fa5b89`.
- Reviewed the direct apply plan, then ran `nctl apply dnsmasq --yes` through
  the bootstrap inventory. SSH preflight was ready; setup and deployment both
  completed with zero failed/unreachable hosts; dnsmasq restarted active on
  agdnsmasq.

## Remaining transition

agautolab1 still holds its pre-change DHCP lease at `192.168.0.220`; the lease
file on agdnsmasq and `ip addr` on the node agree. The node requires privileged
DHCP renewal. Ansible connectivity works, but `sudo -n` is correctly denied and
`networkctl renew ens18` requires interactive authorization. No credential was
guessed or embedded.

Operator action requested:

```sh
ssh eiji@agautolab1.local 'sudo networkctl renew ens18'
```

A reboot from the Proxmox console is an equivalent alternative. After that,
this step will verify `.130`, hostname resolution, rerun the Gitea-backed
`setup_autolab_node.yml --limit agautolab1` path, and finalize the report.
