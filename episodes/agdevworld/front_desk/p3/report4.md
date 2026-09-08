# Step 4 — integration, live verification, deployment

agdevworld `a87fdf9`; pyagag `0e38c25`; agautolab `111ed7f`. One live
completion was carried out, on the developer's explicit approval.

## Tests and build

| repository | result |
|---|---|
| `agdevworld/agentroom` | `uv run pytest -q` → **216 passed** (`test_closing.py` 23, `test_close.py` 25 new in p3) |
| `pyagag` | **470 passed** (5 new for the shared completion rule) |
| `agautolab` | **219 passed** on the upgraded pyagag pin — `test_intro.py::test_the_intro_is_posted_for_this_instance`, which failed at the p1 pin with `'Client' object has no attribute 'whoami'`, passes again |
| `agdevworld` | `npm run build` and `tsc --noEmit` clean |

## Resident state before deploying

`nctl status` (pj-clusterintent): Nautobot 3.1.3 authenticated, 1 celery
worker, 0 pending jobs, 8 nodeutils dumps, every submodule clean.
`nctl drift --host agstudio` → **converged**, 17 placements `applied`,
`agdevworld-web` and `agfront` among them.

## Deployment

- `devenv/launchd/com.agdev.agentroom.plist.in` gained
  **`AGENTROOM_PLANE_ENV`**; regenerated into `.local/launchd/`, installed,
  and the job **booted out and bootstrapped** — `kickstart -k` does not
  re-read `EnvironmentVariables`. The relay's startup line now ends with
  `completion with Plane`, and `GET /` lists `/frontdesk/<id>/close-plan`
  and `POST /frontdesk/<id>/close`.
- The credential is the **admin** Plane key. The per-agent `autolab` key is
  HTTP 403 on Ghtrends' state list, so an agent identity cannot drive this
  button today; that is recorded in the launchd README and in `.local/`.
- `docker compose up --build -d web` rebuilt `:8090` with the p3 frontend.

## The live run

`http://localhost:8090/?view=frontdesk&conv=20260908-1600`, `finish ✔`.

Preview: `4 to change · 2 already · 0 blocked · 0 kept`, no gaps, no
exclusions, 10 Zulip calls. Its fingerprint (`2d30f5a4…`) matched the one
the module produced offline against the same realm, so the deployed relay
and the local run agreed.

Result — **closed — 4 changed, 2 already were**:

| target | outcome |
|---|---|
| Plane **G-13** "Cover a new GitHub-trending repository (routine run)" | `applied` — is Done |
| Plane G-14 | `already` |
| `#pj-ghtrends › workplan-trend6` | `applied` — is ✔ |
| `#work-g-13 › workrun-task1-g-13` | `already` |
| `#work-g-13` (channel) | `applied` — is archived |
| `#front › front-desk-20260908-1600` | `applied` — is ✔ |

Read back **independently of the relay**, straight from Zulip and Plane:

```
work-g-13 present in channel list: False
#front topics:        ['✔ front-desk-20260908-1600']
pj-ghtrends topics:   ['✔ workplan-trend6']
G-13 -> completed     G-14 -> completed
```

G-13 is exactly the braindump's complaint — a mission Work left `started`
with every Sub-Work completed — and closing the conversation closed it.

**Second attempt and page reload**: a fresh browser load, `finish ✔` again
→ `0 to change · 6 already · 0 blocked · 0 kept`, and no close button at
all, since there is nothing ready. Nothing was written twice.

## One defect the live run found, and fixed

Archiving a channel is what makes it unlistable, so on the second preview
`get_stream_id("work-g-13")` answered `HTTP 400 Invalid channel name` and
the row read *"kept — the channel's topics could not be read"*: this
operation's own finished work reported as a gap, which is the exact failure
this payload exists to prevent. `Reader.channel_exists` now asks the realm's
channel list — one `GET streams`, and **only** after a topic list has
already failed — and the row reads *"already archived: the realm no longer
lists this channel"*, `done` rather than `kept`; the failed lookup is
dropped from `gaps.errors`, since it was the question that answered. Two
tests pin the pair apart (a listed channel that will not answer is still a
read failure) and one pins the retry. Relay kickstarted; the live payload
now carries `errors: []`.

## Not exercised live

**Reopening a closed conversation.** `POST /frontdesk/<id>/post` un-resolves
only the Front topic; a resolved child topic and an archived channel are
untouched by it, and the screen says *"↩ reopened this conversation — the
work it closed stays closed"*. Any post there buys a paid Front run, so this
was checked in the code and in the panel rather than by posting — stated
here rather than implied as done.

## Screenshots

`agdevworld/.local/shots/p3/` (ignored): `live-b-preview` (before),
`live-c-closed` (after), `live-d-reload` (the second preview, before the
fix), `live-e-already` (after it).
