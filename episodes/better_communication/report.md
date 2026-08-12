# better_communication — final report

Date: 2026-08-12. Outcome: **complete**. A self-hosted Zulip 12.2 now runs on
the local workstation as the future agent communication hub. One human owner,
an Omni Agent bot, and five in-system-agent bots can all participate in the
minimal `ops` and `sandbox` channels. Every actual runtime sent and read a
message with correct sender identity, and the browser UI presents the complete
conversation naturally to a human.

## What is running

The official Docker Zulip stack is isolated under an ignored local deployment
directory. PostgreSQL, Redis, RabbitMQ, memcached, application data, and the
self-signed certificate persist in stack-local named volumes. Only the chosen
web ports are published; dependency and SMTP ports are internal. Outgoing
email is console-only, while login email, mobile push, and Zulip service
usage-statistics submission are disabled.

The full-stack restart test passed, as did final health and LAN browser checks.
Plane, Nautobot, and all prior REST routes were left intact. Final nctl status
was healthy.

## Identity and runtime proof

The organization contains `Developer` plus these generic bots: `Omni Agent`,
`Devworld Assistant`, `Autolab Agstudio`, `Autolab Agautolab1`, `Forge`, and
`Cagent`. Generic bot was chosen for Omni because it supplies full API send/read
semantics and sender identity without another interactive password account.
All six bots subscribe to both channels.

In `sandbox` / `participation-check`, message IDs 15–20 are the six runtime
hellos. Reads from the Omni host shell, assistant container, both autolab
vantage points, forge environment, and cagent environment each returned all
six sender names and bot API email identities. Chromium independently asserted
the six displayed names and retained a private screenshot.

## Credential and implementation footprint

All credentials are ignored, mode-0600 env files. The assistant consumes a
read-only secret mount; local native services use their owning project's
`.local/`; the remote autolab node receives its credential via an Ansible
`no_log` copy patterned after the existing Plane delivery mechanism.

Tracked implementation commits:

- `agdevworld` `747cc39`: assistant credential mount
- `pj-agdev` `3a0d80a`: updated agdevworld pointer
- `ansible_agdev` `035da2a`: autolab Zulip credential delivery
- `pj-clusterintent` `e8603f2`: updated ansible pointer

The provisioning and participation helpers remain ignored local tools. They
were useful during failure farming: Zulip 12.2's bot-create response and owner
authentication email differed from the initial assumption, and the first
bulk subscription attempt gave public read access without bot subscription.
The helpers now handle those shapes, and every bot's own API confirms both
channel subscriptions.

## Scope boundary and follow-up

This episode proves the communication substrate, not a workflow. No mission
dispatch, report routing, Plane write-back, agent-code chat client, or legacy
route removal was implemented. The next episode should connect chat origin to
durable Plane task state using `topic = issue/mission ID`, define retry and
transcript ownership, and retire string-telephone paths only after their
replacements are demonstrated end to end.

Step evidence is in `report1.md` through `report6.md`. Exact local endpoints,
commands, credentials, and screenshots stay in ignored local state.

## Agent run record

- Request/job id: `better_communication`
- Backend: Codex harness + GPT-5
- Outcome: done
- Cost/time: not reported by the harness
