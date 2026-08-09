# Review 1 — Current System vs the Human's Expectations

AI-generated analysis (Omni Agent, 2026-08-09), requested by the human.
Inputs: `human_pondering.md` (the four expectations), `understand_agents.md`
(the agent map), `devpolicy/terms.md` (incl. the new **Deus Ex Machina**
term), and direct reads of each agent's source/docs. Facts below were
verified in source, not recalled.

The four expectations, restated as testable design properties:

- **E1** — One conversational text window per agent; extra endpoints only
  with clear reason. Refined during discussion to: *exactly one
  desire-accepting entrance; additional endpoints must be deterministic
  (evidence reads, auth machinery) and never a second place to express
  desire.*
- **E2** — Agent identity ≠ backend model; one persona, models chosen per
  task difficulty, ideally switchable and comparable everywhere.
- **E3** — A failure-tolerant self-growth system: prevent irreversible loss,
  permit failure, harvest it as knowledge; weak↔strong agents easy to
  compare on any task.
- **E4** — Omni Agent must not absorb in-system agents' work (Deus Ex
  Machina is negative when the goal is developing the inside agents'
  independent workflow).

## E1 — Desire entrances: mostly good fronts, one missing receptionist

| Agent | Entrance today | Verdict |
|---|---|---|
| agforge | `POST /api/requests {"desire": text}` — fully prompt-only; internal agent derives all parameters; `kind` field future-proofs music/video without new endpoints | **Conforms best.** Exactly the anti-`/video?width=` design E1 asks for |
| cagent | Natural-language requests; node entrance (mTLS) and human entrance (token) serve the *same* routes — auth class, not capability, separates them | **Conforms.** The two entrances are deterministic auth machinery, allowed by refined E1; its identity-decides-per-request model is the template for the "please use /mission" receptionist reply |
| assistant | Single `POST /api/chat`, free text in/out | **Conforms**, though it can only answer from a snapshot and switch views |
| autolab gateway | `POST /mission` (bearer) + typed `GET /status,/jobs,...` | **Gap.** The GETs are legitimate evidence reads, but there is *no conversational window at all* — no place to ask "how is project 3's latest job?" in words, no receptionist that redirects a dev request to `/mission`. This is the concrete E1 gap the human named |
| director | CLI, two subcommands (`compose`/`review`), JSON envelope out | **Tension.** An agent invoked as a *function*. Defensible as internal harness machinery rather than an agent's public window — but it is exactly the "traditional function" shape E1 dislikes |
| executor | One plan.md in, report.md out | **Tension**, same shape as director, discussed under E3 |
| node-agent (vision) | `nctl agent attach <node>` interactive session | Conforms by design |

Notable: the ideal in `human_pondering.md` also includes *capability and
cost Q&A* ("what can you generate?", "what does 10s of video cost?") and
*completion notification*. No current entrance does either — agforge is
submit-and-poll, and pricing/capability questions have no addressee anywhere
in the system. That conversational meta-layer (a window that can talk
*about* its service, not just accept jobs) is the unbuilt half of E1 even
where the single-window shape exists.

## E2 — Agent ≠ model: the principle exists, but only as local inventions

Verified per agent:

- **autolab** — full conformance and the origin of the expectation: backend
  is a per-job adapter (`claude_code` one-shot vs `fake`; local backends via
  adapter config), with token/cost captured per iteration in
  `adapter_result.json`.
- **agforge** — closer than the agent map suggested: `AGFORGE_AGENT_BACKEND`
  already switches the request-service agent between `ollama` (default,
  OpenCode) and `claude`, with transcripts recorded either way. Limitations:
  process-level (one backend per service run, not per request), and nothing
  compares the two.
- **executor** — `EXECUTOR_MODEL`/`EXECUTOR_OLLAMA_URL` overrides, but only
  within ollama; the strong-agent side doesn't exist here (arguably by
  design — it probes the floor).
- **director** — hardcoded `claude-sonnet-5`; only the binary path is
  overridable. The clearest E2 violation.
- **assistant** — the code marks an "engine-agnostic seam" but implements
  ollama only.
- **cagent** — one dedicated OpenCode instance; model choice frozen in its
  config, no per-request selection.

So E2 holds in two places (autolab fully, agforge partially), each with its
own invented mechanism (adapter config vs env var). There is **no shared
convention** — no common way to say "run this with the weak/strong backend,"
no common record of which backend served which request at what cost. E2's
second half ("compare easily, everywhere") exists nowhere.

## E3 — Failure tolerance: the harvest machinery exists; the comparison instrument doesn't

What already embodies E3:

- agforge's **problem reports**: on failure the agent writes `problem.md`
  *in its own words*; the harness dictates only the path. Its post-delivery
  check is deliberately minimal — one GET on the delivered URL, commented
  "cheap, deterministic, no judgment... without taking anything away from
  the agent." This is the E3 ideal, implemented.
- autolab's per-iteration evidence, `NOTES.md` handoffs, cost rollups.
- ENT episodes as the review loop.

Safety-device audit, using the two classes from the discussion —
(a) prevents irreversible harm, (b) prevents an agent from being wrong:

| Device | Class | Judgment |
|---|---|---|
| nctl dry-run default, `--yes`, `--allow-destroy`, separate `prune` | (a) | Keep; guards exactly the "不可逆な消去" E3 excludes from tolerance |
| Gitea+Ansible-only node deploy; secrets rules; cagent mTLS/ledger | (a) | Keep; identity and recovery, not judgment-stripping |
| autolab acceptance gates | mixed | Gates are *proposed by the delegate itself* and reviewed — this is judgment given, not taken. Keep |
| director: mechanical manifest/dimension verification, max-2 attempts, never resize/convert | **(b)** | Review candidate. Failure here is cheap and reversible (a bad image); the mechanical clamps exist so the weak point never gets to fail informatively |
| executor: static plan lint, frozen plan contract, single tool | **(b)** | Review candidate; by the human's own §13 test ("そこまで権限を狭くするならスクリプトにすればいい"), executor is defensible *only* as a floor-probing instrument, and only if its failures are being collected as data |
| assistant: snapshot-only, read-only | neither | Interface immaturity, not a safety device |

The larger E3 gap is not any single clamp but the **missing instrument**:
failure/cost evidence is scattered (agforge `.local/problems/`, autolab job
dirs, ENT episodes) and backend choice is uneven (E2), so the system's
stated founding question — *which tasks can a weak local LLM do?* — cannot
currently be answered by any query. The experiment is running without its
measurement apparatus.

## E4 — Deus Ex Machina: named, but invisible in practice

`terms.md` now names the phenomenon and its two-sidedness (acceptable for
mission, negative for developing inside agents' independent workflow).
Nothing yet operationalizes it:

- **No recording.** DEM events leave no trace; the autolab-direction episode
  was noticed by the human, after the fact, by accident.
- **A structural vacuum invites it.** The in-system coordinator (the prime
  agent of `overview.md`) is the system's thinnest piece, so coordination
  work defaults to the Omni Agent. autolab's mediator role is explicitly
  "often the Omni Agent" today.
- **No ownership map.** Nothing states which agent *owns* which class of
  work, so there is no line to notice being crossed. `understand_agents.md`
  is a first draft of one, but it describes what agents *are*, not what work
  is *theirs*.

Minimal conceptual fixes (no code): give every episode doc a one-line DEM
note when the Omni Agent does an inside agent's job ("did X for Y because
Z — candidate for handoff"); and add an ownership column to the future agent
registry. Both make the blind spot visible without restricting anything —
consistent with keeping DEM *available* for mission mode.

## Summary

| Agent | E1 window | E2 model-swap | E3 fails informatively | E4 exposure |
|---|---|---|---|---|
| agforge | ✅ best | 🟡 env-level | ✅ problem reports | low |
| autolab | ❌ no conversational window | ✅ full | ✅ evidence | **high** (mediator = Omni) |
| cagent | ✅ | ❌ frozen | 🟡 | low |
| assistant | ✅ | ❌ (seam only) | ❌ nothing recorded | medium (coordination vacuum) |
| director | 🟡 function-shaped | ❌ hardcoded | ❌ clamps prevent informative failure | medium |
| executor | 🟡 function-shaped | 🟡 ollama-only | 🟡 transcript, no comparison | low |

Ranked distance from expectations:

1. **The comparison instrument is missing entirely** (E2×E3). The system
   was built to learn weak-vs-strong boundaries through failure, but no
   shared backend convention, no cross-agent failure/cost ledger, and no
   "run this both ways" affordance exists. This is the deepest divergence
   because it contradicts the system's stated purpose, not just its style.
2. **DEM is invisible** (E4): no recording convention, no ownership map,
   and a coordinator vacuum that structurally attracts the Omni Agent.
3. **autolab lacks its receptionist** (E1): the one deployed-to-cluster
   agent is also the only one with no conversational window.
4. **director and executor sit in the Tool-Implantation gray zone**
   (E1×E3): agents invoked as functions, clamped so they cannot fail
   informatively. Either their clamps should loosen into the harvest loop,
   or the human's own test says they should become scripts.

The bright spot: **agforge is already the reference implementation** of
E1 and E3 and half of E2 — the expectations are not aspirational; one agent
already lives them. The divergence elsewhere is convention and vacuum, not
impossibility.
