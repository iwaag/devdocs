# better_communication — Step 1 report: preflight and placement

Date: 2026-08-12. Status: **complete**. Zulip will run on agstudio, using host
ports 8380 (HTTP) and 8543 (HTTPS). A Proxmox guest is not needed.

## Evidence and decision

- The host has roughly 1.4 TiB free on its data volume.
- Docker Desktop exposes 16.8 GB RAM to containers. The existing containers
  used roughly 8 GB at the snapshot; the largest users were the Nautobot
  worker (about 4.0 GB) and Plane worker (about 2.2 GB). This leaves enough
  practical headroom for Zulip's approximately 2 GB requirement without
  moving or reconfiguring Plane or any other service.
- `docker system df` showed about 9.5 GB of images and only 276 MB of named
  volume data. Host disk, rather than Docker data, is nowhere near pressure.
- Listener checks found TCP 8380 and 8543 free. The existing containers and
  their published ports were inventoried before choosing them.
- Because this work refers to the live local environment, `nctl status
  --json` was also checked. Nautobot 3.1.3 was reachable and authenticated,
  intent GraphQL was available, one worker was running with no pending jobs,
  and the command returned `ok: true`.

The placement and reserved ports are recorded in the ignored
`pj-agdev/.local/devenv.md`. No tracked file contains local host/IP details or
credentials.

## Agent run record

- Request/job id: `better_communication/step1`
- Backend: Codex harness + GPT-5
- Outcome: done
- Cost/time: not reported by the harness
