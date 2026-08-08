# autodev — Step 5 report: dev node setup (agautolab1 VM)

Status: **complete** (VM created, manual handoff done, runtime provisioned and
verified end to end including a real headless claude run on the dedicated
account).

## VM creation and manual handoff

Per the plan's 2026-08-07 decision the runner is a fresh VM with a manual
boundary. The QEMU creation path was implemented in pj-clusterintent first
(`devdocs/vision/vm/report.md` there), planned as `devdocs/vision/vm/ex1/plan.md`,
and the VM was created and hand-configured by the user: `agautolab1` on
aghub-pve (Ubuntu 24.04.4, 4 vCPU / 8 GB RAM / 31 GB root). The user did the
initial SSH setup (`eiji@agautolab1.local`, `~/.ssh/ansible_key`) and the
Claude login with the dedicated account, per the handoff boundary.

## Automated provisioning (this step's agent work)

- `uv` 0.12.2 installed userspace; no root was needed for anything
  (`loginctl enable-linger` succeeded without sudo; Linger=yes).
- agautolab pushed to the agstudio gitea as `autodev/agautolab` and cloned to
  `~/agautolab` on the VM over HTTP. Gitea token stored at
  `~/.agautolab/.local/gitea/autolab-agent.token` (mode 600) and wired via
  `git credential.helper store`, so clones/pushes need no token-in-URL.
- Test suite on the VM: `uv run pytest -q` → **23 passed**.
- systemd user unit `autolab@.service` installed to `~/.config/systemd/user/`
  (template from `devenv/systemd/`), `daemon-reload` done, instance not yet
  enabled — starting a job loop is Step 6.
- Claude Code CLI 2.1.224: the official installer's `claude install`
  self-setup step **hung reproducibly** (100% CPU busy-loop, 10+ min, twice)
  on this VM. Workaround: the downloaded binary
  (`~/.claude/downloads/claude-2.1.224-linux-x64`) IS the CLI; the user
  placed it directly as `~/.local/bin/claude`. Version check OK.

## Verification

`claude -p` one-shot on the VM (headless, dedicated account):
`"Reply with exactly: OK"` → `OK`, cost $0.057, models haiku-4.5 + sonnet-5
in the usage map. This validates the full Step 6 execution chain: SSH →
claude headless with JSON output → gitea reachable from the VM (API 200,
clone/push verified during setup).

## Notes / follow-ups

- **node/npm are not installed** on the VM. If Step 6's Othello job uses npm
  gates, install Node first (userspace via nvm is fine) or pick gates that
  run under uv/python + playwright.
- The claude-installer hang is worth remembering for future nodes: skip
  `claude install` and place the downloaded binary manually. (Candidate ENT
  item if it recurs on the next node.)
- Auto-updates: the manually-placed binary won't self-update cleanly; pin and
  update by re-download when needed.
- sudo on the VM requires a password, so anything needing root stays a manual
  user step by design.

Also reported in `pj-agdev/devdocs/episodes/agautolab/begin/report.md`.
clusterintent-side work (QEMU creation path) is reported in its own repo
(`devdocs/vision/vm/`).
