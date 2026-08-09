# unshackle_agent / agdevworld turn1 — report

Executed [plan.md](plan.md) on 2026-08-10 (agstudio). Every forced deterministic
path listed for agdevworld in
[shackle_list.md](../../../../../shackle_list.md) is gone except the five the
plan kept. Live-checked against the real assistant, agforge, and the agstudio
autolab node, on the ollama default.

**Headline: the capability survived, and grew.** Ten of eleven live checks
passed; the eleventh failed for a missing tool rather than a missing rail and
passed after that tool was added. The assistant now reads the cluster itself
instead of being handed a digest, and answers questions the digest could not
have answered.

## What was done

| Work item | Result |
|---|---|
| 1. Tool channel | `POST /api/chat` answers with prose, tool calls, or both; the browser runs them and posts raw results back. Backend seam widened to `(system, messages, tools)`; both backends translate the same neutral message shape. Tools: `fetch`, `wait`, `switch_view`, `show_image`. Results — body, status, refusal text — go back verbatim. |
| 2. Prompt-side commands | `ROLE_PROMPT` is now tools, paths and a budget. Every "must", "never", "only when asked" and the whole JSON format contract deleted, including "never invent a price". |
| 3. Digest walls | `summarizeClusterContext`, `summarizeNode`, `summarizeWorkspace`, `summarizeJob` deleted. The envelope validators no longer throw on an unfamiliar `schema` / `kind`; they read what the panels draw. `facts_raw` is no longer withheld. |
| 4. "Ask agent" | Sends the identity of what was clicked and nothing else. The one exception, per plan: an iteration's node-written summary rides along verbatim. |
| 5. agforge exchange | The fixed POST-poll-pick pipeline is gone. The agent starts the request, paces its own polling, and calls `show_image`. |
| 6. Guides | `GUIDE.md` 53 → 47 lines, now tools/paths/costs; the "What it cannot do" contract deleted. `README_DEV.md` rewritten as commands + files + a "Safety devices" section. |
| 7. `fetch-cluster-state.mjs` | "reply with only the download URL" deleted, the reply is scanned for any URL, the three envelope validators deleted. Output path and atomic 0600 write kept. |
| 8. Build | `npm run build` green; `docker compose up --build -d web assistant` green; page and `/healthz` 200. |

Code-side judgment left in agdevworld: none that reads the agent. `main.ts`'s
dispatcher is a name lookup that returns its result to the agent — an unknown
tool or view is now a sentence the agent reads, not a silent drop.

## Live runs

ollama backend (`glm-4.7-flash:latest`), containers rebuilt on the new code.
The browser half of the loop was driven by a stand-in with the same message
shapes, tool implementations and round cap as `src/chatPanel.ts`.

| # | asked | elapsed | rounds | outcome |
|---|---|---|---|---|
| 1 | which nodes are drifting? | 39.1 s | 1 | ✓ fetched `/cluster/state.json` itself, answered from all 19 targets incl. warnings |
| 2 | show me the autolab | 3.8 s | 1 | ✓ `switch_view` |
| 3 | draw me a red lighthouse at dusk | 12.4 s | 8 | ✗ delivery — budget spent polling (F1); the image itself was generated and fine |
| 3b | *same, after F1's fix* | 69.5 s | 7 | ✓ started, waited 25 s + 30 s, verified, displayed (139 KB image/jpeg) |
| 4 | tell me about the node agstudio | 186.8 s | 2 | ✓ but slow (F2) — read `actual.json` (131 KB) and answered with hardware, containers, services, workspaces |
| 5 | tell me about the autolab job binary-cli | 8.9 s | 2 | ✓ status, gates, cost |
| 6 | read me its raw evidence files | 20.7 s | 5 | ✓ hit the 403, reported it honestly, then stalled on the iteration name (F3) |
| 7 | what can you do? | 8.7 s | 0 | ✓ from the card |
| 8 | how much does an iteration summary cost? | 3.2 s | 0 | ✓ "$0.11–0.19, 11–15 s, cached", quoted correctly |
| 9 | start a new autolab mission on agstudio | 9.8 s | 2 | ✓ tried `POST /jobs`, read the 405, explained that this passthrough carries no identity |
| 10 | about iteration iter-0001 (summary in the message) | 11.1 s | 1 | ✓ answered; minor field mix-up (F5) |
| 11 | assistant stopped | — | — | ✓ the page still serves, the chat request fails and the panel says the assistant is unreachable |

Cost: $0.00 — every run was on the local default. Latency is the currency this
turn spent instead (F2).

Safety devices, checked directly: unknown node → `404 unknown_node` naming the
configured nodes; `/evidence/…` → `403 evidence_not_proxied`; `POST /mission`
→ `405` with the identity reason; unknown backend → `502` plus a `failed`
record; claude backend with no key → `502 ANTHROPIC_API_KEY is not set`.

## Findings

### F1 — A pointer to a slow endpoint is useless without a way to wait

Run 3 spent all 8 rounds re-fetching the agforge job in 12 seconds and hit the
budget while the job was still `working`. The image was generated correctly
and was retrievable 45 s later; only the delivery failed.

The old code held the pacing (`setTimeout` 3 s, up to 10 minutes). Deleting the
pipeline deleted the pacing, and the agent had no primitive to replace it with:
it was told the generation takes 20–105 seconds and given no way to spend that
time. **This is agforge's F1 in a new costume — the guide named a door the
grants did not open.** Fixed in-turn per that lesson: a `wait(seconds)` tool
(one wait ≤ 60 s), and the round cap raised 8 → 16, both stated to the agent as
facts. Run 3b then paced itself unprompted — 25 s, then 30 s, then verified.

### F2 — Raw beats digest on quality, loses badly on latency

Run 4 took **186.8 s**, against a digest-era answer that was a few seconds of
model time on pre-chewed lines. The agent pulled `/cluster/actual.json`
(131 KB, `facts_raw` included, previously withheld by code) and answered with
hardware, GPU, every Docker container, services and workspaces — materially
more than `summarizeNode` could produce, since that function chose in advance
what mattered. Run 1 shows the same trade at smaller scale: 39 s for a
whole-cluster answer.

Measured from the run records afterwards (`prompt_tokens` on
`assistant.run.v1`), which is what makes this concrete:

| what the request carried | prompt tokens | duration |
|---|---|---|
| baseline — role prompt + card + a short question | ~1,460 | 2–9 s |
| + one autolab job detail | 2,592 | 5.8 s |
| + the snapshots for run 4 | **45,201** | **183.7 s** |

So the answer cost ~43,700 tokens of tool result and 30× the wall clock of a
comparable question. Latency here is roughly linear in what the agent chose to
read, and the choice is now entirely the agent's — `summarizeNode` used to make
it in advance, at ~20 lines.

The truncation worry named in the plan is answered: 45,201 tokens were
*evaluated*, so nothing was clipped to a small default window, and the model's
own limit is 202,752 — about 4.5× the largest read so far. The ceiling is real
but not close.

### F3 — Deleting the digest also deleted an identifier the agent never learned

In run 6 the agent asked for `summarize/1`. The node answered
`400 bad job or iteration name`; iterations are named `iter-0001`. It read the
error, tried two other paths, then asked the human for the right identifier —
honest, but a capability the popup performs without asking.

The information *was* reachable: `/api/autolab/<node>/jobs/<job>` lists
`evidence[].iter`. The agent listed jobs but never opened one. So this is a
discoverability gap, not a missing grant — the ENT-ordered fix is one line of
fact in the card, not code. **Deliberately not patched this turn**, following
agforge's handling of its F2: one occurrence is not yet proof, and the
user-facing path (the popup's `summary` button, unchanged) still works.

### F4 — Refusals carry themselves; no prompt is needed to teach limits

Every prohibition about the passthrough was deleted from the role prompt and
the card. Runs 6 and 9 walked straight into the 403 and the 405, read the
reason text, and explained it to the user accurately — including "this
passthrough carries no identity". The prohibitions had been carrying
information that the refusal already carries. Same result as agforge's F3, on
a different mechanism: **write the reason into the refusal, not into the
prompt.**

### F5 — Small accuracy drift where labelled fields used to be

In run 10 the agent quoted the *summarizer's* cost and turn count ($0.137, 6
turns) as though they were the iteration's ($0.182, 7 turns). The deleted
`summarizeJob` had labelled those fields in prose, so the mix-up could not
happen. This is the price of raw JSON: field names are now the agent's to
interpret. One occurrence, recorded, not patched.

### F6 — The weak local model tool-calls reliably; the lenient path went unused

`glm-4.7-flash` produced well-formed tool calls in all eleven runs, chose
sensible paths, chained POST → wait → GET → `show_image` without being told the
sequence, and never once emitted the old inline `{"action": …}` JSON. The
lenient alternative kept in `chatPanel.ts` was therefore never exercised —
it costs nothing and remains untested by behaviour.

### F7 — The claude-backend contrast run could not be made

`ANTHROPIC_API_KEY` is not set in `.env` or the environment on agstudio, and
agdevworld's claude backend uses the API rather than a CLI, so the starred
contrast runs from plan item 9 were not possible. What was verified instead:
the backend allowlist errors correctly on an unknown value, and the claude
backend fails with its own words and a `failed` record when the key is absent.
The contrast remains open — it needs a key from the operator.

## Deus Ex Machina note

Added the `wait` tool and raised the round cap on the assistant's behalf while
diagnosing F1 — handoff candidate.

## State after this turn

- `web` and `assistant` containers rebuilt and running on the new code (:8090,
  :8091); `npm run build` green.
- Nothing committed; the working tree holds all edits.
- One agforge job (`91a2a35a…`) was left un-collected by run 3 and later
  completed on its own — its image is the evidence for F1.

## Next turn candidates

1. F3's card wording (name the `iter-NNNN` form, or the job-detail path that
   reveals it), then re-run the evidence question a few times.
2. F2's latency: 183.7 s for one answer is the worst number in this turn.
   Candidates are HTTP range/field selection as *offered* facts (not enforced
   filters), a smaller default snapshot, or accepting it as the price.
3. The claude-backend contrast, once a key exists (F7).
4. Identity for the assistant. It is the single thing standing between this
   turn's kept guard ("no unauthenticated write-through") and letting the agent
   start missions itself — the last big capability it is structurally denied.
5. agautolab's turn, carrying F1's lesson in first: its charters name many more
   doors than agdevworld's did.
