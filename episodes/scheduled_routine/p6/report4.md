# Step 4 — upper-actor handoff and resume attempt

The first test reached the intended persisted `waiting_external` boundary.
Its local-only handoff requests a supported provider API key or provider
selection for Prime Agent. It predicts no container, service, GPU allocation,
or port; the only temporary runtime impact is a short-lived CLI process and a
disposable scratch directory, both already cleaned up. Read-only readiness is
the provider environment variable being available to the one-shot command.

The Omni Actor rejected the operation for this phase: no provider key or
provider selection was supplied, and an existing scoped credential must not be
repurposed. The state remains `waiting_external`; no install or one-shot retry
is authorised.

Front registered and the production dispatcher fired a second one-shot event,
`r11` / `e41`, at `2026-08-28T06:36:43Z`. It did not create a duplicate
repository. However, recording the rejection exposed a live workflow defect:
Autolab's post-completion re-plan re-used a resolved workrun topic without
binding it to the new task. The topic correctly replied that it was unbound,
so no repository mutation occurred.

Front then hit its runtime session limit while trying to obtain the fresh
bound task required to finish the record:

```
front run exited 1: You've hit your session limit
```

The live evidence led to an Autolab repair: completed-task rework now opens a
fresh, anchored `workrun-rerun-task…` surface rather than posting to the
resolved topic.  The change is in agautolab commit `cb8556b` (parent project
pin `4468f54`), with 169 focused tests passing.  It was pushed before this
resume test.

## Resumed result

After the Claude limit reset, the upper actor selected the already-running
local Ollama service instead of a cloud credential.  Prime Agent v0.8.1 was
given its documented custom provider configuration in ignored local material:
`http://localhost:11434/v1`, `openai-completions`, and model
`qwen3.8:27b-mlx-bf16`.  The nominal `apiKey` field is required by Prime
Agent's OpenAI-compatible provider schema, but Ollama did not receive or
require a real credential.

The first retry exposed a macOS Unix-socket pathname limit because the
workspace-derived socket was too long.  A fresh short socket under `/tmp` was
then used.  At `2026-08-28T10:19Z`, the bounded offline/no-tools command
returned exit `0`; its JSON event stream names provider `ollama`, API
`openai-completions`, model `qwen3.8:27b-mlx-bf16`, and final response
`OLLAMA_PRIME_OK`.  This is the missing model-response evidence, so the
persisted local-test state changed from `waiting_external` to `verified`.

No container, persistent process, model download, port, or Nautobot Desired
State was created or altered.  `nctl status` remained healthy (Nautobot
reachable, one worker, zero pending jobs) immediately before the retry.

The localtest result commit is `3f222b5`.  Its required push to the local
Gitea repository could not complete because Git was unable to obtain a
username for `http://agstudio.local:3000` (`failed to get: -25308` / `could
not read Username`).  No credential was guessed or substituted.  The
repository is locally committed but not yet remotely pushed; this is the
remaining external handoff before Steps 5–6 can truthfully claim the Step 3
repository-push verification.
