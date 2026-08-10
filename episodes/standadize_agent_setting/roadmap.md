# Standardize agent settings across agdev projects

## Goal

Give agents in `agforge`, `agautolab`, and `agdevworld` one shared way to
select a harness and model. The standard agent harnesses are `opencode` and
`claude_code`; Ollama is a model provider behind OpenCode, not a harness.

Remove the current naming collisions and direct-model paths rather than
preserving compatibility:

- agforge `ollama` currently means OpenCode + Ollama.
- agautolab `ollama` means a direct Ollama `/api/chat` call, while `claude`
  means Claude Code CLI.
- agdevworld `ollama` means direct Ollama and `claude` means the Anthropic
  Messages API, not Claude Code CLI.

The environment is experimental and not in production. Each phase may
rewrite or delete old settings and implementations. Plans should give the
implementing agent useful facts, tools, and acceptance evidence, then leave
code structure, migration tactics, and recovery choices largely to its
judgment.

## Target shape

A common, language-neutral configuration format defines:

- canonical harness IDs: `opencode`, `claude_code`, and test-only `fake`;
- canonical model IDs such as
  `ollama/qwen3.6:35b-a3b-coding-nvfp4` and
  `anthropic/claude-sonnet-5`;
- named profiles pairing a compatible harness and model;
- project roles selecting profiles;
- local overlays for executable paths, provider endpoints, and secret
  references;
- capability checks where a role needs something special, such as an
  external UI action channel;
- run records that state role, profile, harness, provider, model, outcome,
  duration, usage, and cost when reported.

Prompts, charters, tools, permissions, working directories, artifacts, and
success judgments remain owned by each project and role. Common settings must
not turn those into one deterministic workflow.

Expected roles include:

- agforge: `generator`;
- agautolab: `front`, `director`, `mediator`, `coding`, `summarizer`;
- agdevworld: `front` initially, with room for later internal roles.

## Phase 1 — common contract and conformance examples

Define `ag.agent-config.v1` before changing a runner. Specify the TOML shape,
canonical names, profile resolution, committed-config/local-overlay
precedence, capability declaration, failure behavior, and normalized run
metadata. Extend the common agent-record convention as needed.

The contract's canonical location is `devpolicy/contracts/agent/` at the
workspace root. Keep that directory contract-only: format specifications,
field semantics, explanatory documentation, and illustrative configuration
examples belong there. Scripts, loaders, generated data, live configuration,
environment-specific values, test output, and other implementation artifacts
belong to the implementing projects or their test suites, not to the contract
directory.

Keep the contract language-neutral: agforge and agautolab are Python while
agdevworld is JavaScript. Prefer a small specification plus shared valid and
invalid contract examples over a premature cross-language runtime library.
Each project may turn those examples into its own fixtures and implement its
own loader; cross-project conformance checks live with implementation/tests
and must make semantic drift visible without putting executable machinery in
`devpolicy/contracts/agent/`.

Phase evidence:

- contract documents and examples under `devpolicy/contracts/agent/`, covering
  all known roles and valid/invalid cases such as unknown harness/model,
  incompatible capabilities, and local overlay behavior;
- no scripts, runtime data, machine-specific settings, or generated test
  artifacts in the contract directory;
- an explicit list of legacy names and paths that later phases will delete;
- a short phase report recording unresolved choices for the first adopter.

## Phase 2 — agforge as the first adopter

Move agforge onto the common configuration and use it to prove the contract.
It already runs both desired harnesses, so this phase should be mostly naming,
configuration, metadata, and conformance work:

- rename the OpenCode path from the misleading `ollama` backend to
  `opencode`;
- select `opencode` or `claude_code` through profiles;
- retain agforge-owned tool grants and charter behavior;
- emit normalized run metadata while keeping useful raw transcripts;
- delete superseded environment variables and compatibility branches.

Exercise both harnesses against representative image requests. Agent behavior
is live evidence, not something to encode as deterministic unit assertions.

Phase evidence:

- loader/conformance tests and ordinary agforge tests pass;
- one successful recorded run per harness when locally available;
- settings and run records use only canonical names;
- a phase report notes any contract changes learned from the first real use.

## Phase 3 — agautolab role and runner conversion

Apply the same profiles to all agautolab agent roles. Remove direct Ollama
from the window and make `front`, `director`, `mediator`, `coding`, and
`summarizer` independently selectable between OpenCode and Claude Code.

Use a common process-runner seam inside agautolab for command resolution,
timeouts, OpenCode JSONL extraction, Claude JSON extraction, errors, and run
metadata. Keep role-specific authority separate: a front or summarizer should
not inherit coding tools merely because the harness launcher is shared.

Useful existing seams:

- `agent/gateway.py` owns front, director dispatch, and summarization;
- `agent/session.sh` launches mediator sessions;
- `src/agautolab/adapters/` owns iteration-level coding agents;
- `job.yaml` is the natural place for a coding-profile override;
- mission blocks and `start_mission()` do not depend on a particular model.

Backward compatibility with `AUTOLAB_WINDOW_BACKEND=ollama`, direct Ollama
code, or old adapter configuration is not required.

Phase evidence:

- deterministic stub coverage for both harness protocols and every role's
  profile resolution;
- at least one mission proving front -> mediator -> coding with recorded
  backend identity at each boundary;
- director and summarizer smoke evidence;
- a phase report identifying permission or prompt differences discovered by
  trying both harnesses, without converting those findings into broad rules.

## Phase 4 — agdevworld OpenCode front and tool bridge

Replace agdevworld's direct Ollama chat path with one OpenCode process per
chat request. Preserve its stateless conversation contract: the browser
remains the owner of history and each request supplies the relevant history,
screen context, and guide to a fresh agent run.

Move the tool loop into the agent run:

- server-capable tools such as fetch, wait, autolab, forge, and notes execute
  through a bounded agdevworld tool service;
- UI-only operations such as `switch_view` and `show_image` are collected as
  actions and returned with the final answer for the browser to apply;
- existing reach boundaries, such as the finite autolab node list and raw
  evidence refusal, stay at the tool boundary rather than becoming prompt
  prohibitions.

Prefer a harness-neutral tool boundary, such as a small MCP service, so the
next phase can give the same tools to Claude Code. The implementing agent may
choose another mechanism if live experimentation shows a simpler reliable
route.

Phase evidence:

- OpenCode handles ordinary chat, a multi-step fetch/poll task, view
  switching, and image presentation;
- one `/api/chat` request corresponds to one recorded OpenCode run;
- the browser no longer owns the model's multi-round reasoning loop;
- direct Ollama code and its settings are deleted;
- container build, application checks, and a live browser run succeed.

## Phase 5 — agdevworld Claude Code parity

Add `claude_code` to the same agdevworld front role and tool boundary. Remove
the misleading `claude` backend and the direct Anthropic SDK path unless a
new, evidenced role genuinely requires an API-native chat harness.

The deployment must make binary and authentication availability explicit.
An unavailable selected harness should fail clearly; do not silently fall
back to a different model or harness.

Phase evidence:

- the same representative front-agent tasks run through OpenCode and Claude
  Code without project-specific configuration vocabulary;
- both records carry the same normalized identity and outcome fields;
- direct Anthropic Messages API code and unused dependency/configuration are
  removed;
- observed behavioral differences are reported as findings rather than
  hidden by harness-specific guidance.

## Phase 6 — deployment and local environment reconciliation

Make the standardized configuration reproducible outside the developer
checkout:

- agdevworld images contain the pinned harness runtime and generated local
  configuration they require;
- the agautolab Ansible path installs/configures OpenCode and Claude Code
  availability without unnecessarily coupling a CLI user to an unrelated
  long-running OpenCode server;
- agautolab nodes receive role/profile settings and provider endpoints;
- local executable paths, endpoints, and credentials remain local or
  deployment-supplied rather than committed machine facts.

Use Nautobot/nctl to inspect the intended and actual service placement before
and after node deployment. The scratch environment permits rebuilds,
restarts, migrations, and failed experiments; preserve only material secrets
and evidence worth learning from.

Phase evidence:

- agstudio services start from their documented workflows;
- an agautolab node reports healthy and completes a profile-selected agent
  run using the deployed configuration;
- nctl status/drift evidence is recorded with any unrelated drift called out;
- deployment documentation no longer describes legacy backend names.

## Phase 7 — cross-project convergence and cleanup

Run a small matrix that uses the same named profiles across all three
projects. Confirm that a profile means the same harness and canonical model
everywhere, while each role retains its own tools and judgment.

Delete stale environment variables, duplicate parsers, obsolete dependencies,
dead documentation, and migration-only adapters found during the matrix.
Keep raw failures that teach something; do not add retries, fallback paths,
or instructions merely to make the matrix look uniformly successful.

Phase evidence:

- shared conformance gate passes for all three projects;
- profile/harness/model identity is comparable in their run records;
- each project has a concise live verification record for every available
  standard profile;
- final report describes the resulting contract, remaining exceptions, cost
  and latency observations, and the best next ENT experiments.

## Minimum constraints for phase plans

- Never commit credentials or generated private payloads.
- Do not introduce silent harness/model fallback.
- Keep enough raw output to diagnose a failed agent run.
- Do not add `--dangerously-skip-permissions`, `opencode run --auto`, or an
  equivalent unrestricted native-host mode.

Everything else, including exact module boundaries, MCP implementation,
configuration loader structure, test doubles, rollout order within a phase,
and whether a failed experiment is retried, is the implementing agent's
judgment. Each phase plan should prefer tools and evidence over new mandatory
procedures.
