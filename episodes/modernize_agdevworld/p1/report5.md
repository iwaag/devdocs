# p1 step 5 — strip agdevworld

Done, and **fully deleted** — the plan allowed partial removal at the
implementer's discretion, and nothing here was worth keeping. `agdevworld`
now hosts no agent and no backend at all.

## What went

| Removed | Note |
|---|---|
| `assistant/` | the whole directory: the Python package, its tests, `Dockerfile`, `GUIDE.md`, `pyproject.toml`, `uv.lock` — 22 tracked files |
| `compose.yaml` `assistant` service | with it, all fifteen of its environment values, both secret mounts, the `extra_hosts` pin and the `assistant_records` volume declaration |
| `nginx.conf` `location /api/` | the proxy to `assistant:8091`, and its 310 s read timeout |
| `vite.config.ts` | the file held nothing but the dev `/api` proxy |
| `agents.toml` | agdevworld defines no role now, so a role/profile contract here would be a lie |
| `.env` | every key in it (`AUTOLAB_NODES`, `PLANE_*`, `ZULIP_LAN_HOST`) existed only for the assistant container; the file is now empty of purpose |
| `.gitignore` "Python (assistant)" block, `.dockerignore` `**/.venv` etc. | no Python in this repository any more |

`.env` was git-ignored and held real hostnames, so it was copied to the
(also ignored) `agdevworld/.local/env.before-p1-strip` before being removed —
deleting a developer's local values outright would have been the one
irreversible act in this step. `.local/` is now explicitly ignored.

The `assistant_records` Docker **volume was not deleted**. The service that
wrote it is gone, but the records it holds are evidence of past runs, and
`docker volume rm` is not something to do on the way past. It is unreferenced
and can be removed by hand whenever the developer wants it gone.

`docker compose up --build -d --remove-orphans web` stopped and removed the
running `agdevworld-assistant-1` container, which had been up for 46 hours.

## What stayed, deliberately

- **The chat panel** (`src/chatPanel.ts`) renders exactly as before. It posts
  to `/api/chat`, which no longer exists — the send path fails. The plan
  called removal work on it wasted, since it becomes a thin `#front` wrapper
  in a later phase, and that is still true.
- **The views' `/api/*` reads** (`autolabState.ts`, `planeState.ts`, and the
  `workspaces` view) fall back to their sample JSON. The `nodes` view is
  unaffected: it reads the cagent snapshot out of `public/`, which never went
  through the assistant.
- **`src/detailPopup.ts`'s profile note** still says "Profile changes go
  through the assistant conversation". It names a conversation that no longer
  exists here; it moves with the chat panel in the same later phase. Recorded
  rather than silently patched.

One seam does answer where a route used to be: nginx's SPA catch-all
(`try_files … /index.html`) returns `200` with the HTML page for `/api/chat`.
That is the single-page fallback doing its job, not a surviving backend route
— the request returns markup, and the chat panel's JSON parse fails on it.

## Documentation

- `agdevworld/README_DEV.md` rewritten: the assistant sections, the
  `uv run pytest` test entrance, the harness table, the runtime-variable
  paragraph and the safety-device list are gone. A new "No backend this
  phase" section states plainly what is dead and why the chat panel is still
  drawn. The Plane-dispatch and autolab-profile sections were kept as
  descriptions of what the views implement, each marked as having no backend.
- `pj-agdev/.local/devenv.md` gained a note at the top saying agdevworld is a
  pure frontend since today, and that the assistant material further down is
  history kept for what it explains (the Zulip-from-container DNS lesson, the
  autolab passthrough), not a running service.

## Verification

| Check | Result |
|---|---|
| `npm run build` | `tsc && vite build` — built in 332 ms, only the pre-existing Phaser bundle-size advisory |
| `docker compose up --build -d --remove-orphans web` | image rebuilt, `agdevworld-assistant-1` removed, `agdevworld-web-1` up |
| `docker compose ps` | one service: `web`, `0.0.0.0:8090->80` |
| `curl -I http://localhost:8090/` | `HTTP/1.1 200 OK` |

## Known loss, deliberately deferred

The assistant's project-start route (`projects.py`: Gitea repo pair → Plane →
Zulip) duplicated `agautolab/init_project.py`. After this step **project
creation is agautolab-side only**. Nothing in agdevworld can start a project,
and no phase-1 work restores it.

Two other things are now unused rather than removed, both out of this phase's
scope: the **Devworld Assistant Zulip bot** (user 10, credential file
`.local/zulip/devworld-assistant.env`) and its **Plane agent account**
(`.local/plane/devworld-assistant.env`). No process holds either. They are
identities in external systems, and retiring an identity is its own decision.
