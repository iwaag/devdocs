# agent_mindmap — Roadmap

AI-generated (Omni Agent, 2026-08-09), from `human_pondering.md`, `review1.md`
and the human's review reply. Three phases; p1 is doc-only, p2 and p3 are
independent of each other and may interleave. Write a `p<N>/report.md` per
phase (ENT).

## Scope decisions (from the human)

- Failure/backend **recording**: define one common policy and apply it.
  Comparing or exploiting the records is **out of scope**.
- Deus Ex Machina: policy/guideline text only. No workflow, no tooling.
- autolab gets a conversational window. Every agent except executor must
  have a generic plain-text entrance.
- director becomes a broadly autonomous, highly nondeterministic persona;
  wide first, narrowed slowly later, based on evidence.
- **executor: untouched.**
- Q&A guidance ("what can you do", "what does it cost") is mandatory *in
  form* at every entrance — placeholder or "unknown" answers are fine.
- Destructive phase: **no backward compatibility required.** Experimental
  environment: minimal prohibitions, maximal implementer discretion.

## Prohibitions (complete list)

1. Don't modify executor.
2. Secrets and cluster payloads stay out of git (existing rules).
3. Existing irreversible-destruction confirmations (nctl `--allow-destroy`
   class) stay. Everything else — API shapes, file layouts, existing
   endpoint removal — is implementer's choice.

---

## Phase 1 — Policies and the common record (docs only)

**Goal**: the four expectations exist as written policy; one shared
recording convention exists that every agent harness can follow.

Steps:

1. Add to `devpolicy/policy.md` (guideline tone, matching its current
   style — recommendations, not enforcement):
   - **Single Entrance**: an agent has exactly one desire-accepting
     conversational window; extra endpoints must be deterministic
     (evidence reads, auth machinery), never a second place to express
     desire. Auth may split *entrances* (cagent-style) but identity, not
     endpoint shape, decides what a request may do.
   - **Entrance Guide**: every window must answer capability and cost
     questions, like `--help` on a CLI. Tentative values and "unknown" are
     acceptable answers; absence of the Q&A form is not.
   - **Agent ≠ Model**: backend model/harness is a swappable parameter of
     the agent, never its identity. Every agentic run records which
     backend served it.
   - **DEM note**: when the Omni Agent performs work that belongs to an
     in-system agent, leave a one-line note in the episode doc ("did X for
     agent Y — handoff candidate"). Positive for mission, negative for
     workflow growth; the note is the whole obligation.
2. Add `devpolicy/agent_records.md` — the common recording policy. Keep it
   to one page. Per agentic run, record: request/job id, backend
   (model + harness), outcome (done/failed/aborted), cost/time when the
   backend reports them, and on failure a free-text report **in the
   agent's own words**. Location: each workspace's existing evidence dir
   (`.local/problems/`, autolab job dirs); the policy fixes *fields*, not
   paths or formats.
3. Optionally add `Entrance` / `Entrance Guide` to `devpolicy/terms.md`.

Hints: agforge is the model — `service/agent_run.py` already records
transcripts, problem reports (path-only rule, content agent's own), and
backend choice; autolab's `adapter_result.json` already records cost.
The policy should describe what these two already do, generalized.

Acceptance: policy files read coherently; agforge and autolab conform
without code changes (or with trivial field additions).

---

## Phase 2 — Entrances and guides

**Goal**: autolab has a conversational window; every entrance (agforge,
assistant, cagent, autolab) answers guide questions in some form.

Steps:

1. **autolab window**: add one desire-accepting free-text route to
   `agautolab/agent/gateway.py` (e.g. `POST /window {"text": ...}`).
   Behavior, in words: answer job/progress questions by reading the same
   job state the typed GETs expose; on a development request, reply that
   `/mission` with bearer auth is the door; on capability/cost questions,
   answer from the guide (step 3). Placeholder quality is fine.
2. **Window backend**: a small local model via ollama by default,
   switchable per Agent ≠ Model — copy the `AGFORGE_AGENT_BACKEND`
   pattern (`agforge/service/agent_run.py:agent_backend`) rather than
   inventing a new mechanism or a shared library. Record runs per the
   Phase 1 policy.
3. **Guide files**: give each of agforge, assistant, cagent, autolab a
   small capability card the window reads to answer "what can you do /
   what does it cost" (tentative numbers fine). Precedent: cagent already
   serves `llms.txt` and re-reads it from disk per request — that pattern
   (plain file, no restart needed) is the suggested shape everywhere.
4. Wire the guide into each existing entrance: agforge's service and the
   assistant answer guide questions at their normal window; cagent via its
   existing instructions/llms.txt mechanism.
5. **Backend selectability sweep** (Agent ≠ Model, applied to what already
   exists): wherever an agentic run happens, backend/model must be
   selectable through env/config using the same `AGFORGE_AGENT_BACKEND`
   pattern, and each run recorded per the Phase 1 policy. Concretely:
   - assistant: implement its commented "engine-agnostic seam" — ollama
     stays the default, a strong backend becomes selectable.
   - cagent: expose its OpenCode model choice as explicit, documented
     config (per-request switching not required).
   - agforge: already conforms at process level — just verify and document.
   - autolab delegate: already conforms via adapters — nothing to do.
   Process/config-level switching is enough everywhere; per-request
   switching is optional, not required.

Hints and known terrain:

- `gateway.py` is a stdlib-only single file; keeping it dependency-free is
  nice but not required.
- Deploy path for agautolab1 is agstudio Gitea + Ansible
  (`pj-agdev/.local/devenv.md` has the exact two commands); the node may
  run a stale checkout — push first, then the playbook.
- macOS quirk: a bare `node`/local process may get `EHOSTUNREACH` to LAN
  hosts while curl works; test through the Docker container path
  (see devenv.md).
- The assistant (`agdevworld/assistant/server.mjs`) already has an
  engine-agnostic seam comment — a natural place for the backend switch,
  but upgrading its engine is optional here; only the guide Q&A form is
  required.
- Completion *notification* (vs polling) was part of the original ideal
  but is not required in this phase; note it as future work if skipped.

Acceptance: asking each of the four entrances "what can you do?" and
"what does N cost?" returns a sensible sentence (placeholder ok); asking
autolab's window about a real job returns real state; asking it to build
something returns the /mission redirect. For the sweep: each agent's
docs name the switch, and flipping it on one sample run demonstrably
changes the backend recorded in that run's record.

---

## Phase 3 — director as a creative persona

**Goal**: director stops being a two-subcommand function and becomes a
broadly autonomous conversational agent for creative direction — from
casual human questions ("would John think this way here?", "is this twist
too heavy for this game?") to practical inter-agent requests ("verify this
generated line, pass/fail"). Wide autonomy first; narrow later, slowly,
from evidence.

Steps:

1. Give director a single conversational entrance (HTTP window like the
   others, per Single Entrance; it may also keep a CLI wrapper for
   reconcile.py's convenience — same window, two transports is fine if
   desire flows through one conceptual entrance).
2. Widen the workspace: read access to the direction workspace, the game's
   docs/manifest, and delivered assets — context placement stays the
   sandbox-free philosophy the current director.py states.
3. Loosen the (b)-class clamps *(review1.md, E3 table)*: the mechanical
   manifest/dimension verification and the 2-attempt cap move from
   harness-enforced to advisory — available as tools/info to the agent,
   not gates in front of it. `reconcile.py` may break or be rewritten;
   no backward compatibility.
4. Backend per Agent ≠ Model: default to a strong model (creative judgment
   is the hard-task end of the spectrum), switchable to local via the same
   pattern as Phase 2; record runs per Phase 1 policy.
5. Narrowing comes later, in future episodes, from accumulated records —
   do **not** pre-build guardrails in this phase.

Hints:

- Current shape: `pj-agdev/director/director.py` (one-question runner,
  `compose`/`review`, JSON envelope, `DIRECTOR_CLAUDE_CMD` override) and
  `reconcile.py` (bounded delivery loop). Both are small; rewriting beats
  patching.
- The persona's knowledge (who John is, the game's tone) belongs in files
  in the direction workspace (`brief.md` already exists) — extend that
  pattern rather than stuffing prompts into code, so humans can edit the
  persona without touching the harness.
- agforge's charter template (`service/charter.md` +
  `compose_charter`) is the local precedent for file-based persona/task
  framing.

Acceptance: a casual creative question and a practical verify request both
get in-character, useful answers through the same window; a delivery flow
(compose → agforge → review) still completes end-to-end in whatever new
form; runs are recorded per the Phase 1 policy.
