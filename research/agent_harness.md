# Agent Harness Research: Is "the model cannot handle the working directory" really a model defect?

Date: 2026-08-15.
Motivated by the front-role incident recorded in
`pj_agdev/devdocs/episodes/renewed_agautolab/warning.md`, where
`ollama/qwen3.6:35b-a3b-coding-nvfp4` under opencode rewrote a correctly
delivered relative path (prefixing the repository root, prefixing the home
directory, or dropping path segments), while `claude-sonnet-5` under
claude_code used the same path verbatim. That comparison changed the model and
the harness at the same time, so it could not attribute the failure. This
research isolates the variables.

## Method

- A disposable git repository with the working directory set to a
  subdirectory (`<repo>/agent/front`), mimicking the production layout where
  the workspace root and the working directory differ.
- One fixed task: "Read the chat log at `.local/topics/…/chat.md` and reply
  with only the marker line it contains." The target exists both under the
  working directory (correct) and under the repository root (decoy), each
  holding a distinct marker string; in the instrumented batches every run got
  freshly generated unique markers so answers could be attributed to the file
  actually read.
- A logging reverse proxy between the harness and the ollama server captured
  every request byte-for-byte, so all claims about "what the model was told"
  are based on captured payloads, not on assumptions or on the model's
  self-report.
- Versions: opencode 1.18.10, Claude Code CLI 2.1.232, ollama 0.31.1,
  model `qwen3.6:35b-a3b-coding-nvfp4` (35B MoE, nvfp4). The model was
  confirmed to run with its full 262144-token context; prompts were ~7–20k
  tokens, so context truncation was ruled out as a cause.

## Experiments and results

### 1. Reproduction under opencode

The production failure reproduced in the minimal environment: across batches,
20–60% of runs resolved the relative path against the repository root instead
of the working directory (2/6 baseline; 6/10 in a later instrumented batch).
The failure was never random scatter — every wrong resolution used the
workspace root as the base.

### 2. What opencode actually tells the model

The captured system prompt revealed the mechanism:

- opencode's built-in `read` tool declares "The filePath parameter should be
  an absolute path", so relative→absolute conversion is delegated to the
  model itself.
- The `<env>` block presents **two** candidate base directories side by side:
  `Working directory:` and `Workspace root folder:`.
- No rule anywhere in the prompt says which base relative paths resolve
  against.

So the model is asked to make an unstated choice between two plausible bases
on every relative path. Incidentally, when the model disobeyed the tool spec
and passed the relative path verbatim, opencode's `read` tool resolved it
correctly against the working directory — the tool never needed the model to
absolutize at all.

### 3. A prompt-level rule does not fix it

An explicit instruction ("resolve relative paths against the Working
directory, never the workspace root"), injected through the AGENTS.md
mechanism and confirmed present in the captured system prompt, did not
reliably help: one 10-run batch failed 2/10, the next failed 6/10.
Batch-to-batch variance is large; the rule is noise-level.

### 4. String-copy corruption

The instrumented runs surfaced a second, path-independent defect: in 3 of 10
runs the model read the correct file (tool result verified byte-level in the
captured payload, e.g. `marker: ROOT-R6-5037`) and answered a mangled copy
(`marker: ROOT-6-5037`, `marker: R8-2265` — dropped tokens). This is the same
defect class as the dropped path segment seen in production. A model that
cannot faithfully copy a short string it has just read will corrupt paths
regardless of how they are delivered — including absolute ones.

One run answered a marker value that never appeared in its session at all.
Direct-API replay attempts (identical payloads differing only in the tool
result, including cache-warming sequences of 8 identical requests followed by
a changed one, streaming and non-streaming) failed to reproduce any
cross-request contamination — the ollama KV-cache-leak hypothesis found no
support. That single run remains unexplained.

### 5. Same model, different harness: Claude Code over ollama

ollama implements the Anthropic Messages API (since January 2026), so the
unmodified Claude Code CLI can drive the same local model. With identical
fixtures and the same instrumented protocol:

| Harness | Wrong-base path resolution | String-copy corruption |
|---|---|---|
| opencode + qwen3.6-35b | 6/10 (and 2/6 baseline) | 3/10 |
| Claude Code + qwen3.6-35b | **0/10** | **0/10** |

The captured Claude Code system prompt shows the structural difference: it
names exactly **one** directory ("Primary working directory: …"). Its read
tool also demands absolute paths, but with a single base there is no wrong
choice available.

### 6. Operational notes on Claude Code + ollama

- No Anthropic account or real API key is needed. Verified with an empty
  `HOME` and a sanitized environment: `ANTHROPIC_BASE_URL` pointed at ollama
  plus any dummy `ANTHROPIC_API_KEY` value suffices (the variable must be
  set; the backend never validates it). No inference traffic reaches
  Anthropic.
- The `total_cost_usd` in its JSON output is computed from Anthropic's price
  table and is fictitious for a local backend; no billing occurs.
- Telemetry and update checks are separate traffic; disable with
  `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1` and `DISABLE_TELEMETRY=1`.
- Claude Code loads the invoking user's global configuration (memory,
  instruction files) into the system prompt. For in-system agent roles this
  must be isolated (e.g. a dedicated `HOME`), or the operator's personal
  context leaks into the agent — the same leak already observed elsewhere in
  this project.

## General facts established

1. **The failure is an interaction, not a model defect alone.** The same
   model that fails 20–60% under opencode scores 10/10 under a harness that
   presents an unambiguous working directory. "qwen cannot handle the current
   directory" is false as stated.
2. **Harness prompts that delegate path resolution to the model and present
   multiple candidate bases are a latent trap.** Strong models (sonnet) walk
   past it on convention; weaker models fall in probabilistically. The trap
   is invisible when testing only with strong models.
3. **Prompt-level patches do not reliably repair a structural ambiguity** for
   a weak-instruction-following model. Fixing the interface (one base, or
   tool-side resolution of relative paths) works; adding a rule sentence does
   not.
4. **Small local models can corrupt literal strings while copying them from
   their own context.** This is a genuine model weakness (observed only under
   the opencode-shaped context in these experiments, so prompt shape or
   sampling parameters may modulate it), and it caps the reliability of any
   path-passing protocol with such models.
5. **Verify at the wire, not by asking the agent.** Every decisive fact here
   came from captured request payloads. The model's self-reports (it
   correctly named its working directory when asked) were true but useless —
   it knew the working directory and still resolved against the root.
6. **A capable-model harness driving a local model via an API-compatible
   backend is a practical, zero-cost, auth-free configuration**, and it
   doubles as a clean instrument for separating model effects from harness
   effects.

## Discussion

The production workaround (rewriting relative paths to absolute before they
reach the front agent) treated the symptom in the transport layer. These
results locate the disease elsewhere: the harness forces a conversion the
model should never perform, and offers two answers to a question it never
states. The workaround also inherits the string-copy defect — an absolute
path is still a string the model must reproduce faithfully, and it sometimes
cannot.

Three durable options follow, in increasing order of ambition:

1. Keep the model, change the harness: run the front role on a harness that
   presents a single working directory (demonstrated 10/10 here with Claude
   Code over ollama), with per-role `HOME` isolation.
2. Keep opencode, remove the ambiguity: replace its built-in file tools with
   tools that resolve relative paths themselves (custom agent + MCP tools),
   so no path string round-trips through the model.
3. Remove paths from the protocol entirely: hand the front agent the content,
   not a pointer. This is the only option that also neutralizes string-copy
   corruption, and it aligns with Tool Giving — the agent gets a capability,
   not a fragile literal to transcribe.

Sample sizes are single-digit to low-double-digit per condition; the 0/10 vs
6/10 contrast is decisive for direction-setting but the individual rates
should not be quoted as stable numbers.

## Addendum (2026-08-15): third condition — agcode

The privateharness project re-ran this protocol with its own minimal
harness (agcode: single named base directory, tool-side path resolution,
built-in verbatim wire capture replacing the logging proxy, transcript-level
verdicts that require the working-dir marker to have been served in a
`tool_result` before a run counts as OK):

| Harness | Wrong-base path resolution | String-copy corruption |
|---|---|---|
| agcode + qwen3.6-35b | **0/12, 0/12, 0/12** | **0/12, 0/12, 0/12** |

This independently reproduces the Claude Code result and strengthens fact 1:
a second, unrelated harness presenting one unambiguous base also eliminates
the wrong-base failures in the same model. An optional cross-check with
`glm-4.7-flash` (10 runs) showed 0 wrong-base and 1 output-fidelity failure
(correct file served at the wire, model answered the wrong line) —
consistent with fact 4's caveat that copy corruption is a model property
prompt shape can only modulate. Details and wire evidence:
privateharness `devdocs/p2/report3.md` / `report4.md`.
