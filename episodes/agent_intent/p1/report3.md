# agent_intent p1 — Step 3 report: the registration collector and its ingest

AI-generated (Omni Agent, 2026-08-17).

## Shape

The same three-part shape as `observation.py`, for the same reason: the side
that holds credentials does the reading, and the side that writes to Nautobot
holds none.

```
nctl agents observe            reads Zulip + Plane, builds one payload
  -> Nautobot Job "Ingest Agent Registration"   validates, writes
     -> nintent ObservedAgentRegistration       one row per DesiredAgent
```

### What is read

- **Zulip** — `GET /users` to find the account behind `zulip_user_id`, then one
  `GET /users/{id}/subscriptions/{stream_id}` per *desired* channel. Accounts
  are matched on the numeric id, never on email, because the realm hides real
  addresses from events.
- **Plane** — the workspace members list, matched on `plane_user_id`, recording
  the workspace role integer.

A useful discovery: a plain **member** bot may ask whether another user is
subscribed to a stream it can itself see. No realm-owner credential is needed,
so the collector runs on the existing Omni Agent bot env file instead of
requiring a new privileged account.

`zulip_channels` records the desired channels **confirmed subscribed**, not the
agent's whole subscription list — enumerating another user's subscriptions does
need admin rights, and the drift question is only ever "is the agent on the
channels it must hear". A consequence worth stating plainly: a desired channel
that does not exist in the realm reads identically to one the agent simply is
not on. Both are a missing subscription, which is the honest answer for a
subscription gap.

### The observed model

`ObservedAgentRegistration` (nintent) — one row per `DesiredAgent`, replaced
each round: `observed_at`, `collector`, `zulip_present`, `zulip_user_id`,
`zulip_is_active`, `zulip_channels`, `plane_present`, `plane_user_id`,
`plane_role`. Deliberately minimal, per the plan's "don't over-model".

`observed_at` is the collector's single timestamp for the round, so staleness is
measured against one clock rather than against per-realm claims.

### Refusals built in

- The Job **skips** a payload row naming an unknown agent, with a reason,
  rather than creating a `DesiredAgent`. Desired state is never invented from
  an observation.
- Payload validation **rejects** rather than repairs: an unknown field, a naive
  timestamp, an empty agents list, a repeated slug, or a malformed channel list
  fails the ingest. A partially understood observation written as actual state
  is worse than no observation.
- The collector only reads. It never writes to Zulip or Plane.

### Configuration

New optional `[zulip]` and `[plane]` sections in `nctl.toml`. Each names a
`credentials_file` — an existing 0600 `KEY=value` file — rather than a token, so
no key is copied into a second store and nctl's "never a token in the config
file" rule stays intact. `[zulip]` also carries `verify_tls = false`, because
that realm serves a self-signed certificate on the LAN.

## Evidence

Live, against the running cluster:

```
$ nctl agents observe
agforge:            zulip=yes channels=[FreeForge, general, ops] plane=yes
autolab-agautolab1: zulip=yes channels=[general, ops]            plane=yes
autolab-agstudio:   zulip=yes channels=[general, ops]            plane=yes
cagent:             zulip=yes channels=[general, ops]            plane=yes
devworld-assistant: zulip=yes channels=[FreeForge, general, ops] plane=yes
ingested through the Nautobot Job
```

- The Job ran to `success` and five `ObservedAgentRegistration` rows are
  readable through GraphQL, all sharing one `observed_at`, with
  `plane_role: 20`.
- **Negative check**: adding a `no-such-channel` entry to agforge's desired
  channels and re-observing returned the same three real channels — the
  observation does not hallucinate a subscription, and the missing one is
  visible as desired-minus-observed. The desired value was restored afterwards.

Test gates:

| gate | result |
|---|---|
| nctl ordinary (`uv run pytest -q`) | 1322 passed (7 new collector tests) |
| nauto ordinary (`python3 -m unittest discover -s tests`) | 121 tests, OK (8 new payload tests) |
| nintent Django-free fast | 147 tests, OK, 10 expected skips |
| Nautobot runtime reuse gate | `cases=263`, OK |
| container `makemigrations --check` | "No changes detected" |

## Deployment finding worth keeping

`docker compose build` alone **did not** pick up the new nintent commit: the
`pip install git+…` layer is cached and the Dockerfile did not change, so the
rebuilt image still reported the previous `nintent_commit` and the new
migration was invisible. `--no-cache` (or bumping the `NINTENT_BRANCH` build
arg) is what actually redeploys. Verified by reading
`/opt/nautobot/build_info.json` in the container before and after — which is
exactly why that file exists, and why "I rebuilt it" is not evidence that a
commit is deployed.

Deployed: nintent `3bbdb15`, nauto `a8778e6`, nctl `e35bf89`. The nauto Git
repository was synced in Nautobot and the new Job enabled (Nautobot creates
Jobs from a synced repository disabled by default).

## Blemish

The nintent commit message for `ObservedAgentRegistration` contains a stray
character ("replaced每 collection round"). It is cosmetic, and rewriting a
pushed `main` tip to fix a typo is a worse trade than leaving it recorded here.
