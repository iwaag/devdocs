# better_communication — Step 2 report: Zulip deployment

Date: 2026-08-12. Status: **complete**. Zulip 12.2 is running as an
independent Compose stack on the local workstation and the LAN login page is reachable over
self-signed TLS.

## Deployment

- Cloned the official `zulip/docker-zulip` repository at commit
  `4e575b63ba2fdbaff1f2a6d1409d15ebfe4e6202` into the ignored
  `pj-agdev/.local/zulip-selfhost/` directory.
- Deployed image `ghcr.io/zulip/zulip-server:12.2-0` with the official
  PostgreSQL 14, Redis, RabbitMQ, and memcached services. All dependency ports
  remain internal; only the two ports recorded in the ignored local environment
  memo are published, and SMTP is not published.
- Persistent application, PostgreSQL, RabbitMQ, and Redis data use named
  volumes. Compose secrets are randomly generated in an ignored mode-0600
  `.env` file.
- Configured a ten-year self-signed certificate, a console-only Django email
  backend, disabled login emails, disabled the Zulip mobile-push service, and
  disabled Zulip service usage-statistics submission. Queue workers use the
  lower-memory threaded mode.
- No Plane, Nautobot, or other existing service was stopped or reconfigured.

## Verification

- `docker compose up -d zulip --wait` reported the Zulip application and all
  four dependencies healthy.
- Restarted the complete Zulip Compose stack, waited for all health checks,
  and confirmed the HTTPS endpoint still answered.
- After creation of the minimal Step 3 realm, a real Chromium run loaded the
  LAN login URL, found the username input, and captured
  a 1280x800 screenshot in the ignored deployment evidence directory. A
  separate LAN-name `curl -k` returned HTTP 200 and title `Log in | Zulip`.
- The live generated settings were inspected without displaying secrets and
  showed the console email backend plus false values for login email, mobile
  push, and usage-statistics submission.

One setup wrinkle was macOS Keychain being unavailable to the noninteractive
shell for GHCR. The public image was pulled with a deployment-local anonymous
Docker config; no keychain or registry credential was changed.

## Agent run record

- Request/job id: `better_communication/step2`
- Backend: Codex harness + GPT-5
- Outcome: done
- Cost/time: not reported by the harness
