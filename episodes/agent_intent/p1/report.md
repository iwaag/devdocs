# agent_intent p1 — Phase report

AI-generated (Omni Agent, 2026-08-17). Steps 1–5 of `p1/plan.md` are done and
verified live. Step 6 was optional and the developer chose to skip it; the
developer also approved the agautolab1 Ansible run and the remaining
enrollments, which are done (see the end of this report).

## What exists now that did not before

An agag agent is a first-class desired/actual pair in cluster intent:

```
DesiredAgent  (nintent)          name, slug, lifecycle, workspace, placement,
                                 zulip_user_id, plane_user_id, desired channels
   |
   |  nctl agents observe        reads Zulip + Plane with real credentials
   v
ObservedAgentRegistration        what the realms actually say
   |
   |  nctl drift
   v
registration gaps (drift)  +  liveness_class (information, never a gap)
                                 ^
                                 |  nodeutils reads <workspace>/.local/agag-status.json
                                 |  pyagag writes it after every successful poll
```

Five agents are declared — `agforge`, `cagent`, `devworld-assistant`,
`autolab-agstudio`, `autolab-agautolab1` — each with its own Plane account and
personal API token. Four are enrolled end to end; three are live-`polling`.

| step | outcome | report |
|---|---|---|
| 1 | `DesiredAgent` model, batch kind, UI, nctl read/export | [report1.md](report1.md) |
| 2 | four per-agent Plane accounts, tokens, ritual, rows recorded | [report2.md](report2.md) |
| 3 | `nctl agents observe` + `Ingest Agent Registration` Job + observed model | [report3.md](report3.md) |
| 4 | `agag-status.json` in pyagag, picked up by nodeutils | [report4.md](report4.md) |
| 5 | `agent_evaluation.py`: gaps as drift, liveness as information | [report5.md](report5.md) |

## The decision the whole phase turns on

**Registration is drift. Liveness is not.**

Registration is deterministic and false-positive-free: an account either exists
in the realm or does not; a subscription either exists or does not. Liveness is
a claim about a process that may be restarting, mid-deploy, or deliberately
stopped, and treating it as drift would train everyone to ignore the drift
output. So `liveness_class` rides an info-severity diff that `derive_status`
structurally cannot turn into `drifting`, and no reconciler maps its code. The
tests assert the negative — every liveness state asserts `gaps == []` — so a
later promotion has to be deliberate.

The supporting honesty rules, each of which is a design property rather than a
convention:

- The status file is written **only** after a poll that actually returned, so a
  failing listener cannot refresh its own liveness. Staleness is the absence of
  a write, which no bug in the writer can fake.
- Ages are computed **within one machine at a time** — node-side file age plus
  controller-side collection age — so no two clocks are ever subtracted.
- Every "we don't know" carries a reason (`no_status_file` vs
  `workspace_unobserved` vs `no_desired_workspace`), because *we looked and saw
  nothing* and *we never looked* are different facts.
- The ingest Job **skips** a payload row naming an unknown agent rather than
  creating one: desired state is never invented from an observation.
- Observability is on by default at the agag convention path, so an agent that
  follows the protocol is observable without being configured — which is the
  braindump's actual thesis.

## Acceptance, live

All four criteria in the plan were demonstrated against the running cluster and
are recorded in [report5.md](report5.md) (1–3) and [report2.md](report2.md) (4):
converged+`polling`; listener stopped → `stale` with **no** gap; a wrong desired
channel → a registration gap; a Plane issue attributed to the agent's own
account.

## Gates

| gate | result |
|---|---|
| nctl ordinary | 1336 passed |
| nintent Django-free fast | 147, OK, 10 expected skips |
| nauto ordinary | 121, OK |
| nodeutils ordinary | 95 passed, 2 subtests |
| pyagag | 206 passed |
| Nautobot runtime reuse gate | "No changes detected", `cases=263`, OK |

## Deployed

nintent `3bbdb15` · nauto `a8778e6` · nctl `ef93b80` · nodeutils `d68b963` ·
pyagag `cb3e4d2` · agforge `b7bc722` · agautolab `3a92c2c` ·
pj-clusterintent `13e418b` · pj-agdev `8fc522e`.

Everything pushed to GitHub and reflected onto what consumes it: the Nautobot
container rebuilt and migrated, the nauto Git repository synced and the new Job
enabled, the agforge listener restarted.

**Deployment finding worth keeping:** `docker compose build` alone does *not*
pick up a new nintent commit — the `pip install git+…` layer is cached and the
Dockerfile did not change. `--no-cache` (or bumping the `NINTENT_BRANCH` build
arg) is what actually redeploys, and `/opt/nautobot/build_info.json` is how you
check rather than assume.

## Completed after the developer's decisions

The developer approved the Ansible run and the remaining enrollments, and chose
to skip step 6. All three are now done.

**agautolab1 now reports to Plane as itself.** `setup_autolab_node.yml
--limit agautolab1` ran with `AUTOLAB_NODE_PLANE_CREDENTIALS_SOURCE` pointed at
`.local/plane/autolab.env` (`ok=21 changed=4 failed=0`); the node's
`.local/plane.env` carries the per-agent key and its checkout is at `3a92c2c`.

One thing the role forced: `plane.env.j2` reads `PLANE_URL`, `PLANE_API_KEY`,
and `PLANE_WORKSPACE_SLUG` from **one** source file, while the per-agent files
held only the key. All four now repeat the URL and workspace slug, and the
creation script emits them. Deploying before noticing would have written a
`plane.env` with two empty values.

**Remaining enrollments.** pyagag pins bumped and listeners restarted for
cagent (`13e418b`) and agautolab (`3a92c2c`); new `DesiredWorkspace` rows
`agautolab-agstudio` and `agautolab-agautolab1` declared, and every agent
linked to the workspace its listener actually runs in. Current picture:

```
agforge             converged   liveness=polling
autolab-agstudio    converged   liveness=polling
cagent              converged   liveness=polling
autolab-agautolab1  converged   liveness=unobserved  reason=no_status_file
devworld-assistant  converged   liveness=unobserved  reason=no_desired_workspace
```

Every registration is converged. The two `unobserved` rows are the phase
earning its keep — both are real facts, not gaps:

- **agautolab1 runs no Zulip listener.** The node holds `.local/zulip.env` for
  the `Autolab Agautolab1` bot (user 12) and the bot is subscribed to its
  channels, but the `autolab_node` role installs only
  `autolab-gateway.service`. Nothing polls for that identity there. Recorded in
  `pj-agdev/.local/devenv.md`; whether to deploy a listener unit is a design
  decision, not a p1 fix.
- **The devworld assistant has no poll loop at all.** It uses
  `agag.zulip.ZulipClient` to send, never `serve`/`sweep_serve`, so there is
  nothing to write a status file. Its liveness is structurally `unobserved`,
  and declaring a workspace for it would only change the reason text, not the
  truth.

**Step 6 (Zulip presence): skipped**, per the developer. The status file stays
the source of record because it also works when Zulip is down, and a per-poll
presence POST has no reader yet.

## Deus Ex Machina notes

- Created four Plane accounts that in-system agents could arguably create for
  themselves — handoff candidate.
- Bumped pyagag dependencies and restarted the agforge, cagent, and agautolab
  listeners, which those agents' own maintenance loops could own — handoff
  candidate.
- Ran `setup_autolab_node.yml` against agautolab1, which the autolab agent could
  arguably drive for itself — handoff candidate.
