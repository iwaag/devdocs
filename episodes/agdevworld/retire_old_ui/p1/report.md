# retire_old_ui p1 — final report

The agdev dashboard is now the site's Home. Usage is integrated into it,
built from current workflow execution evidence with explicit coverage. The
world views, the operation room and ops board, the routine dashboard and the
cost gauge page are gone, together with their relay routes and the work
behind them. The Front Desk works as before and links Home. All of this is
deployed and verified over the LAN.

Step reports: [report1](report1.md) (boundaries), [report2](report2.md)
(Usage), [report3](report3.md) (entrance), [report4](report4.md) (removal),
[report5](report5.md) (verification and deployment).

## Changes

| Repository | Commit | What |
|---|---|---|
| agdevworld | `5d70c17` | Usage in WFE: `api/usage`, `api/usage/budget`, the Home Usage and Execution nodes cards, `#/usage`, `docs/usage.md`, tests |
| agdevworld | `91f58bd` | The entrance: nginx redirects a bare `/` Home; the bundle is the Front Desk / Arguing Room only; the Front Desk links Home |
| agdevworld | `dbd974b` | Retired UI and its relay routes and runtime removed; READMEs updated |
| pj-agdev | `69a8bd0` | Pin `dbd974b`; launchd template and README for what the relay still serves |
| devdocs | this episode, README_DEV | Step reports; *The agdev site* section; retired mentions marked historical |

## Completion criteria

1. **Opening the site presents the agdev dashboard.** `:8093/` and `:8090/`
   redirect to `:8093/wfe/`, checked from this host and from another PC.
   Any other address of the old bundle lands Home.
2. **Usage is integrated and reflects current WFE execution evidence.**
   - Every attempt in the exec store is read from its log on the
     coordinator; remote workers ship theirs.
   - Unknown (11 interrupted attempts) and running are never zero.
   - API-equivalent USD (claude_code, list price under a subscription) is
     kept apart from tokens-only executions (codex). A resumed session's
     cumulative USD is made per attempt.
   - The figures match an independent recount.
   - Subscription windows show fresh, stale and unavailable states, with
     provider failures isolated.
   - It uses the dashboard's surfaces and its Simple/Detail modes.
3. **The retired screens are absent**, along with:
   - their renderers, styles, samples and polling;
   - the cluster snapshot importer;
   - the relay routes `/agents`, `/work`, `/ops`, `/routines`,
     `/inflight`, `/chat`, `/complete`, `/frontdesk` and the argue writes,
     and the engine parts behind them.

   No request to a removed endpoint is made by any remaining page.
4. **The Front Desk is unchanged.**
   - Existing conversations open from Slack-style links on both listeners
     and read their history and run links from WFE.
   - Composition works on the isolated fixture.
   - It was not redesigned or moved.
5. **No project was migrated and no history was deleted.** The exec,
   conversation and mirror stores, the run records and the pre-WFE agent
   records are kept; the pre-WFE records are now shown as history in Usage
   Detail. Only regenerable, ignored cluster snapshot files were removed.
6. **Deployed and checked over the LAN.**
   - WFE: 451 tests. Relay: 140 tests. Bundle: build passes.
   - Browser checks on the deployed site, desktop and narrow, all pass.
   - The service versions and nctl state are recorded in report5.
7. This report.

## Collection limitations

- Subscription windows come from this host's harness logins only. The
  remote worker's own login is not observed. Its executions are measured
  from their shipped logs, and their USD is API-equivalent as well.
- No executions carry *estimated* USD, because no metered-key harness runs
  under WFE. Estimates remain only in the pre-WFE history.
- A codex attempt that continues a thread is flagged, not adjusted. None
  exists, and codex's resumed-thread semantics are unobserved.
- Usage covers the exec store from its start (2026-10-11). Earlier activity
  is the relay's pre-WFE history.

## Findings recorded, not changed

- The `/wfe/` route does not refuse host-path operations, although the
  Home text said it did. The gate was removed in workflow_editor p11, and
  the proxy change kept the same behaviour; only the stale text was
  corrected (report3).
- `nctl drift` reports `agobserver` `service_missing`, which predates this
  phase.

## Kept deliberately

- **The Arguing Room** (argue history and close). It was not in the
  retirement scope and holds historical argues. It keeps the relay's
  completion door and the conversation index alive.
- **The relay process.** The Front Desk's settings, contexts and progress
  panels, the Arguing Room and Usage read it.

## The remaining Front Desk episode

This phase changed only the Front Desk's "Home ↗" link and the page frame.
The next episode is about an entertaining conversational UI for talking
directly with project agents, Front included: its interaction design, its
presentation, and how it relates to workflow execution. That episode also
inherits these questions:

- whether the Arguing Room and the desk's relay-backed panels (progress over
  the legacy Zulip mirror, contexts, settings) still belong there;
- with them, whether the relay and its mirror are still needed.
