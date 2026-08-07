# autodev — Step 4 report: fresh agent-only Gitea on agstudio

Status: **complete** (old gitea removed, fresh instance deployed and verified
end to end via API + git push/clone; compose and setup notes recorded in
`agautolab/devenv/gitea/`).

## Removal of the old instance

The old experimental gitea was not a standalone deploy: it was the `localgit`
service inside `~/services/service_scripts/docker-compose.yml` (container
`gitea`, ports 3000/2222, bind mount `~/services/gitea_data`, ~6 MB with 5
old repos: testapp, testgame, bnklook, autolab-front, agcore).

- A throwaway tarball of the data was taken into the session scratchpad
  before removal (pure insurance; auto-cleaned with the session, nothing
  kept per plan's "no value" ruling).
- The permission classifier blocked `docker stop/rm` from the agent, so the
  **user removed the container and `gitea_data` by hand**; I then removed the
  `localgit` service block and the now-unused `gitea` network from
  `service_scripts/docker-compose.yml` (validated with `docker compose
  config -q`) so a later `up -d` cannot resurrect it.

## Fresh deploy

`agautolab/devenv/gitea/compose.yaml` — `gitea/gitea:latest` (1.27.1) as
container `autodev-gitea`, named volume `autodev_gitea_data`, ports 3000
(HTTP) / 2222 (SSH), sqlite3 + `INSTALL_LOCK=true` (no web installer),
`DISABLE_REGISTRATION=true` (agent-only), push-create enabled for org/user.
Setup commands and conventions in `devenv/gitea/SETUP.md`.

Account setup per plan: admin user `autolab-agent` (random password) created
via the container CLI; API token (`--scopes all`) generated and stored with
the password in `agautolab/.local/gitea/` (mode 600, `.local/` gitignored —
token value never entered a tracked file or the transcript); org `autodev`
created via API (visibility private).

## Verification

- `GET /api/v1/version` → 1.27.1 on both `localhost:3000` and
  `agstudio.local:3000`.
- API repo create `autodev/step4-smoke` (201) → git push over HTTP with the
  token → fresh clone round-trips the pushed README → smoke repo deleted
  (204), scratch clones (which embed the token in their remote URL) removed.

## Notes / follow-ups

- Old-gitea removal happened on the "agstudio = this Mac" premise from the
  local env memo; the `service_scripts` compose edit is outside any git repo,
  so it is recorded here rather than in a commit.
- Whether clusterintent should carry a desired-state entry for "gitea runs on
  agstudio" stays a Step 7 question; no clusterintent work was needed now.

Also reported in `pj-agdev/devdocs/episodes/agautolab/begin/report.md`.
