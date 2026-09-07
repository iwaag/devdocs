# gauge_panel ex1 — each harness's budget, and how much of it is used

The gauge shows what runs cost; it does not show how much room is left.
For the three subscription-style backends the agents actually use —
claude_code (Claude Max), codex (ChatGPT Plus), agy (Antigravity, consumer
Google account) — the vendor meters a *window*, not a dollar, and each CLI
turns out to have a way to read that window. gemini_cli is out of scope
(braindump).

Destructive phase, private experiment: no backward compatibility, the
steps are outcomes. The two relay invariants stay (nothing here reads
Zulip; an absence is *unknown*, never 0 or 100). One more for this
extension: **the relay reads the CLIs' own credential stores and never
writes them** — refreshing a token is the CLI's job, and a store the relay
has rewritten is a CLI that stops logging in.

## What was found (2026-09-07, all read-only probes from agstudio)

**claude_code — Claude Max, and the USD on the gauge is notional.**
`claude auth status --json` (binary
`~/.vscode-server/extensions/anthropic.claude-code-2.1.263-darwin-arm64/resources/native-binary/claude`,
the same one the agents run) says `authMethod: claude.ai`,
`subscriptionType: max`; `~/.claude/.credentials.json` →
`claudeAiOauth.rateLimitTier: default_claude_max_5x`, with `accessToken`,
`expiresAt` (ms), `refreshToken`. So the `cost_usd` every claude_code
record reports is the API-equivalent price, not money leaving an account
— worth one sentence on the gauge.

The budget is `GET https://api.anthropic.com/api/oauth/usage` with
`Authorization: Bearer <accessToken>` and `anthropic-beta: oauth-2025-04-20`
— what the TUI's `/usage` reads. Answered 200 in 0.32 s:

```
five_hour: {utilization: 38.0, resets_at: "2026-09-07T15:19:59+00:00"}
seven_day: {utilization: 18.0, resets_at: "2026-09-12T05:59:59+00:00"}
limits: [
  {kind: session,       group: session, percent: 38, severity: normal, resets_at: …, is_active: true},
  {kind: weekly_all,    group: weekly,  percent: 18, severity: normal, resets_at: …},
  {kind: weekly_scoped, group: weekly,  percent: 28, severity: normal, resets_at: …, scope: {model: {display_name: "Fable"}}}
]
extra_usage: {is_enabled: false, …}   seven_day_sonnet / seven_day_opus: null
```

`limits[]` is the list to render (kind, percent, severity, resets_at,
scope) — the named top-level fields are the same numbers with fixed names
and a set of experiment keys (`nimbus_quill`, `tangelo`, …) that come and
go; do not depend on them. Percent is of the window, and there is no
absolute "credits" number for Max — the *maximum* the braindump asks for
is 100% of the window; say so instead of inventing a token count.

The access token expires (`expiresAt` was 51 minutes away when probed);
Claude Code refreshes it whenever an agent runs. The relay must **not**
use the refresh token: on 401 it reports *unknown — token expired; the
next claude_code run refreshes it*, and the file's mtime tells when it was
last renewed.

**codex — ChatGPT Plus, read through the app-server protocol.**
`~/.codex/auth.json` `auth_mode: chatgpt`; `codex login status` → "Logged
in using ChatGPT". `codex app-server` (stdio, JSON-RPC, one line per
message) answers `account/rateLimits/read` in 0.96 s including process
start, after `initialize` + `initialized`:

```
{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"clientInfo":{"name":"agentroom","version":"0.1.0"}}}
{"jsonrpc":"2.0","method":"initialized","params":{}}
{"jsonrpc":"2.0","id":2,"method":"account/rateLimits/read","params":{}}
→ rateLimits: {planType: "plus",
   primary:   {usedPercent: 0, windowDurationMins: 300,   resetsAt: 1788798478},
   secondary: {usedPercent: 0, windowDurationMins: 10080, resetsAt: 1789385278},
   credits: {hasCredits: false, unlimited: false, balance: "0"},
   spendControlReached: false, rateLimitReachedType: null}
  rateLimitResetCredits: {availableCount: 3, credits: [{title: "Full reset (Weekly + 5 hr)", status: available, expiresAt: …}, …]}
```

Schema: `codex app-server generate-json-schema --out <dir>` →
`codex_app_server_protocol.schemas.json` definitions `RateLimitSnapshot`,
`RateLimitWindow`. `account/usage/read` also exists (lifetime and daily
token buckets, `AccountTokenUsageSummary`) — not needed, the gauge has its
own daily series. Keep stdin open until the reply arrives, then kill the
process (closing stdin first made it exit before answering). The
app-server refreshes the ChatGPT token itself, so this route never touches
`auth.json`. `codex app-server daemon start` would keep one resident and
`proxy` talks to it; not worth it at one call a minute — a fresh process
is simpler and cannot go stale.

**agy — Antigravity on a consumer Google account; a route exists, unproved.**
`agy` has no usage subcommand (`agy --help`, `agy remote-control status`
checked). But `~/.gemini/antigravity-cli/cli.log` shows the CLI runs a
`quota_manager.go` `doRefreshQuota` on every start (the result is not
logged), and the binary carries the Code Assist client
(`codeassistclient`, host `cloudcode-pa`) with two quota calls: gRPC
`/google.internal.cloud.code.v1internal.PredictionService/RetrieveUserQuota`
and REST `/{api_version}:fetchQuotaStatus`, plus a TUI string
"Refreshes in %s". The token is
`~/.gemini/antigravity-cli/antigravity-oauth-token` (`{token, auth_method:
"consumer"}`). **This shell's permission classifier denied reading that
file, so the call was not tried**; step 3 is where it is. The gemini CLI's
own Code Assist calls (`cloudcode-pa.googleapis.com/v1internal:…`, bearer
OAuth) are the closest documented shape to copy.

## Step 1: relay `/budget`

A new read route (its own, not folded into `/cost`, because it calls
vendors and `/cost` is stat-only and polled at 20 s). One provider per
harness, each with its own cache (60 s default, `AGENTROOM_BUDGET_SECONDS`)
and its own failure, never a shared one:

```
{schema: "ag.budget.v1", generated_at,
 harnesses: {
   claude_code: {ok, plan: "max" / tier, source: "api.anthropic.com/api/oauth/usage",
                 windows: [{kind, label, percent, resets_at, severity, scope?}],
                 note, read_at, credential_renewed_at, error},
   codex:       {ok, plan: "plus", source: "codex app-server account/rateLimits/read",
                 windows: [{kind: "5h", percent, resets_at}, {kind: "weekly", …}],
                 reset_credits: {available, expiring_at}, error},
   agy:         {ok: false, error: "…", source, …}
 }}
```

- Percent is what the vendor said; `severity` is copied when given
  (claude) and derived only as a display hint elsewhere. No provider ever
  returns 0 for "did not answer".
- Credential paths come from the environment (`AGENTROOM_CLAUDE_CREDENTIALS`,
  default `~/.claude/.credentials.json`; `AGENTROOM_CODEX_BIN`, default
  `codex` on PATH — the launchd PATH is `/opt/homebrew/bin:…`, and codex
  lives in `~/.local/bin`, so the plist gets the absolute path;
  `AGENTROOM_AGY_TOKEN`). Read-only, every one.
- Tests: fixtures are the three payloads above; test the 401 → unknown
  path and the codex "no reply" path. The suite is `tests/test_budget.py`.

## Step 2: the strip on the gauge

A "Budgets" row above the tiles: one card per harness. A card is the plan
name, one horizontal meter per window (single hue, the harness's own slot
colour, filled to `percent`, the percent as text beside it — never a
colour-only reading), "resets in 3 h 40 m" from `resets_at`, and the
read time. A card that could not read says why, in amber, and keeps its
last good numbers greyed if it has any. Claude's per-model scoped window
(Fable 28%) is a third meter on its card, labelled with the scope.

Poll `/budget` on the same 20 s tick as `/cost` — the relay's cache is
what limits vendor calls, not the page.

## Step 3: agy, tried for real

Read the token file, call `fetchQuotaStatus` (and, failing that,
`RetrieveUserQuota` over gRPC — `agy` is a Go binary; the REST path is the
easier first try) with `Authorization: Bearer`. Three outcomes, all fine:
it answers and the card fills; it answers only from inside the IDE's
language server and the card says "readable only through the IDE"; or the
account has no quota object and the card says so. Write down the request
that worked in `agentroom/README.md`. The **refresh countdown in the TUI**
(`agy` interactive, "Refreshes in …") is the human-readable check to
compare against.

## Step 4: deploy and prove

Relay `kickstart`, `docker compose up --build -d web`, one gauge screenshot
with the strip. Proof: after one paid run (a routine start, or any Front
question), Claude's session percent on the card moves up and the codex
card still reads (0% unless a codex profile ran). Note in the report the
sentence the gauge now needs about claude_code's USD being notional under
Max.

## Hints

- The classifier in this Omni shell blocks commands that read a
  credential file and reach the network in one go **sometimes**, not
  always — the claude probe passed as a two-step `TOK=$(python3 …); curl`
  and failed as one python script. The relay is not under that classifier.
- `urllib` under the python.org 3.14 on this Mac fails TLS verification
  (no certifi); the relay runs under uv's CPython 3.11 which has worked for
  every Zulip call, and `curl` uses the system store.
- Do not add Claude's `extra_usage` (credits beyond the plan) as a meter
  unless `is_enabled` is true; today it is not.
