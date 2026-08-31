# agent_notifier — Step 3 report

Completed 2026-08-31.

- Added `comfynotify watch` with agent-oriented help. It writes one atomic
  ticket, defaults the destination to `AGENTCHAT_HOME`, defaults the backend
  URL to `AGFORGE_COMFYUI_URL`, and returns without polling.
- Handed the executable to autolab and agforge role environments through
  PATH, with focused handover tests passing for both projects.
- Submitted a 640×640 six-node SDXL still, ran `watch` in a shell that
  mimicked a run home topic, and returned immediately. The ticket stayed open
  while ComfyUI ran; the daemon posted one successful callback 18 seconds
  after submission with the output reference and VRAM reading, then archived
  the ticket.

This also completes the running-ticket check deferred from Step 2.
