# retire_old_ui p1 — Step 2: Usage integrated into the dashboard

agdevworld `5d70c17` (workflow_editor only). Not deployed yet; checked on a
read-only service started from the working tree against the live area.

## What was built

- **`server/usage.ts`, `shared/usage.ts`** (`workflow_editor/docs/usage.md`).
  - `GET api/usage?period=today|7d|30d|all` reads every attempt in the
    coordinator's exec store and the attempt's log on the coordinator.
    Each log is read once per size and mtime, as a bounded tail for
    claude_code and the whole file up to 16 MiB for codex.
  - The answer carries per-period totals, coverage with reasons, 14 days,
    breakdowns (project, workflow, harness/model, execution node), the
    newest 200 executions, and the pre-WFE history.
  - `GET api/usage/budget` passes on the relay's subscription windows. It is
    a separate route because a cold read asks the vendors (about 12 s
    measured), and the executions' figures do not wait for it.
- **Measurement meaning kept.**
  - claude_code tokens come from `usage`, which counts the process only.
  - USD is `total_cost_usd`, labelled *API-equivalent* when `costBasis` is
    `list`; it is the list price under a subscription, not a charge.
  - A resumed session's USD is cumulative, so the session's previous total
    is subtracted (`adjusted: resumed`).
  - codex has tokens only and no USD (`tokens only`).
  - An attempt without usage is `unknown` with its reason, and one not ended
    is `running`. Neither is counted as zero, and every total says what it
    does not cover.
- **Relay sources** (`/budget`, `/cost`): each has its own 20 s timeout and
  cache (30 s and 60 s). A relay failure marks only its own section and keeps
  the last answer under its time. A provider failure stays that provider's
  card, as the relay already reports it. A window read more than 10 minutes
  ago is drawn stale. The browser never calls the relay: everything is on the
  dashboard's origin.
- **Dashboard (Home)**, in both modes:
  - a *Usage* card with each login's two first windows, the reset times,
    the 7-day executions and API-equivalent USD, and what that figure does
    not cover;
  - an *Execution nodes* card with each worker's state and its running
    run(s) linked (`api/workers`).

  Both refresh every 20 s and stop when the view is left. The header links
  Usage and the Front Desk. The "← agdevworld" links to the operation room
  are gone from Home, User nodes and Context.
- **`#/usage`**.
  - Simple: the subscription windows with plan, reset times, read times and
    credits; the period's figures (executions, API-equivalent USD, tokens,
    not covered); coverage; projects linked to their pages.
  - Detail adds workflows, harness/model, execution nodes, days, the
    executions with links to their runs, and *History before workflow runs*
    (the relay's `/cost` over pre-WFE `ag.agent-run.v1` records, labelled
    historical).
  - The period choice is a per-viewer convenience. Polling every 60 s
    aborts when the view is left.
- Routine-session attribution is not used anywhere. Executions are attributed
  by attempt → workspace/workflow/run.

## Validation

- `npm run check` in `workflow_editor`: tsc, 451 tests pass, vite build.
- `test/usage.test.ts` (7 tests):
  - per-harness token normalization;
  - a resumed session's cumulative USD;
  - codex tokens without USD, and a continued thread;
  - unknown, missing-log, running and unsupported-harness attempts never
    counted as zero, each with its reason;
  - attribution with each attempt counted once;
  - periods by end time and local midnight;
  - relay failure isolation with the last answer kept, and a provider's
    failure card passed through.
- **Coverage on the live area** (2026-10-20): 407 executions since
  2026-10-11, of which 396 are measured and 11 unknown ("no result event in
  its log", all interrupted claude_code attempts on `studio`); none running.
  The 4 executions of the remote worker `gpupc` are all measured from their
  shipped logs ($1.79 API-equivalent).
- **Reconciled with the source records**: an independent Python pass over
  the same logs gives 217.970688 USD, output 3,172,689 and input 635,055,291
  tokens, 11 unknown and 61 tokens-only. The service gives 217.970687 USD and
  the same tokens and counts. Per worker the figures agree to 1e-6.
- **Browser** (Chromium; desktop 1400×1000 and narrow 390×844, Simple and
  Detail):
  - Home shows the Usage line and the execution nodes;
  - Usage shows coverage, and in Detail the executions and the history;
  - there is no horizontal scroll at the narrow width;
  - no request goes to the relay port;
  - no page errors.

  Two layout defects were found and fixed this way: the top bar did not wrap
  and the tables overflowed at the narrow width.

## Limitations

- Subscription windows are this host's harness logins only. The remote
  worker's own login is not observed, and its executions' USD is likewise
  API-equivalent.
- No metered-key harness runs under WFE today, so no *estimated* USD exists
  for executions. Estimates remain in the history section, from the relay's
  price table.
- A codex thread continued across attempts is flagged, not adjusted. None
  exists in the store, and whether codex turn usage is cumulative across a
  resumed thread is not observed.
