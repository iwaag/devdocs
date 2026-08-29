# sage p1 — step 2 report: listener and serving

Implemented and pushed the custom entrance listener in arxivsage.

## Implementation

- `SPEC` uses only `plan_prefix="entrance-"`; the generated run prefix was
  removed because sage has no run-topic vocabulary.
- `entrance-` topics use `serve_topic` with the shared acknowledgement and a
  streaming `front` run. The topic workspace receives `chatlog.md` and
  `transcript.jsonl`; records are written beneath
  `.local/agent/entrance_front/`.
- Each run is told its knowledge and study-queue placements at runtime and is
  given the current short Git revision. The same revision is stored as
  `knowledge_revision` in its run record.
- Other topics in the sage's own channel are served without a model run and
  receive: "Please ask in a topic named `entrance-…`."
- A missing knowledge clone yields the explicit revision `unavailable`; this
  is intentional until the deployment step creates the clone.

## Verification and versions

- `uv run python -m compileall -q src` completed successfully.
- An import-level check confirmed the empty run prefix and both placement
  lines in the generated prompt.
- Pushed arxivsage commits: `fe00ae0` (listener) and `a6a704b` (the generated
  `uv.lock`, pinning the GitHub pyagag dependency).

No model run was made in this implementation step; acceptance-run costs are
recorded in step 6.
