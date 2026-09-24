# Clearer Chat UI ex1 — Preserve Requests and Respect Non-Answers

## Goal and scope

Fix the two review findings: a task's required confirmation must survive a model's report intent, and selecting “not an answer” must prevent the next post from implicitly settling a request.

This remains a breaking-change experiment. Backward compatibility is unnecessary; choose the simplest coherent contract and update affected consumers together. Exact field names and implementation structure are left to the implementer. Preserve the distinction between receiving an answer and accepting work.

Paths below are relative to the workspace containing the projects.

## step1 — Preserve the handler's confirmation requirement

**Finding:** `pj-agdev/agautolab/src/agautolab/zulip_listener.py` returns a `TopicResult` with `response_request` / `confirmation` when the worker wrote a report but its requester has not agreed. Its wrapper also supplies the model's output. In `pyagag/src/agag/topics.py`, `split.meta or meta` lets a model's `report` replace that confirmation requirement. The delivered text says agreement is needed, but no request appears in the wait list.

- Define how handler metadata and model metadata combine for a single delivered post. A handler's required confirmation must survive a weaker model intent such as `report` or `progress`.
- Reuse or extend the existing intent merge helpers where appropriate. Specify recipient, question/confirmation, and reference precedence explicitly; a strongest-intent comparison alone does not resolve competing recipients or request kinds.
- Keep the handler's actual confirmation recipient and processed-input boundary intact. Ordinary model questions and reports without a handler requirement should retain their declared intent.
- Check repair and failure paths: repairing the model's text must not erase a still-applicable confirmation requirement. If the handler never reached that state, a failure must not invent one.

**Verification:** exercise the real `serve_run` → shared posting path with a worker returning `ag-reply intent=report` while the task awaits agreement. Assert that the delivered post carries a confirmation request to the requester and appears as pending in `read_requests`. Also cover model progress, a model question, conflicting metadata, and reply repair. Confirm that this change does not itself accept or integrate a task.

## step2 — Carry an explicit non-answer through posting and readback

**Finding:** `pj-agdev/agdevworld/src/scenes/FrontDeskScene.ts` handles “not an answer” by clearing `answering`. Submission then carries no reference. `pyagag/src/agag/outstanding.py` treats an unreferenced recipient post as an answer when exactly one request is open, so the supposedly unrelated post clears the wait.

- Represent three distinct composer choices: automatic correlation, an explicit answer to a request, and an explicit non-answer. Do not overload `null` to mean both automatic and non-answer.
- Add a small durable correlation field to the shared post contract. Choose its name and define its interaction with `re=`; conflicting combinations should have an explicit validation outcome.
- Pass the non-answer choice through the shared Front Desk / Arguing Room client, relay, and persisted Zulip post. `read_requests` should skip implicit settlement for that post.
- Preserve the existing convenience rule for ordinary posts: with one pending request and no explicit choice, next-post correlation still works. Explicit references continue to settle only the named requests.
- Make the selected non-answer mode visible before submission, allow returning to automatic or explicit-answer mode, and reset it after a successful send or conversation change. A failed/uncertain send should retain the draft's correlation choice for a consistent retry.
- Teach agent-facing readers the new field's meaning and update CLI support if needed for the shared contract. Reuse the existing prepared-message delivery path so retries preserve the choice.

**Verification:** with one pending question, select it, choose “not an answer,” and submit an unrelated update. Assert that the recorded post carries the choice and the question remains pending after reload/reconstruction. Then answer explicitly and confirm it clears. Cover automatic next-post answers, two pending questions, mode changes, and failed-send retry. Verify both dialogue rooms in the browser.

## step3 — Validate, deploy, and report

- Run focused pyagag post/reply/outstanding/lifecycle tests, autolab task-serving tests, relay posting tests, and the frontend build. Add regression cases for the two failures rather than only testing the new helpers in isolation.
- Validate the UI against existing fixture/demo facilities, then use a small live trial to check stored metadata and visible wait transitions. For autolab, a controlled task fixture can prove the confirmation path without starting unrelated work.
- Before inspecting or updating running services, consult Nautobot or `nctl` as described in `pj-clusterintent/nctl/README.md`. Update dependency pins for consumers of the changed shared contract and refresh affected services and the frontend using the local environment notes.
- Commit and push the changes and necessary submodule pointers. Update the post contract documentation and write `ex1/report.md` with regression evidence, deployed scope, and any remaining limitations. Update the parent report if its contract description changes; keep machine-specific details in ignored local notes.

**Completion:** required task confirmations remain visible even when the worker reports success, and a deliberately unrelated human post leaves the outstanding question open. Both behaviors survive posting retries and history reconstruction.
