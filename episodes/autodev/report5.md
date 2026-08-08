# Step 5 report — dev node setup (agautolab1)

Date: 2026-08-07. Outcome: **completed**. The job-runner VM is fully provisioned;
`autolab loop` can now run real jobs under systemd with a logged-in Claude Code CLI.

## What was set up

VM: `agautolab1.local` = Proxmox VM 109 on `aghub.local` (Ubuntu 24.04.4,
4 vCPU / 8 GB RAM / 31 GB disk). Per the plan's handoff boundary, the user did the
initial SSH setup and the Claude login; everything else was automated over SSH as `eiji`.

- SSH access with `~/.ssh/ansible_key` confirmed.
- `uv` 0.12.2 in user space (`~/.local/bin`).
- agautolab pushed to gitea (`autodev/agautolab`) and cloned to `~/agautolab` on the VM;
  `uv run pytest -q` → 23 passed.
- Gitea token in `~/.agautolab/.local/gitea/` (mode 600) + git credential store;
  push/clone from the VM verified.
- systemd user unit `autolab@.service` installed to `~/.config/systemd/user/`,
  `loginctl enable-linger` on (Linger=yes). Added
  `Environment=PATH=%h/.local/bin:%h/.local/node/bin:/usr/local/bin:/usr/bin:/bin`
  to the unit (systemd user units don't include `~/.local/bin` by default, and the
  claude adapter shells out to `claude`).
- Claude Code CLI 2.1.224 installed via npm (`@anthropic-ai/claude-code`) under a
  user-space Node 22 (`~/.local/node`), with `~/.local/bin/claude` symlinked to it
  and `~/.local/node/bin` on the login PATH.
- User logged in with the dedicated Claude account. Headless smoke test:
  `claude -p "Reply with exactly: OK" --output-format json` → `"result":"OK"`,
  7.3 s wall, model claude-sonnet-5, no permission denials.

## Incident: Claude CLI busy-loop (the bulk of this step's time)

The official installer hung twice at the self-setup stage with 100 % CPU. Root cause:
the VM was created with Proxmox's default CPU type `kvm64` ("Common KVM processor"),
which has **no AVX/AVX2**; the bun-compiled native `claude` binary busy-loops instead
of crashing on that CPU. Every avenue that ultimately executes the native binary
(installer, direct binary placement, and the npm 2.x package — which is only a wrapper
vendoring the same ELF) spun identically. Network was ruled out early (9.5 MB/s).

Fix (root on aghub, run by the user): `qm set 109 --cpu host` then a full
`qm shutdown` / `qm start`. After that the VM exposes the host CPU (Intel N350, AVX2)
and `claude --version` returns instantly. The npm install was kept.

Registered as WorkflowEpisode `701ad4e6-00c0-4cc0-b367-1e55d2548927`
("Claude Code CLI busy-loop on new Proxmox VM (kvm64 CPU without AVX2)", candidate,
painful). Main follow-up candidate recorded there: clusterintent's `create_qemu.yml`
sets no `--cpu`, so every future guest running modern toolchains will hit this again
unless `cpu: host` becomes the default.

## Ready for Step 6

Job repos can be cloned from / pushed to agstudio's gitea from the VM; the claude
adapter has a working, authenticated CLI on the service PATH; `autolab@<job>.service`
is installable per job. Next: create `autodev/othello-web` and start the first
full-auto run.
