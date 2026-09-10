# refactor p3 step 4 — Plane stopped, proved unnecessary, and removed

## What was asked

Find Plane in desired and observed state, stop expecting it, deploy the
changed packages and schema, stop the service, prove the retained workflows
run without it, then delete the deployment — without taking another stack's
database with it.

## What Plane was, read rather than assumed

`nctl desired export` and the ignored environment notes agreed on three
declarations and one stack:

| what | where |
|---|---|
| `desired_service` | `slug: plane`, lifecycle `active` |
| `desired_endpoint` | `agstudio` / `plane` / `service`, `http`, port `8290` |
| `desired_service_placement` | `plane-agstudio`, profile `manual_toolchain`, management `manual` |
| the stack | Plane CE v1.4.1, Docker Compose from the ignored `plane-selfhost/plane-app`, 13 containers, 10 `plane-app_*` volumes, the `plane-app_default` network |
| credentials | `plane-credentials.env` (two copies), four per-agent env files and three account scripts |
| startup | none — no launchd job, no LaunchAgent plist, no crontab entry; the stack was started by hand |

**Stack ownership was established before anything was deleted.** Plane's
PostgreSQL, Valkey, RabbitMQ and MinIO were internal to its own Compose
project. The realm's other stores are separate stacks —
`zulip-selfhost-database-1`, `zulip-selfhost-redis-1`, `my_postgres_db`
(Nautobot's), `nintent-redis`, `nautobot-minio-1` — and every one of them is
still running.

## What was done, in order

1. **Stopped expecting it.** `.local/p3-retire-plane.yaml` deleted the three
   declarations through `nctl desired apply` — preview `delete: 3,
   conflict: 0`, then `--yes`. The same three blocks were removed from the
   operator input `.local/desired-state.yaml`, which re-previews
   `unchanged: 86, conflict: 0`.
2. **Deployed.** The Nautobot image was rebuilt from the pushed nintent
   revision and migration `0033` applied (step 2). `agautolab1` — the node
   p1 and p2 deliberately left alone, still at `agent_standardize` p3 —
   was redeployed: `~/agautolab` to `b38a6af`, `~/agecho` to `3c9e77f`,
   neither venv containing `agag/plane.py`, and **no `plane*` file anywhere
   under `~`**, because the role now deletes `.local/plane.env` rather than
   merely stopping to write it. Every local venv was re-synced to pyagag
   `0d26145` and every listener restarted; the frontend was rebuilt and
   redeployed to `:8090`.
3. **Stopped Plane.** `env -u DEBUG ./setup.sh stop`. Before:
   `http://localhost:8290/api/instances/` → `HTTP 200`. After: curl exit 7,
   connection refused. Every container gone.

## The verification, run while Plane was unavailable

**A missing Python import is not the evidence here** — the credential files
were deleted and the service was down for all of it. Where the import graph
is cited it is *supporting*: `agag.plane` cannot be imported by anything any
more, because the module does not exist.

### cagent registers and answers a change request

`#cagent-agstudio1 › p3-step4-plane-stopped` (message 5882), a deliberately
harmless documentation-only request that needed no infrastructure change and
says so. cagent answered:

```
recorded c5885 "Record that Plane is no longer part of this environment
(documentation-only)" in #**cagent-agstudio1>change-p3-step4-plane-stopped-o5882**
```

Read back through the anchor alone: `c5885`, at
`cagent-agstudio1/change-p3-step4-plane-stopped-o5882`, origin `5882`,
statement `5887`, state `requested`.

### Registration observation, ingest and drift

`nctl agents observe` printed all seven agents and `ingested through the
Nautobot Job`. `nctl drift` holds **no target and no diff code containing
`plane`**, and still reports the two real Zulip problems
(`agent_zulip_channel_unsubscribed` on `agforge` and `agecho-agautolab1`).

### One autolab → forge → autolab request, through completion

`#pj-refactorp3`, opened for this run; every message below is one a person
would have made.

1. **Plan.** `workplan-frame` → mission **m5894**, two tasks: get a 256x256
   PNG frame icon from forge and commit it, then log its provenance.
2. **Start**, then task 1 in `work-m5894 › workrun-task1-m5894`.
3. **Delegation.** autolab opened `#agforge-agstudio1 › assetplan-frame-icon`
   and asked. forge planned it and answered:

   ```
   recorded a5919 "Plan: Generate assets/frame.png" (toolsets: toolset-image)
   posting in assetrun-frame-icon-a5919 starts it
   ```

   — the p2 identity scheme, live, with Plane down. autolab triggered the
   run; forge generated through SwarmUI and delivered into the `assetplan-`
   topic with a presigned URL and
   `[S3KEY] files/2026-09-10/d1e58460ee6f4f6398d6158399e29e0f.zip`.
4. **The delivery is the callback.** It named autolab, which resumed
   `workrun-task1-m5894`, downloaded the zip, checked the PNG, and **said
   the image was not the style it asked for** — ornate rather than flat —
   listing the two options rather than deciding. Told to accept it, it
   committed `main/assets/frame.png` (`c2cc09d`).
5. **Task 2** added the devlog note citing the request and the S3 key
   (`13afbef`), and both repositories were pushed after confirmation.
6. **Inspect.** The artifact is real and meets the hard constraints: `PNG
   image data, 256 x 256, 8-bit/color RGB`, an empty picture frame in a
   copper accent on black. It is in the repository and in the object store.
7. **The UI.** The operation room's completion door, read twice — once while
   the work was in flight and once when it was finished:

   ```
   ready  work         work:m5894                       every one of its 2 tasks is finished; accepting them
   ready  work         work:a5919                       its asset was delivered; accepting it
   done   topic        …/workrun-task1-m5894            already ✔
   done   topic        …/workrun-task2-m5894            already ✔
   ready  topic        …/assetplan-frame-icon           will be marked ✔
   ready  topic        …/assetrun-frame-icon-a5919      will be marked ✔
   ready  channel      channel:work-m5894               will be archived
   ready  conversation …/workplan-frame                 will be marked ✔ once everything above is done

   counts {'ready': 6, 'done': 2}
   status {'zulip_read': True, 'zulip_write': True, 'reason': ''}
   gaps   {'truncated': False, 'unread': [], 'bounded': [], 'errors': []}
   ```

   Two work records, both read from the conversations, **no Plane row, no
   credential warning, no gap — and no `issue_id` field at all**, which is
   step 3's removal showing up in the payload. Applying it:

   ```
   applied work m5894 · applied work a5919 frame-icon
   applied topic assetplan-frame-icon · applied topic assetrun-frame-icon-a5919
   applied channel #work-m5894 · applied conversation workplan-frame
   partial: False
   ```

8. **Read back through the anchor after everything closed.** `a5919` →
   `state='accepted'`, stem `frame-icon`, tools `['toolset-image']`, results
   `('files/2026-09-10/d1e58460…zip',)`, in a conversation that is now `✔`.

**Cost**: 19 agent runs, **$2.85**.

## Removal

- 13 containers and the `plane-app_default` network went with the stop.
- 10 volumes deleted by name: `plane-app_pgdata`, `plane-app_uploads`,
  `plane-app_redisdata`, `plane-app_rabbitmq_data`, the four `logs_*`, both
  `proxy_*`. Each was confirmed to have no container attached first.
- The six `makeplane/*:v1.4.1` images.
- `.local/plane-selfhost/` (the Compose project and `setup.sh`),
  `.local/plane/` (four per-agent env files, `plane_agent_account.py`,
  `plane_agent_rotate.py`, `plane_agent_retire.py`), and **both** copies of
  `plane-credentials.env`.

No archive was kept. The plan permits that, and the data was disposable
experimental state in a private environment.

## Confirmation

```
containers matching plane : none
volumes matching plane    : none
networks matching plane   : none
images matching plane     : none
:8290, :8490              : curl exit 7 (connection refused)
launchd / LaunchAgents / crontab mentioning plane : none
nctl drift                : no plane target, no plane code
```

Nothing will recreate it: the stack had no supervisor, the desired state no
longer declares it, and the Ansible roles that used to install its credential
now delete one.

**Gates.** nctl 1335, nintent Django-free 147 (10 expected skips), nauto 121,
cagent 198, mTLS conformance 23, Nautobot runtime `--clean` `cases=263`,
pyagag 515, agautolab 237, agforge 221, agfront 103, agentroom 267.

## Unrelated drift, reported not repaired

- `service_missing` on the `agfront` **service** target, from an observation
  timestamped 12:12Z — before this step touched any service. It is not
  Plane's and was not expanded into.
- `agent_zulip_channel_unsubscribed` on `agforge` and `agecho-agautolab1`:
  real declared-channel gaps, and exactly the kind of report that had to keep
  working. Left as they are.
