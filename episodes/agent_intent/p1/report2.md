# agent_intent p1 — Step 2 report: per-agent Plane accounts

AI-generated (Omni Agent, 2026-08-17).

## What changed

Every agent now acts in Plane as itself. Four member accounts — `agforge`,
`cagent`, `autolab`, `devworld-assistant` — each with its own personal API
token, replace the single shared `plane-agent-admin` key that made every issue
look like it came from the same principal.

No code changed. `agag.plane` reads one `PLANE_API_KEY`, so attribution is
purely a matter of which env file a deployment gets:

```
pj-agdev/.local/plane/<agent>.env    # 0600, git-ignored, one file per agent
  PLANE_AGENT_SLUG, PLANE_AGENT_EMAIL, PLANE_AGENT_USER_ID,
  PLANE_AGENT_PASSWORD, PLANE_API_KEY
```

The accounts hold the **same** workspace and project role (20) as the shared
account they replace. This step changes *who acts*, not *what an agent may do*;
folding a permission redesign into an attribution change would have made any
later failure ambiguous between the two.

## The creation ritual

Plane CE here has no email delivery — the original admin and viewer accounts
were made by pulling codes out of logs. Rather than repeat that, accounts are
created directly against the API container's Django shell, which is
deterministic and re-runnable:

```sh
# from pj-agdev, with the script below on hand
umask 077
docker exec -e AGENT_SLUG=<agent> -i plane-app-api-1 \
  python manage.py shell < plane_agent_account.py > raw.out
sed -n '/^RESULT_BEGIN$/,/^RESULT_END$/p' raw.out | sed '1d;$d' \
  > .local/plane/<agent>.env
chmod 600 .local/plane/<agent>.env && rm -f raw.out
```

`plane_agent_account.py` (idempotent; re-running an existing agent returns the
same account and mints no second token):

1. `User` — `<agent>@agstudio.local`, random password, `is_active`,
   `is_email_verified`, `is_password_autoset=False`;
2. `Profile` — onboarded, so the account is usable in the web UI too;
3. `WorkspaceMember` on `agautolab`, role 20;
4. `ProjectMember` role 20 on every project in the workspace;
5. `APIToken` labelled `<agent> agent`, scoped to the workspace.

It prints the resulting slug, email, user id, password, and token between
`RESULT_BEGIN`/`RESULT_END` markers, which is what the shell snippet above
slices into the 0600 env file. No token value was displayed in a terminal, a
tracked file, or sent anywhere.

It stays a manual, run-when-you-add-an-agent ritual, which the plan explicitly
allows: observation and drift stay automatic, and account creation happens once
per agent.

## Recorded into desired state

Five `DesiredAgent` rows now exist, created through
`nctl desired apply` and re-exported into the operator input:

| slug | zulip user id | plane account | placement | workspace |
|---|---|---|---|---|
| `agforge` | 13 | agforge | `agforge/agforge-agstudio` | — |
| `devworld-assistant` | 10 | devworld-assistant | `agdevworld-assistant/agdevworld-assistant-agstudio` | — |
| `cagent` | 14 | cagent | `cagent-api/cagent-api-agstudio` | `pj-clusterintent` |
| `autolab-agstudio` | 11 | autolab | `agautolab/agautolab-agstudio` | — |
| `autolab-agautolab1` | 12 | autolab | `agautolab/agautolab-agautolab1` | — |

Two findings the enrollment surfaced:

- **autolab is two Zulip identities, one Plane identity.** The realm holds
  `Autolab Agstudio` (11) and `Autolab Agautolab1` (12) — one listener per
  node — while the plan asked for a single `autolab` Plane account. So the
  model was populated per *listener identity*: two `DesiredAgent` rows sharing
  one `plane_user_id`. A Zulip account is what makes an agent addressable, so
  it, not the Plane account, is the natural grain.
- **Desired channels are the required minimum, not what is observed.** The
  bots are currently subscribed to ~17 channels each, almost all per-episode
  `pj-*` project channels. Declaring those as desired would turn every finished
  episode into permanent drift. `general`/`ops` for all, plus `FreeForge` for
  the two agents that actually serve requests there.

`desired_workspace` is null for four of the five: only `pj-clusterintent` has a
`DesiredWorkspace` row today, and pj-agdev has none. That absence is honest and
left visible rather than papered over; step 4 needs workspace rows to find
`agag-status.json`, and will address it where the requirement is concrete.

## Evidence

- The `agforge` token created a real issue in the FreeForge project and the
  response's `created_by` equalled `agforge`'s own user id, not the shared
  admin's — the plan's end-to-end attribution acceptance criterion, met. The
  probe issue was deleted (HTTP 204).
- `cagent`, `autolab`, and `devworld-assistant` tokens each answered the
  project `states/` route with HTTP 200.
- All four env files are mode 0600 and confirmed git-ignored
  (`git check-ignore`).
- `nctl desired apply` committed 5 creates, 0 conflicts; a re-export
  (`nctl desired export` → `apply -f`) previews 86 unchanged, 0 changes.

## Not done in this step

`AUTOLAB_NODE_PLANE_CREDENTIALS_SOURCE` still points at the shared
`.local/plane-credentials.env` on the deployed nodes. Repointing it at
`.local/plane/autolab.env` requires an Ansible run against the real
`agautolab1` node, which is an explicit approval boundary in this repository's
environment classes, not an ordinary local action. It is queued for the
enrollment step and will be raised with the developer then.

## Stale doc corrected

`pj-agdev/.local/devenv.md` still described `ProjectA` (`PA`) as the managed
project. It no longer exists in the running Plane instance; the standing
projects today are `FreeForge`, `ClusterAdmin`, `Assetpipe1`, and per-episode
ones. Corrected there, along with the new per-agent credential layout.

Created four Plane accounts that in-system agents could arguably have created
for themselves — handoff candidate.
