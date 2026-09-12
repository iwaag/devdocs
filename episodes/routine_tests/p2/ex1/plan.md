# routine_tests p2 ex1 — repair conversation input and anchor lifecycle

Implement [problem.md](problem.md) in order **B1 → C → A → B2**. The result
should be a routine whose delegates answer its run, whose supervision survives
plan replacement, and whose first serving receives the request directly.

This is experimental development: breaking internal APIs, replacing helpers,
and recreating test conversations are reasonable choices. No compatibility
layer or general security hardening is required. The steps below specify
observable outcomes; helper signatures, prompt limits, and test organization
are the implementer's choice.

## Implementation map and useful findings

Paths below are relative to the projects workspace.

- `pyagag/` is a separate repository beside `pj-agdev/`, not inside it.
  Shared owners are `src/agag/selfnote.py`, `zulip.py`, `topics.py`, and
  `chat.py`. Consumers install it through Git-backed dependency locks.
- Front's active entry point is `agfront.listener`; workspace construction
  and callback handling still live in
  `pj-agdev/agfront/src/agfront/zulip_listener.py` (`serve`, `front_prompt`,
  `handle_mention`). All three roles use this prompt path: `front`,
  `character_talk`, and `routine_run`.
- Front already renders the serving's history before calling the model.
  Desk/run roles use `format_evidence`; ordinary Front uses `format_chatlog`.
  Reuse those rendered bytes, including their existing filtering and evidence
  metadata, instead of fetching or formatting a second conversation.
- `ZulipClient.message(id)` already returns a message's current channel/topic
  after moves and renames, or `None` when unavailable. Autolab's
  `worklog._conversation_of` illustrates its use. `retire_conversation`
  releases the old name before replacement, and `anchor.py` already writes
  the `replaces` relation. No new retirement protocol is needed.
- Routing has several readers: `rootchat_home`, `rootchat_notes`,
  `remotes_for_home`, `sweep_rootchats`, and the mention recovery route.
  `rootchat_notes` currently selects the earliest note independently of
  `own_rootchat`; fixing only one reader would leave inconsistent routing.
- Inspect services through Nautobot/nctl before deployment, using
  `pj-clusterintent/nctl/README.md` and the two ignored environment memos.
  Planning used `nctl status --json` and `nctl drift --json`; these describe
  recorded infrastructure state, not proof of the running Python revision.
  Keep environment-specific evidence in `.local/`.

## 1. B1 — let the new run delegate

Update `pj-agdev/agfront/agent/guides/front/guide.md` beside the run-opening
instruction: opening the run completes this serving's work for that routine;
report its location and finish the reply. The subsequently served run makes
its own delegations, so their anchors name the run. Explain that causal link
in the guide. Keep the existing opening-post contents and continuation flow.

Re-measure through the ordinary entrance with a small routine that actually
delegates. Record how many opening servings delegate on behalf of the new
run, and inspect the resulting anchor and callback destination. Success is
zero such opening-serving delegations and a callback serving `routine_run`.
Use the p2 publish trace as the demonstrated failure baseline; a text-match
test on the guide is not behavioral proof. Do not add B3's automatic rewrite
guard unless the changed guidance fails in a measured run.

## 2. C — put the conversation in the model input

Add a shared prompt-context helper in `agag.topics` and wire Front's
`serve`/`front_prompt` to supply the rendered chatlog as conversation content.
Include the full chatlog when small. For larger histories, include a bounded
recent portion containing the newest real message and sender, state what was
omitted, and retain `chatlog.md` and `threads/` for further reading. Define
and test the oversized-single-message behavior explicitly; truncation must
be visible. Keep conversation text distinguishable from the role's guide.

Use the same serving snapshot for the prompt and file. Preserve the current
removal of selfnotes, system notices, and acknowledgments. A callback's home
stays its conversation; its remote remains available as a thread.

Capture the prompt at the harness boundary with a fake harness that performs
no file reads. Assert that a new conversation's request and sender reach it;
the old implementation must fail that assertion. Cover the three Front roles,
large-history truncation, and a genuinely empty conversation. This proves
input delivery, not that a model can never misinterpret supplied text.

Check other users of `chatlog_placement`: shared `agag.entrance` and forge's
`assetplan_topic` have the same file-only pattern. Reuse the helper where
straightforward; document any remaining callers without making a broad
prompt redesign a prerequisite for Front's repair.

## 3. A — recover an anchor through one replacement relation

In the shared lookup, first resolve this agent's own anchor in the current
topic. Only if absent, read a valid `replaces` message id, fetch that
message's current conversation, and inspect that conversation for this
agent's own anchor. Follow one replacement hop only.

The relation is written by the replacing agent, often autolab, so filtering
it to Front's sender id would reproduce the bug. The recovered root note
must still belong to the requesting agent. Keep `own_rootchat` a pure
history selector and put network resolution in `zulip.py` or an equivalent
shared helper. Missing/malformed pointers or targets produce no anchor;
do not guess from the reused display name. Current-topic anchors take
precedence over inherited ones.

Exercise retirement and replacement with two agent identities: an original
workplan contains the supervisor's root note; retirement moves it; the
replacement reuses the name and names the retired mission by id. A mention
from the replacement must serve the original supervising run and make the
replacement's callback text readable in that serving. The old code must
ignore that same mention. Also check a moved target, deleted target, foreign
root note, and a second replacement hop that is deliberately not followed.

Check both event delivery and startup/recovery. A replacement has no own
root-note search hit, so ensure the mention route and callback thread
construction can discover it; adjust shared discovery as needed. Do not
copy another agent's notes to make this work. A mention from an unanchored
task with no replacement relation is still outside this repair.

## 4. B2 — provide deliberate anchor correction

Add `[selfnote][rootchat-moved] <channel>/<topic>` to the shared convention.
Use one effective-anchor rule everywhere: the newest valid explicit move
written by this agent wins; otherwise its earliest ordinary root note wins.
A later ordinary repeat never redirects the topic. Apply this rule within
both the current topic and A's inherited topic.

Expose a small explicit `agentchat` repair command (name and arguments are
open) that writes the move as the authenticated agent. Explain its use in
the tool documentation; retain ordinary send's automatic anchoring behavior.
Align note search, recovery, and `remotes_for_home` with the effective anchor,
so corrected delegates appear under the new home rather than the old one.
The move remains a selfnote: hidden from chatlogs and insufficient by itself
to trigger a serving.

Test Desk anchor → ordinary run anchor repeat → explicit run correction →
another ordinary repeat. Only the explicit correction changes the effective
home. Check foreign/malformed moves, a second explicit correction, thread
ownership, replacement plus correction, and restart recovery without replaying
already-served callbacks. Preserve resolved-home handling: a late answer
must not reopen a finished run or create a bare-name twin.

## Validation, rollout, and completion

Extend existing fixtures rather than building a new test framework:
`pyagag/tests/test_{selfnote,zulip,topics,chat}.py` and
`pj-agdev/agfront/tests/test_{zulip_listener,routine_run,role_run}.py`.
Use autolab's replacement fixtures where helpful. Demonstrate that each new
code regression check fails against the corresponding old behavior and
passes with its fix; record the failure, not just the final test count.
Run `uv run pytest -q` in each changed Python package and the owning
project's additional required checks. Test consumers against the intended
pyagag revision, not an accidentally old installed dependency.

Commit/push shared code first, update affected consumer locks with
`uv lock --upgrade-package pyagag`, sync, validate, and commit/push consumers
and changed superproject pointers. Inventory pyagag consumers across both
projects when rolling out shared routing changes. Use the existing deployment
routes from the local memos; verify the installed dependency commit and
import path in each relevant service environment, then restart code-holding
listeners before retesting. Guide files are read per serving.

Finish with a short live sequence: a fresh request, a routine's own
delegation and completion, a replacement callback, and a deliberately
mis-anchored test delegation repaired through the new command. Several
fresh conversations should be sampled for C's previously intermittent
symptom. Verify the complete run lifecycle: callback evidence, entries,
one terminal block, report delivery, and resolution. Record all assistance
separately from autonomous success; a successful unit test or restart alone
does not prove the workflow.

Update `pyagag/README.md` and `devdocs/README_DEV.md` with prompt delivery,
replacement lookup, and explicit correction semantics. State plainly that
retiring a plan renames its conversation and moves every agent's notes.
Write this episode's `report.md` with before/after evidence, B1 and C sample
counts, tested/deployed revisions, validation results, and remaining limits.
Leave research methods and unrelated routine policies to their existing
owners. Commit and push the final documentation.
