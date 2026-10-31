# Concise workflow guides

Implemented in agautolab commit `8f158702ae7b3adee923ac4b5baa818f6a9294fa`.

## Result

The executor supplies the actual request and delivery route first, followed by a short common guide, existing scoped context and the WFE brief. Browser requests, conversation messages and delegated requests have different delivery routes. A delegated request keeps its requester route even when it carries the original conversation source.

The common guide is 2,633 characters. Six operational references are available through an absolute directory on each execution host; their hashes join the contract used to decide session reuse. Reference bodies are not automatically injected. Existing compact continuations and reconstruction remain in place.

Using curator/freework/run-001's historical input, context and initial brief, the assembled prompt shrank from 42,849 to 12,016 characters (28%). This was an offline assembly check, not a replay or a new agent answer. Current project instructions, environment blocks and the coordinator's catalog/brief are unchanged.

## Coverage

`coverage.md` maps the previous guide to its new locations. `previous-guide.md` preserves historical rationale. Conversation roles, provenance of instructions, correction obligations, Git branch saving and pinned challenge inputs were restored to the prototype's references during review. Detailed CLI syntax remains owned by tool help.

## Verification and deployment

- Full agautolab suite: `uv run pytest -q` — 51 passed, 1 skipped, 467.77 seconds. Earlier pre-adjustment runs failed two assertions about the old common-guide-first order; those expectations were updated and the final full run passed. The final delegated-route precedence change also passed all four focused routing/reference tests.
- Guide links resolve; `git diff --check` and Python compilation passed.
- Local executor restarted with no active attempts; launchd reports running and its startup log names the studio worker and conversation lane.
- GPU executor deployed through its existing `wfe_worker` profile with only executor_version changed. Ansible: ok=21, changed=2, unreachable=0, failed=0. Read-back confirms commit `8f158702ae7b3adee923ac4b5baa818f6a9294fa`, active service and six readable reference files.
- The executor commit is pushed. Deployment batches, prior desired state and logs remain in ignored local records. Both hosts now use the same implementation; their absolute reference directories are host-local facts.
- Rollback: deploy executor commit `eeddf9560a49d0bbc14ceebbc13bc7fe12fa8913` and restart between attempts. No WFE store migration is involved.

## Limits

No real-model trial was run and curator's historical records were not changed. Automated tests prove prompt delivery, routing, reference availability and existing execution behavior; they do not prove that a model will consistently choose to deliver a useful answer. Scoped-context filtering and automatic incident retrieval are outside this change.
