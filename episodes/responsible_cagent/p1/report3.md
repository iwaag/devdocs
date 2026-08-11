# Step 3 report — Nautobot self-hosting posture

Status: complete.

## Posture

Nautobot is the cluster intent store and cannot be its own availability
authority. Availability is therefore split deliberately:

- launch/recovery is owned locally by the container engine and Compose
  (`restart: unless-stopped`);
- visibility is owned by six `manual` placements in Nautobot, so cagent can
  explain the stack without claiming nctl can repair its own store while it is
  unavailable;
- recovery evidence is a PostgreSQL dump plus this cold-start procedure.

Registered manual placements on agstudio for `postgres`, `redis`, `nautobot`,
`nautobot-worker`, `nautobot-scheduler`, and `minio`. Public service endpoints
record :5432, :6379, :8000, and :9100. The structured preview reported 16
creates, zero conflicts/errors and no commit; the subsequent `--yes` apply
committed the same 16 creates. `nctl relations` then showed all six placements
with `management_mode: manual`. Their observation gaps are expected until the
Step 5 refresh.

## Start-order proof

Tested a deliberately reversed cold start:

1. stopped the Nautobot and dependency Compose projects;
2. started the Nautobot web container alone with no dependencies;
3. confirmed it remained under the restart policy and logged Redis connection
   refusal rather than corrupting state;
4. started PostgreSQL and Redis;
5. ran the full Nautobot Compose `up -d`;
6. observed PostgreSQL/Redis healthy, then Nautobot web/worker/scheduler
   healthy and host `/health/` HTTP 200.

There was one short window where container health had become healthy but the
host HTTP socket still returned an empty response while uWSGI was entering its
serving state. Repeated verification reached HTTP 200 seconds later. The
operational check is therefore the host HTTP response, not container health
alone.

## Cold-start runbook

From the `pj-clusterintent` root:

```sh
# 1. The desktop container engine (OrbStack on agstudio) must be running.

# 2. Start stateful dependencies first.
docker compose -f devenv/nautobot-dependencies/compose.yaml \
  --env-file devenv/.env up -d
docker compose -f devenv/nautobot-dependencies/compose.yaml \
  --env-file devenv/.env ps

# 3. Start the intent store, workers and object store.
docker compose -f devenv/nautobot/docker-compose.yml \
  --env-file devenv/.env up -d
docker compose -f devenv/nautobot/docker-compose.yml \
  --env-file devenv/.env ps

# 4. Verify the external boundary and worker queue.
curl -f http://localhost:8000/health/
uv run --project nctl nctl status --json
```

Expected final state: PostgreSQL and Redis healthy; Nautobot web, worker and
scheduler healthy; `/health/` HTTP 200; `nctl status` reports Nautobot
reachable/authenticated and at least one Celery worker with no stale pending
jobs. MinIO can be checked independently with
`curl -f http://localhost:9100/minio/health/live`.

If PostgreSQL state is damaged, the pre-change dump is the ignored
`.local/backups/responsible-cagent-step2.dump`; normal scratch recovery uses
the existing `.local/backups/` convention. Do not copy tokens or Compose env
values into a report.

Documented Nautobot's self-hosting boundary for cagent — handoff candidate.
