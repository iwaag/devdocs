# unshackle_agent / agforge turn1 — remove the forced paths

Goal: strip agforge of Tool Implantation. Every place where code forces the
in-system agent down a deterministic path is removed; what remains is the
*fact* that scripts and services exist, told once, in the fewest words that
still let the agent find them. Then run the same desires that worked before
and see what the agent still does on its own.

Source of the survey: [../../../../../shackle_list.md](../../../../../shackle_list.md)
(agforge section). Direction: [../../../../../memo.md](../../../../../memo.md).

This is an experiment, not a hardening pass. A capability that quietly
regresses is the finding we are here to collect — do not re-add a guard to
prevent it. Failures leave evidence; that is enough.

## What stays (the complete list — everything else goes)

These are not exempt because they are useful; they are exempt because they
either guard against irreversible harm, or they are pure observation that
takes no judgment from the agent.

1. **The outward HTTP contract shape.** agdevworld
   ([chatPanel.ts:192-198](../../../../../pj-agdev/agdevworld/src/chatPanel.ts#L192-L198))
   is a machine, not an agent: it requires `status` as a string, keeps
   polling on `working`, throws with `detail` on anything but `done`, and
   reads `artifacts[].url` where `kind === "image"`. The contract cannot be
   deleted — but its *author* moves from code to the agent (work item 1).

   It survives for one reason only: today's caller is an `if` statement. It
   is not a principle. `GET /api/requests/{id}` is an evidence read, not an
   entrance (devpolicy/terms.md), so a fixed shape there constrains no
   desire — and the day agdevworld's side can read prose, this contract
   dissolves too. Out of scope here; revisit in agdevworld's turn.
2. **The wall-clock kill** (`agent_run.py:56, 232`). Resource guard, not a
   wrongness guard: without it a hung opencode holds a worker thread and the
   caller polls forever. The number may be raised; the kill stays.
3. **Transcript capture** (`agent_run.py:212-216, 344`). Constrains nothing;
   it is the evidence this whole turn is measured with.
4. **`generate.py`'s `nctl-outbox` refusal** (`generate.py:138-139`) — but
   only as a property of the tool. The charter sentence commanding it is
   deleted; the agent may attempt it and the tool declines. Cross-boundary
   damage to a project agforge does not own.
5. **No `--dangerously-skip-permissions` / `opencode run --auto`.** Runs
   natively on the agstudio Mac (local policy). See work item 4 for how far
   the allowlist widens instead.

## Work items

### 1. Move authorship of the result from the parser to the agent

Delete `parse_outcome` (`agent_run.py:276-301`) and the charter's finish
contract (`charter.md:65-95`) as a *mandate*.

Replace with: the service tells the agent, as one line of fact, where the
caller will look —

    .local/jobs/<request_id>/result.json   # what the caller receives

— and serves whatever is there, **unvalidated**. No schema check, no key
whitelist, no coercion. The agent decides what `status` means, what goes in
`artifacts`, whether to include a `reply`. Merge only what is absent (the
service knows the request_id; it does not know the outcome).

Keep the `RESULT_URL:` / `RESULT_FAILED:` marker scan alive as a *lenient
alternative* — not the mandated path. Two natural ways to answer is less
forced than one; an agent that ends in prose and a written result file is
fine, so is one that only says the marker.

When the run ends and neither exists, the service reports the fact — the run
ended, no result reached the caller — carrying the agent's own final words
and the transcript path. Whatever `status` that ends up being, it describes
what reached the caller, never a verdict on the agent. Drop the punitive
framing entirely: nothing is "treated as a failed request even if the image
was generated perfectly".

The test for whether this item was done right: **judgment must leave
agforge's code, not merely move.** Say the agent writes `status: "done"`
with an empty `artifacts`. Today `parse_outcome` pre-empts that and rules it
a failure. Afterwards agforge passes it through and agdevworld says
`result contains no image artifact` — the caller stating its own
requirement. Similar outcome, different judge. If a reviewer can point at a
line of agforge code that decides whether the agent did well, this item is
not finished.

Also drop the event-stream filtering that decides what counts as the agent's
words (`agent_run.py:158-192`, `244-257`) down to the minimum needed to find
the file/marker. Prefer keeping the raw text.

### 2. Delete the runner-side URL verification

Remove `verify_result_url` and its call site (`agent_run.py:307-329,
356-366`). It exists to catch the agent mistranscribing a presigned URL —
wrongness-prevention, exactly the class being removed this turn. If the
class returns, it returns as an observation in the report, not as a guard.

Whether the URL is worth checking is now the agent's call; that `curl`/fetch
is available is a fact the guide may state once.

### 3. Delete the guide-question regex; let the agent read the card

Remove `GUIDE_QUESTION`, `GENERATION_VERB`, `GUIDE_QUESTION_MAX_CHARS`,
`is_guide_question` and the `answered` short-circuit
(`request_service.py:76-106, 189-201`). Every desire now reaches the agent.

Replace with one line of soft instruction, roughly: *"`service/GUIDE.md`
answers what agforge can do and what it costs — read it when the desire is
asking that."* The agent decides whether the desire is a question.

`GET /guide` stays (deterministic evidence read, not an entrance). The
Entrance Guide obligation (devpolicy/policy.md) is met by the agent being
able to answer, not by a matcher. Accepted cost: a capability question now
spends a run — $0 on the ollama backend, real money on `claude`. That
trade-off is the point; record what it actually costs in the report.

### 4. Widen the tool grants (Tool Giving)

- `opencode.json`: `"webfetch": "deny"` → allow. Widen the bash map well
  past the current seven commands — python, curl, image tooling, read-only
  git, `POST /API/ListModels` probes, whatever the agent plausibly reaches
  for. Deny-by-default with a wide allowlist, not `"*": "allow"`.
- `agent_run.py:61-72` (`CLAUDE_ALLOWED_TOOLS`): widen to match.

**Decided (user, 2026-08-10): the allowlist form stays.** Full `*` is
functionally `--dangerously-skip-permissions` in blast radius, because these
agents run natively on the developer's own Mac rather than in a container;
going fully open should follow a move into a sandbox, not precede it. So
this item widens the grant; it does not open it. No further approval needed
to execute.

Corollary for the report: if the agent reaches for something the widened
list still denies, that denial is a **finding** — name the command in
`report.md` and widen next turn. Do not treat a denied command as the
agent misbehaving.

### 5. Rewrite `charter.md` down to facts

Target shape: paths, commands, one line of explanation each. Delete every
prohibition and every policy sentence. Specifically gone:

- "Today's only capability is one still image; anything else cannot be
  fulfilled" (`:18-20`) — the agent can read what the tools do.
- "never pass `--model`, never change model settings" (`:31-33`).
- The 64–2048 bound, the multiple-of-64 rule, "never invent dimensions"
  (`:26-30`) — SwarmUI and the model will say so themselves.
- "never hand-roll S3 calls, never touch `nctl-outbox`" (`:52-54`) — the
  tool refuses; the charter need not.
- The data-shaping rules (`:57-63`) and "Never output `RESULT_URL:` for a
  file you did not verify" (`:90`).
- The finish contract as a mandate (`:65-95`) — see work item 1.

What remains is roughly: the request_id, the desire verbatim, the tool
paths (`scripts/generate.sh`, `service/transform.py`, `service/GUIDE.md`),
where the caller looks for the result, where problem reports live, and the
wall-clock fact. Nothing about what the agent *must* or *must not* do.

### 6. Loosen the problem-report path rule

`problem_path` (`agent_run.py:114-118`) picks the directory name. Reduce to
stating the inbox exists — `.local/problems/` — and that a report left there
gets read. Filename and structure become the agent's. Keep
`AGFORGE_PROBLEMS_DIR` so tests can redirect it.

### 7. Shrink `GUIDE.md` and `README_DEV.md`

Same target as the charter: bullet list of paths, commands, endpoints, each
with a single line. Cut the prohibition sections outright — README_DEV.md's
"Hard rules" and "Agent instruction: no S3 configured" walkthrough, GUIDE's
"What it cannot do today" contract. Cost and latency figures stay (they are
facts, and the Entrance Guide answer depends on them).

### 8. Tests

`tests/test_service.py` currently asserts the marker contract, charter
composition, and the guide-question matcher — most of those assertions are
being deleted along with their subjects. Keep the deterministic shell only:
the HTTP surface, the timeout path, the `AGFORGE_AGENT_CMD` stub. Do not
write tests that re-assert the removed contracts in another form.
`uv run pytest -q` green with no live services.

### 9. Live check — did the capability survive?

The whole point of the turn. Re-run through HTTP, ollama backend, service
restarted on the new code:

- a plain desire (no size stated)
- a size + format desire that needs post-processing, e.g.
  `"a 300x300 png icon of a paper crane"` — 300 is not a multiple of 64, and
  nothing tells the agent that anymore
- an impossible desire (music/video) — does it still report honestly, and
  does it still leave something under `.local/problems/`?
- a capability question and a cost question, which now spend a run
- at least one repeat on the `claude` backend for contrast

Then `report.md` with: which of these still work, what the agent did
differently without the rails, what regressed, what it cost in time and
money versus the ex3 numbers, and — most valuable — where the removed guard
turned out to have been holding something up. Regression is a finding, not
a defect to patch in the same turn.

## Facts worth not rediscovering

- The charter is re-read per request; wording changes need no restart. The
  service and `agent_run.py` do need one.
- The default agent is deliberately weak (ollama via opencode) so that
  wording gaps surface as behavior. Under ex3 it was observed retyping
  presigned URLs lossily, inventing dimensions when the desire was
  size-silent, and ending in prose without a marker. All three counter-
  wordings are being deleted this turn — expect them back, and record them.
- SwarmUI emits JPEG regardless of what was asked. `sips` and Pillow are
  both available.
- ex3 baseline (ollama): ~30–45 s per request, $0. claude: ~18 s, $0.134.
- `AGFORGE_CLAUDE_CMD` in `.local/.env` is an absolute VS Code extension
  path and goes stale on every extension update; the failure looks like
  infra, not config.
- Service: `service/serve.sh` on :8092, hand-started, jobs in memory, log at
  `.local/out/service.log`.

## Out of scope

agautolab and agdevworld (their own turns), music/video/multi-image,
persistent job store, moving the runtime into a sandbox, and any change to
the wire shape agdevworld already speaks.
