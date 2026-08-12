# better_communication — Step 5 report: participation check

Date: 2026-08-12. Status: **complete**. All six agent identities sent from
their runtime environment and all six independently read back the complete
sender set.

## Send evidence

All messages were sent to `sandbox`, topic `participation-check`:

| Runtime vantage point | Zulip sender | Message ID |
|---|---|---:|
| Omni/host shell | Omni Agent | 15 |
| native agstudio autolab environment | Autolab Agstudio | 16 |
| native forge project environment | Forge | 17 |
| native cagent project environment | Cagent | 18 |
| inside the running assistant container | Devworld Assistant | 19 |
| agautolab1 shell via production-inventory Ansible | Autolab Agautolab1 | 20 |

The assistant used its Node 26 runtime and read-only mounted secret. The other
five used Python stdlib from their host/project runtime; the remote script was
copied temporarily with mode 0700 and removed after verification. Self-signed
TLS verification was disabled only in these experimental checks.

## Receive evidence

After all sends, each of those same six vantage points fetched the latest
`sandbox` / `participation-check` messages with its own bot credential. Every
read returned message IDs 15–20 and the same visible sender mapping:

- `Omni Agent` — `omni-agent-bot@...`
- `Autolab Agstudio` — `autolab-agstudio-bot@...`
- `Forge` — `forge-bot@...`
- `Cagent` — `cagent-bot@...`
- `Devworld Assistant` — `devworld-assistant-bot@...`
- `Autolab Agautolab1` — `autolab-agautolab1-bot@...`

The domain is abbreviated here to avoid committing a local hostname. Full
read-back JSON stayed in the tool transcript; no API keys were present.

A Chromium check additionally asserted that all six full sender names were
visible in the conversation and saved a 1280x1011 screenshot at
`pj-agdev/.local/zulip-selfhost/evidence/step5-participation.png` (ignored,
because it contains local UI/environment evidence).

## Subscription correction found during visual verification

The first screenshot revealed that generic bots could read the public channel
but had not become subscribers through the administrator's bulk-principal
request. Each bot then subscribed itself to both `ops` and `sandbox`. API
read-back under every bot identity now returns both subscriptions. This makes
the Step 3 “everyone” requirement true as a subscription property, not merely
as public-channel readability.

## Agent run record

- Request/job id: `better_communication/step5`
- Backend: Codex harness + GPT-5
- Outcome: done
- Cost/time: not reported by the harness

DEM note: ran participation checks for all in-system agents — handoff candidate.
