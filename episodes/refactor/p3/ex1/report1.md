# refactor p3 ex1 — step 1: the dead autolab UI is gone

## What was actually dead, and how far

`agdevworld`'s `autolab / now` view read `/api/autolab/<node>/…`. That
passthrough lived in the embedded assistant service, which
`modernize_agdevworld` p1 deleted a month ago; nginx serves static files only.
So the whole view — node picker, mediator headline, `projects` tab, `jobs`
tab, iteration evidence, and the paid per-iteration `summary` button — had
been failing on every click since then. It is the same finding `refactor` p3
made about `tasks / plane`, one screen later.

Nothing else in the realm called those routes. `agentroom/src/agentroom/`
also has an `autolab.py`, and it is **not** this: it reads autolab's record
out of Zulip conversations and is live. It was traced and left alone.

## Deleted

**Browser side (`agdevworld`)**

| what | where |
|---|---|
| the whole feed client | `src/autolabState.ts` (248 lines, deleted) |
| the view config, its chips, node picker and job/project row styles | `src/views.ts` |
| the `autolab` and `autolab-project` selection types | `src/views.ts` |
| the job popup, the project popup, the iteration list and the summary button | `src/detailPopup.ts` (~180 lines) |
| the `.dp-iter*` / `.dp-summary.collapsed` popup CSS | `src/detailPopup.ts` |
| the scene and its import | `src/worldViews.ts` |
| the `autolab` nav entry | `src/viewSwitcher.ts` |

Navigation is reconnected: `workspaces`'s `switchTo` now names `agentroom`,
and `VIEW_KEYS` is five — `nodes → workspaces → agentroom → ops → routines →`
back to `nodes`. Shared rendering utilities the retained views still use
(`.dp-summary-text`, `.dp-summary-meta`, `.dp-summary-error`, `STATUS_COLOR`,
`kvList`, `el`) were kept; only the ones with no remaining caller went.

**Node side (`agautolab/agent/gateway.py`, 181 lines removed)**

The gateway kept a *stub* surface — `/status`, `/log`, `/jobs…`, `/projects`,
`/game`, `/monitor`, documents marked `"stub": true` — and its own docstring
said why: "so `agdevworld/assistant`, which proxies them at
`/api/autolab/<node>/…`, keeps working against an empty node". That caller is
now gone twice over, so the stub is gone: with it `status_document`,
`projects_document`, `STUB_LOG`, `send_text`, the `SAFE_NAME`/`ITER_NAME`
validators and the `project_settings` import.

**What was not deleted, and why.** `POST /window` and `GET /healthz` stay.
`/window` runs the `front` role for real and writes a run record;
`/healthz` is what the deployment's readiness probe reads
(`ansible_agdev/roles/autolab_gateway`). A dead browser caller does not make
a whole service dead — the plan says so, and here it was literally true:
the same file held six dead routes and one live entrance. The systemd unit,
the role and `autolab_node_gateway_port` are unchanged.

## Disposable pre-refactor work, retired

The deleted `projects` tab is what listed autolab's local projects, so the
sweep went through them. Retired through the system's own paths:

- **Seven `work-<Plane label>` channels** — `work-g-11`, `work-g-15`,
  `work-g-21`, `work-s4-7`, `work-s4-9`, `work-s4-11`, `work-s4-13`. Each
  held exactly one resolved `workrun-task1-…` topic and is named after a
  Plane label that stopped naming anything when Plane was deleted. No
  `Mission` exists behind them, so neither `zulip_listener.archive_work_channel`
  nor `worklog` could reach them; they were archived directly as the
  Developer. No adapter for the old naming existed or was written.
- **Three finished verification projects** — `refactorp1`, `refactorp2`,
  `refactorp3`, one per phase of the episode that just ended, through
  `python -m agautolab.project_archive`. Their Gitea repos, channels,
  `work-m<id>` channels, folders and local workspaces are retired. The
  reports in `devdocs/episodes/refactor/p{1,2,3}/` are the evidence and were
  kept.

The agent room's sweep went from **20 channels to 10**.

`pj-ghtrends`, `pj-mediagen`, `pj-papers`, `pj-studyarxiv` and
`pj-studyrealworld` were left alone — `studyrealworld` still has open work,
and none of the others is a spent fixture.

## A bug the retirement found

Archiving all three projects failed at the last step:

```
Zulip folder archive failed for pj-refactorp1: PATCH channel_folders/14 ->
HTTP 400: You need to remove all the channels from this folder to archive it.
```

The folder *was* empty as far as the code could see. **An archived channel
keeps its `folder_id`, and `GET /streams` does not return archived
channels** — so `archive_zulip_folder`'s guard saw nothing filed, tried, and
got a 400 it could not explain. Before today it never even got that far: a
live `work-` channel made it return `kept` and stop, which is why the failure
had never surfaced. Six orphaned folders had accumulated — `pj-runsmoke1`,
`pj-runsmoke2`, `pj-simpleshooter`, `pj-foodchain`, `pj-rtnotes`,
`pj-studynourl`, one per project archived before today, every one of them
still in the channel picker.

Fixed in two places:

- `agag.zulip` — `channels(include_archived=True)` (`exclude_archived=false`,
  verified against this realm's Zulip), and a new `clear_channel_folder`,
  the counterpart of `set_channel_folder`. An archived channel does accept
  that PATCH.
- `agautolab.project_archive.archive_zulip_folder` — counts archived channels
  when deciding `kept`, takes every filed channel out of the folder first,
  then archives it.

All six orphaned folders were then retired through the fixed path, plus the
three from this step. The channel picker holds no folder without channels.

## Verified

| check | result |
|---|---|
| `tsc --noEmit` | clean |
| `npm run build` | built, 5 chunks |
| `agautolab` suite | 237 passed |
| `pyagag` suite | 515 passed |
| five-view cycle in a real browser | `nodes → workspaces → agentroom → ops → routines → nodes`, screenshots in `.local/shots-ex1/` |
| `performance.getEntriesByType('resource')` filtered for `/api/` | `[]` — on the cycle and again after opening a detail popup |
| retained detail view | clicking `agautolab1` renders intent, hardware, diffs and raw facts |

The gateway's own test now asserts the shape rather than the stub: `/status`,
`/log`, `/jobs`, `/projects`, `/game`, `/monitor` and `/guide` all 404, and
`/healthz` and `POST /window` answer.

## Not done here

`agecho-agautolab1` is the other fixture agent found in the sweep. Its
retirement is step 4's, where the plan puts it beside its drift gap, so it
was left standing rather than half-retired from two places.
