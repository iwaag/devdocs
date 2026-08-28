# Step 5 — experimental service disposition

## Inventory and disposition

The verified Prime Agent retry ran as one foreground CLI process.  It started
no Docker Compose project or container, no persistent process, no listener,
and no model download.  Its short `/tmp` daemon socket was transient.  The
only runtime it used was the pre-existing local Ollama endpoint.

Disposition: **retained managed service, already managed**.  This episode did
not create Ollama and therefore made no proposed or applied Desired State
change.  The scoped `nctl drift --host agstudio --service ollama --json`
filter returned no service target (that filter does not expand placements),
so the node-scoped verification was used instead.  At 2026-08-28T10:29Z,
`nctl drift --host agstudio --json` reported `converged: 1`, zero warnings or
errors, and records desired service `ollama`, placement
`ollama-agstudio`, deployment profile `ollama`, endpoint `ollama-api`, and
production effect `applied`.

## cagent and discovery evidence

A read-only cagent window request, `req_c6caddd0412b4f4fbb792ee64a941f52`,
reported the experiment and asked whether the existing service was registered
and observable.  It completed without a repair or Desired State mutation:
the service is converged, placement `ollama-agstudio` is active with profile
`ollama`, and nodeutils recorded the active `ollama` process and HTTP 200
endpoint on agstudio.  The agent also clarified the scope: this is true
because the test's `localhost:11434` is the agstudio process; a separate
Ollama on another host would not be covered by this observation.

No new discovery code is justified.  `nodeutils` already documents sanitized
container, Compose-project, port, and important-service collection, and its
existing tests cover an Ollama macOS launchd/process probe and HTTP endpoint
observation.  The live cluster state has the matching managed placement;
there is no unregistered localtest Compose project or retained experimental
runtime to compare.  No exact nctl batch was prepared or applied.
