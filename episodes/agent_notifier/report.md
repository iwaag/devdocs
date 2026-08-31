# agent_notifier — Final report

The deterministic ComfyUI-to-Zulip notifier is installed and running. Agents
call `comfynotify watch <prompt_id>` once, end their run, and the dedicated
Comfy Notifier bot posts a durable terminal record back to the chosen topic.

Measured results: a 640×640 SDXL still completed and was posted after 18 s;
a real 640×640, 124-frame MiniMax clip completed and was posted after 350 s.
Tickets survive daemon restarts and archive only after a successful post.

The short-job two-serving experiment exposed a race: a five-second job can
finish before the initial agent run exits, allowing that run's eventual
completion post to supersede the notifier callback. This is recorded in
`report4.md`; the long clip establishes the intended no-paid-wait interval.
