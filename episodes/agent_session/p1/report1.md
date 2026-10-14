# Agent session p1 — step 1: harness capabilities and the fresh-session baseline

Date: 2026-10-14. Raw evidence (probe streams, baseline script and table) is
in the ignored notes `pj-agdev/.local/agent-session-p1/step1/`.

## What runs today

- The RTS workflows (`rts-vs-bot` `improve-game`, `freework`) name no
  profile, so every attempt uses autolab's `wfrun` role default: profile
  `sonnet` = harness `claude_code`, model `claude-sonnet-5`. All 16 RTS
  attempts on record (improve-game run-001…005) ran that way, on Claude Code
  **2.1.283**, in `bypassPermissions`.
- `agautolab.wfexec` launches one `claude -p --output-format stream-json
  --verbose --model … --allowedTools … --dangerously-skip-permissions` per
  claimed attempt, writes the whole prompt (common contract + project guides +
  brief) to stdin, and records only the pid. No session id is chosen or
  recorded; `server/activity.ts` and Observer read the session from the log.
- `agag.harness.build_argv` has no session parameter. Codex is launched with
  `--ephemeral` (nothing persisted).
- Services checked before probing: `nctl status` ok (Nautobot, worker,
  nodeutils dumps fresh); WFE release `314c98b`; executor idle (no slot, no
  queue); held jevlab experiment untouched.
- The CLI is logged in with a **claude.ai Max subscription**. Every
  `total_cost_usd` below is therefore Claude Code's client-side list-price
  estimate (`costBasis: list`), not a bill, and not subscription quota.

## Capability table — `claude_code` 2.1.283 (the selected profile)

Verified by probes in throwaway directories (two "run copies", A and B) unless
marked *doc*. References: `claude --help` of the installed binary; Claude Code
CLI reference (`code.claude.com/docs/en/cli-reference`) and cost-tracking
guide (`code.claude.com/docs/en/agent-sdk/cost-tracking`), read 2026-10-14.

| Question | Finding |
| --- | --- |
| Explicit creation | `--session-id <uuid>` creates the conversation under an id **the caller chose**; the `system/init` event and the `result` carry it. So the id is known before launch, not only after a successful result. |
| Explicit resumption | `--resume <uuid>` continues that conversation: a fact given in A's first invocation was recalled after the process ended. The init event repeats the same id. |
| Persistent storage | One JSONL transcript per session under the user's Claude config directory, filed by the **cwd of its first invocation**. Normal exit also writes a `cost-state` record (cumulative cost and model usage). |
| Scope of an id | Not scoped by directory: `--resume <A's id>` launched in B **found and continued A's conversation** (appended to A's file, with B as cwd). Docs confirm the id search covers every project since 2.1.223. Isolation between runs must come from WFE never handing one run's id to another. |
| Another run inherits nothing | A fresh session in B (own `--session-id`) answered "UNKNOWN" to the same question. |
| Re-using an existing id for creation | `--session-id <existing>` → `Error: Session ID … is already in use.`, exit 1, before any model call. A creation can never silently attach to an old conversation. |
| Missing session | `--resume <unknown uuid>` → exit 1, `result` `subtype: error_during_execution`, `errors: ["No conversation found with session ID: …"]`, zero usage and cost, no tool ran. Deterministic and distinguishable. |
| Death before the prompt was read | Killed 0.3 s after launch (stdin still open): no transcript exists; resume → "No conversation found". Creating it again with the same id is possible. |
| Death after the prompt was read | Killed 1.5 s after launch: the transcript exists (user turn persisted); resume works and continues it. |
| Death during a tool call | Killed while a `Bash` `sleep` ran: on resume the dangling tool call is closed as interrupted; the agent said the call "was blocked, no result came back" and **did not re-run it**. The command's effect (a marker file) never happened. |
| Concurrent writers | **Not refused.** A second `--resume` of the same id while the first process was still inside a tool call ran to completion and appended to the same transcript. Single-writer must be enforced by WFE/the executor. |
| Prompt on resume | The new stdin text is appended as one more user turn; earlier turns are not re-sent by the caller. The stream does **not** replay old events (only the new turn's events), except after compaction (below). |
| System prompt on resume | *doc + help*: recorded on the conversation's first request and reused verbatim on every resume until compaction (`--system-prompt-snapshot on`, default). wfexec puts contract/guides in the user prompt, so this matters only if that changes. |
| Model/settings on resume | `--model` is **not locked** to the session: resuming A with Haiku succeeded, init reported Haiku, and the whole prefix was re-written to cache (34 k cache-write, 0 read). WFE must compare the session's recorded configuration with the attempt's and decide replacement explicitly. |
| Usage on resume | `result.usage` covers **this invocation only** (and excludes compaction and subagents). `total_cost_usd` and `modelUsage` are **cumulative for the session** (restored from `cost-state`): 0.0615 → 0.0705 → 0.0852 over three invocations. Summing results double-counts; per-attempt reported cost = cumulative now − cumulative at the previous normal end. |
| Killed invocations | A killed invocation writes no `cost-state`; its spend is absent from the next cumulative figure (as it is absent from today's fresh records). Must be labelled unavailable, not zero. |
| Compaction | `/compact` on a resumed session kept the **same id**, emitted `system/compact_boundary` with `compact_metadata {trigger, pre_tokens, post_tokens, …}`, then a synthetic user "summary" event in the stream; its `result.usage` was all zero while the cumulative cost rose 0.0176. Context survived (codeword recalled). Auto-compaction uses the same event (`trigger: auto`, *binary strings*). |
| Prompt cache | On this subscription the main conversation's writes are **1-hour** TTL (`ephemeral_1h_input_tokens`). ~20–30 k tokens of system prompt/tools are already a cache hit for brand-new sessions. |

### Other harnesses

| Harness (profiles) | Status in this phase |
| --- | --- |
| `claude_code` (`sonnet`) | **Supported** — explicit create/resume verified above. |
| `codex` (`codex`, `codex-mini`) | Unsupported: launched `--ephemeral`, nothing to resume. |
| `agy` (`agy`, `agy-claude`), `gemini_cli` (`gemini`) | Unsupported: resume behaviour not verified; not implemented. |
| `agcode` (`local`), `fake` (`stub`) | Unsupported: no conversation store. |

`run` policy on an unsupported profile must be refused visibly (step 2), never
executed fresh every time.

## Fresh-session baseline (RTS improve-game, 16 attempts)

Per attempt, from each attempt's own stream log (dedupe by assistant message
id; `result` for totals). Cost = harness-reported client estimate.

| Run | Attempts | Reported cost | Cache write | Cache read | Output | Mean 1st-request context | Active time |
| --- | --- | --- | --- | --- | --- | --- | --- |
| run-001 | 3 | $1.60 | 121 k | 4.50 M | 21.6 k | 34.7 k | 327 s |
| run-002 | 3 | $2.00 | 156 k | 5.17 M | 34.0 k | 35.7 k | 506 s |
| run-003 | 3 | $2.67 | 214 k | 7.28 M | 36.1 k | 38.4 k | 588 s |
| run-004 | 4 | $2.55 | 173 k | 7.51 M | 33.7 k | 38.5 k | 657 s |
| run-005 | 3 | $3.45 | 192 k | 10.54 M | 46.2 k | 39.7 k | 1029 s |
| **all** | **16** | **$12.28** | 856 k | 35.0 M | 172 k | 37.5 k | 3107 s |

Uncached input is negligible (≤ 170 tokens per attempt). Each fresh attempt's
first request is ~35–40 k tokens, of which ~20–26 k were already cached
(system prompt and tools shared by every session) and **13–20 k were written
fresh**: the contract, guides and brief. The typical pattern is three
attempts per run (plan + question, implement…present + question, cleanup +
report); the middle attempt carries most of the cost (50–90 turns, 4–8 M cache
reads).

What this says before any comparison:

- Fresh sessions already get large cache hits; the fixed bootstrap a retained
  session could save is roughly one 13–20 k cache write per later attempt plus
  the re-orientation turns (reading the run, plan and files again).
- A retained session instead carries the previous attempts' whole transcript
  into every later request. After a 50–90-turn attempt that context can be
  well above the bootstrap, so later attempts may pay *more* in cache reads.
  Whether retention saves anything is an empirical question for step 5.

## Probe cost

About $0.40 reported across the probe invocations (a few cents each; the
killed ones reported nothing).

## Decisions carried into steps 2–4

- Session identity for `claude_code` is **chosen by the executor** (a UUID per
  session) and passed with `--session-id` on creation, `--resume` afterwards.
  It can be recorded in `attempt.begin` before launch.
- Resume failure modes are classified from the stream: `No conversation found`
  (exit 1, no usage) = the conversation was never persisted or is gone →
  explicit replacement path; anything else after launch = an ordinary
  interruption.
- The configuration a session was created with (harness, profile, model,
  contract/guides digests) is recorded with it and compared at each claim.
- Per-attempt cost under `run` = cumulative delta; usage from deduplicated
  per-invocation `usage`; compaction events recorded.
- Single-writer is WFE's existing one-attempt fencing plus "never launch while
  a process of the run lives"; the harness itself gives no protection.
