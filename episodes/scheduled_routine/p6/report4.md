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

This makes the remaining resume verification impossible through the required
Front → Autolab path. The run is stopped here; Steps 5 and 6 have not been
started. No cluster Desired State, container, or persistent process was
created by the experiment.
