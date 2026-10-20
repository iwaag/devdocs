# retire_old_ui p1 — Step 5: verified and deployed

Deployed 2026-10-20 on the coordinator host:

- the WFE release agdevworld `dbd974b`;
- the relay restarted on the same commit;
- the web container rebuilt.

pj-agdev pins it at `69a8bd0`. The machine-specific addresses, release
paths and rollback commands are in ignored local evidence
(`pj-agdev/.local/devenv.md`, *retire_old_ui p1 on this host*, and
`pj-agdev/.local/retire-old-ui-p1/`).

## Checks

| Check | Result |
|---|---|
| WFE `npm run check` inside the release build (tsc, tests, vite) | 451 / 451 pass |
| agdevworld bundle `npm run build` | passes |
| relay `uv run pytest -q` | 140 pass |
| executor tests | not run: the executor is unchanged (usage is read from its existing records) |
| `wfe-release.sh deploy` health check | the service and `bin/wfe` report `dbd974b` |
| relay after restart | mirror `live (resumed queue)`, 3 Zulip calls |

- The kept relay routes answer `200`: settings, contexts, progress, argues,
  cost, budget. The removed ones answer `404`: ops, routines, agents, work,
  inflight, complete, frontdesk.
- Before the restart: no open execution attempt (`wfe-release.sh status`),
  so no execution was interrupted.

## Browser verification of the deployed site

Chromium drove the LAN name, at desktop 1400×1000 and narrow 390×844
(`deployed.ts`; output and screenshots in the local evidence). All checks
pass:

- Site root `:8090/` lands on the dashboard at `:8093/wfe/`. From there a
  project opens, then its runs, then a run view.
- Home links Usage and the Front Desk. There is no "operation room" or
  "← agdevworld" link.
- Usage, in Simple and Detail:
  - coverage reads "407 executions … 396 measured, 11 unknown, 0 running";
  - executions link their runs (for example front · converse/run-036);
  - the history section is present.
- No horizontal scroll at the narrow width on Home, Usage or Usage Detail.
- Window states, from intercepted responses on the deployed page:
  - fresh: the meter and percent are drawn;
  - stale: "Stale: last read …";
  - provider failure: that card says "Unknown now — codex app-server did not
    answer", and its last reading is greyed, while the other card stays
    fresh;
  - relay failure: "Subscription windows unavailable", with the last answer
    and its time.
- An unavailable execution-usage read is said on Home, not shown as zero.
- No request goes to a removed endpoint (old relay routes, gauge or graph
  pages, cluster snapshots, sample JSON). The dashboard never calls the relay
  from the browser.
- Existing Front Desk conversation, opened with a Slack-style link
  `/?view=frontdesk&conv=p8ex1-trial-b` on both `:8093` and `:8090`:
  - the URL is kept, and the page loads the conversation from WFE;
  - "Home ↗" points at the dashboard;
  - no page error.
- The conversation's history over the route has 3 messages and 1 linked
  run. That run link opens its run view (front · converse/run-030).
- Composition on the isolated fixture (`/?view=frontdesk&demo=1`): a typed
  message is shown and answered (0 → 4 posts). No conversational write left
  the page.
- Representative totals were reconciled with the source records in Step 2:
  an independent pass over the logs agrees to 1e-6 USD and exactly on tokens
  and counts.

## Over the LAN from another PC

From the GPU PC (agpc, through the cluster's Ansible inventory):

- `:8093/` and `:8090/` answer `302 → :8093/wfe/`;
- the Front Desk conversation URL answers `200`;
- `/wfe/api/version` answers `200`.

## Deployment and recovery evidence

- **WFE**: release `dbd974b`, previous `075500c`. Code only, no store
  change, so rolling back is activating the previous release.
- **Relay**: runs from the agdevworld checkout. A copy of the installed
  launchd plist was kept, although the plist itself was not changed: the
  unused `AGENTROOM_STALLED_SECONDS` it still sets is ignored.
- **Web**: the previous image is kept under a tag. The new image serves the
  entrance rule.
- **Operating data** was preserved:
  - exec store, conversation store, mirror store, run records;
  - the pre-WFE `ag.agent-run.v1` records, still readable as history.

  Only the ignored, regenerable cluster snapshots under
  `agdevworld/public/cluster/` were deleted.

## Nautobot

No change was needed.

- `agdevworld-web` is the only registered service of this system. Its
  placement, ports and lifecycle are unchanged.
- The relay and the WFE service were not registered before and are neither
  added nor retired by this phase.
- `nctl drift` before and after: 46 converged, 1 drifting. The drifting
  one is `agobserver`, `service_missing`, which predates this work and has
  nothing to do with it. `agdevworld-web` and workspace `agdevworld` are
  converged.

## Not verified, and what that means

- No live agent message was sent, as the plan says. A real Front Desk post
  goes through the same unchanged WFE route that the read verified. The
  composer was exercised on the fixture only.
- The Arguing Room's close was verified by the relay tests only, not
  against a live argue.
