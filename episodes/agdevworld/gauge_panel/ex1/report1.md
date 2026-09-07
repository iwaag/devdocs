# gauge_panel ex1 step 1 — relay `/budget`

`GET /budget` on the agentroom relay answers `ag.budget.v1`: one card per
harness, each read through the CLI's own route, each cached and failing on
its own. `agentroom/src/agentroom/budget.py`, `tests/test_budget.py` (6
tests, 125 in the suite), route in `server.py`, wired in `main.py`
(`check` prints the cards too), README section, plist template.

## What it reads, live from agstudio (2026-09-07, ~20:40 JST)

Relay run on a spare port with no ops credential, `curl /budget`:

| harness | plan | windows (percent used) | read |
|---|---|---|---|
| claude_code | max, `default_claude_max_5x` | session 48 % · weekly all 19 % · weekly Fable 30 % | 0.39 s |
| codex | plus | 5h 0 % · weekly 0 % · reset credits 3 available | 0.89 s |
| agy | — | `ok: false`, "not implemented yet (step 3)" | — |

First `/budget` 0.85 s (the two vendors in parallel threads), the second
17 ms from the per-provider cache. Claude's session percent had moved from
the plan's 38 % to 48 % between the plan's probe and this one — that is this
Omni session spending it.

## Shape

```
{schema: "ag.budget.v1", generated_at,
 harnesses: {
   claude_code: {ok, plan, tier, source, windows: [{kind, label, percent, resets_at, severity, scope}],
                 note, read_at, credential_renewed_at, token_expires_at, extra_usage, error},
   codex:       {ok, plan, source, windows: [{kind: "5h"|"weekly", label, percent, resets_at, window_minutes}],
                 reset_credits: {available, expiring_at, titles}, credits, limit_reached, error},
   agy:         {ok: false, error, source, windows: [], stale}},
 settings: {cache_seconds, read_timeout_seconds}}
```

- `percent` is *used*, as the vendor said it. `resets_at` is epoch seconds,
  like every other stamp on the relay, so the card computes "resets in".
- A failed card is `ok: false`, the reason, `windows: []`, and `stale` — the
  last good card kept apart from the failure so the page can grey it. No
  provider ever answers 0 for "did not answer" (tested).
- `limits[]` is what the claude card renders; the named top-level fields
  are only the fallback, because they sit beside experiment keys that come
  and go. The per-model scoped window arrives labelled `weekly, Fable`.
- The claude card's `note` is the sentence the gauge needs: *cost_usd on
  claude_code records is the API-equivalent price, not money leaving an
  account: a Max plan meters these windows, not dollars.*

## The credential invariant

The relay reads `~/.claude/.credentials.json` and never writes it; the
tests assert the file is byte-for-byte unchanged after a read and that no
request ever carries the refresh token. A token the file itself says is
expired is not even sent; a 401 is *unknown* with "token expired; the next
claude_code run refreshes it", and `credential_renewed_at` is the file's
mtime. Measured today: the file was renewed at 13:08 JST and the token's
`expiresAt` is exactly 8 h later, so the window between a token expiring
and the next claude_code run is where the card will honestly read unknown.

codex is read through `codex app-server` with stdin held open until the
`id: 2` reply, then killed — closing stdin first makes it exit silently
(re-measured: a `printf | codex app-server` pipe prints nothing). The
app-server refreshes its own ChatGPT token; `auth.json` is never opened.

## Configuration, all paths

`AGENTROOM_CLAUDE_CREDENTIALS` (default `~/.claude/.credentials.json`),
`AGENTROOM_CODEX_BIN` (default `codex`), `AGENTROOM_AGY_BIN` (default
`~/.local/bin/agy`), `AGENTROOM_BUDGET_SECONDS` (60). The plist template
gains a `__HOME__` placeholder for the two binaries — `~/.local/bin` is not
on the launchd PATH — documented in `devenv/launchd/README.md`. The
installed plist is not touched until step 4, because a changed
`EnvironmentVariables` block needs `bootout`/`bootstrap`, not `kickstart`.

## Not done here

- agy is a stub answering unknown (step 3).
- `serve.sh check` was not run: it costs a 241-call sweep, and the budget
  half was proved on the spare-port relay instead.
