# better_communication — Step 3 report: organization, accounts, channels

Date: 2026-08-12. Status: **complete**. The `Agent Studio` organization has
one human owner, six agent identities, and the two planned channels.

## Organization and identities

- Human owner: `Developer` (`developer@agstudio.local`), role 100 (owner).
  Its randomly generated password and API credential are local-only mode-0600
  files.
- Generic bots: `Omni Agent`, `Devworld Assistant`, `Autolab Agstudio`,
  `Autolab Agautolab1`, `Forge`, and `Cagent`. An API read-back showed all six
  active as bots with member role 400 and a 32-character API key.
- `ops` and `sandbox` exist and the provisioning request subscribed the human
  and all six bots to both.

Omni Agent is a generic bot rather than a full account. This episode needs a
stable API identity that can send and read, while the human owner account
already provides browser administration. A second interactive login would add
password lifecycle without improving participation coverage.

Credentials live in per-identity env files under the ignored
`pj-agdev/.local/zulip/`; the cagent credential was also copied to the ignored
`pj-clusterintent/.local/zulip/`. All files contain only `ZULIP_URL`,
`ZULIP_EMAIL`, and `ZULIP_API_KEY` keys and have mode 0600; values were not
printed or placed in Git.

## Browser post verification

A real Chromium session accepted the self-signed certificate, logged in as
Developer, completed the first-login screens, opened `sandbox`, composed topic
`participation-check`, and sent `hello from developer (browser check)`. The UI
displayed the message and a screenshot was retained in the ignored deployment
evidence directory. An API read-back independently found the same message with
sender `Developer` and the correct topic.

The provisioning work hit two recoverable Zulip 12.2 API-shape differences:
the owner authenticates with delivery email rather than its anonymized API
email, and `POST /bots` returns an empty success while `GET /bots` contains the
created credential. The local provisioning script now handles both and is
idempotent for existing identities/channels.

## Agent run record

- Request/job id: `better_communication/step3`
- Backend: Codex harness + GPT-5
- Outcome: done
- Cost/time: not reported by the harness
