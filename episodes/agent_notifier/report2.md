# agent_notifier — Step 2 report

Completed 2026-08-31.

- Added the `comfynotify` disk-backed daemon and the
  `com.agdev.comfy-notifier` launchd template. It polls ComfyUI history and
  queue state, emits one bot-authored Zulip post per terminal ticket, then
  archives the ticket. Callback records include bounded error text, output
  references and view URLs when available.
- Installed and started the local launchd job. Its environment uses the
  dedicated notifier credential and the existing `agentchat` executable.
- Hand-written verification tickets posted exactly once for a completed
  history entry (`error`), an unknown prompt (`lost` after three polls), and
  an unreachable endpoint. A second sweep produced no duplicate posts.
- Six focused tests pass, including output extraction, pending/lost and
  unreachable state handling, error truncation, and restart-safe archival.

The running-job case is exercised by the end-to-end SDXL ticket in Step 3.
