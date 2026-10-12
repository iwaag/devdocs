# Workflow editor p8 — step 7: deliverables and release readiness

## Deliverables

| Deliverable | Where |
| --- | --- |
| UI implementation | agdevworld `workflow_editor/`, commits `613e00f` (definition screen), `2c6781b` (canvas), `32f61bc` (run view), `b9eac20` (disclosure), `1c82a93` (validation fixes) |
| Representative states and captures | `checks/states.ts`, `checks/statesShots.ts` (`90e0491`, extended in `1c82a93`) |
| Focused browser check | `checks/simplerUi.ts` (71 assertions) |
| Affected checks updated | `ui.ts`, `e2e.ts`, `docsteps.ts`, `runs.ts`, `runInteraction.ts`, `review.ts`, `appearance.ts`, `nodes.ts` |
| Documentation | `workflow_editor/README.md`: the two screens, disclosure, the placement rule, the new checks. `docs/runs.md`: where each browser operation now lives, and that an answer is neither a decision nor a resume. Commit `8cda5c9` |
| Real screenshots and index | [image_after/](image_after/) (39 captures, conditions in its README); the matching baseline in [image_before_states/](image_before_states/) |
| Reports | `report1.md`–`report7.md` and the final `report.md` |

The source revision is agdevworld **`8cda5c9`** (pj-agdev pointer `0a2c100`). `npm run check` and every browser check named in the plan pass on it: the step 6 table, run on `1c82a93`. `8cda5c9` changes only documentation.

## Release readiness (read-only inspection, 2026-10-13)

- `scripts/wfe-release.sh status .local/agdev` (from pj-agdev):
  - current is `a571ae2deb51…`; previous is `6058c0fe…`;
  - `/wfe/api/version` (service) and `bin/wfe version` (launcher) both name `a571ae2`;
  - **open attempts: none**.
- The installed release was not changed. p8 changes presentation only: no store schema, run record, compiler, CLI or server API changed. A rollback is therefore the usual `wfe-release.sh rollback`, with no data step.

### Deployment status: deployed 2026-10-13 (after the person's request)

After this report was written, the person asked for the deploy. `wfe-release.sh deploy .local/agdev 8cda5c9788737b94bd6135f495e0338478a97348` rebuilt and checked the release (211 tests, build), switched `service/current` and `bin/wfe`, restarted `com.agdev.wfe-service`, and reported healthy. Afterwards `status` shows current `8cda5c9…`, previous `a571ae2…`, no open attempt; `/api/version` names `8cda5c9`; the app answers 200 on `:8098`. Rollback: `wfe-release.sh rollback .local/agdev` (to `a571ae2`; no data step). The text below is the pre-deploy assessment.

The plan makes the installed release a separate operation, "from the reviewed source commit". This session did not deploy. The operating service and launcher still run `a571ae2`.

The release includes one fix that matters to the operating service apart from the redesign. In `a571ae2`, after several run views have been visited in one tab, a click is also handled by every earlier run view, each acting on its own run. Stop and Cancel ask through a prompt, once per view, before anything is recorded. With `8cda5c9`, a click is handled only by the run view on screen (report5).

When the person has reviewed the screens and wants it, run from pj-agdev:

```sh
agdevworld/scripts/wfe-release.sh deploy .local/agdev 8cda5c9
```

The script builds and checks the release, refuses while an attempt is open, switches `service/current` and `bin/wfe` together, and requires both to name the commit, else restores the previous release. Afterwards check `wfe-release.sh status .local/agdev`: service and launcher should name `8cda5c9`.

## Still outstanding

- **Native macOS Japanese IME** during background updates: not verified (report6). It needs a person typing in a real browser on this host.
- **Deployment**, as above.
