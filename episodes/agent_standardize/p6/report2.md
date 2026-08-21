# agent_standardize p6 — Step 2 report: the special route is gone

AI-generated (Omni Agent, 2026-08-21).

agautolab `66f3971`. Deleted, not stubbed and not deprecated — there is no
compatibility path from the marker route to the new one, which is what the
plan asked for.

## What the route was

An asset request used to be a *different kind of thing* from a task. The
superdirector wrote `[Asset]` at the end of a `task[N].md`; registration
stripped the marker and turned it into an `asset` Plane label; the
`workrun-` serving read that label back off the issue and, before any coding
run, entered a state machine — look up agforge's Work in Plane under
agforge's own external source, post an order into an `assetplan-asset_<id>`
topic if there was none, report "in progress" and stop if there was, and only
when agforge had moved its Work to `completed` scrape the `[S3KEY]` footer
out of the topic, call `POST /api/resign` on agforge's HTTP service for a
fresh presigned URL, and append that URL to the coding run's guide. In
parallel, a second handler answered agforge's questions: a mention gate on
`assetplan-` topics, a Plane read that recomposed `plan.md` and `task.md`,
and a dedicated `assetplan_answer_superdirector` role.

Six couplings in that description are the reason it had to go: autolab knew
agforge's name, its external source, its topic naming, its delivery footer,
its HTTP endpoint, and its mention protocol — all compiled in, none of it
learned from what agforge itself says.

## What was deleted

`src/agautolab/mission.py`

- `ASSET_LABEL`, `ASSET_MARKER`, `AGFORGE_SOURCE`, `ASSET_TOPIC_PREFIX`
- `strip_asset_marker`
- `Work.is_asset` and its two label reads (in `next_work` and `run_target`),
  `TaskChange.is_asset` and the `[asset]` suffix on the reconcile line
- the whole "asset ledger" section: `asset_topic`, `asset_order_key`,
  `asset_order`, `asset_answer_context`
- the `labels=[ASSET_LABEL] if is_asset else None` argument at Sub-Work
  creation

`src/agautolab/zulip_listener.py`

- `ASSETPLAN_TOPIC_PREFIX` and its entry in `SWEEP_PREFIXES` — autolab no
  longer sweeps `assetplan-` topics anywhere, which is what makes forge's
  topics forge's business again
- `AGFORGE_URL`, `S3_KEY_MARKER`, `S3_KEY_LINE`, `RESIGN_TIMEOUT_SECONDS`,
  `AESTHETICS_FILE`, `ASSET_PROMPT_NOTE`
- `aesthetics_text`, `asset_order_text`, `find_asset_key`, `resign`,
  `asset_gate`, and the `asset check` step in `serve_run`
- `supercoder_prompt`'s `asset_url` parameter — the guide is passed whole
  again, with nothing appended per work
- `mentions_us`, `answer_prompt`, `run_answer`, `handle_assetplan`, and the
  dispatch branch that routed to it
- the `json`, `urllib.error` and `urllib.request` imports, dead once
  `resign` went, and the four mission imports the deleted code used

`agent/guides/assetplan_answer_superdirector/` — the whole folder, and the
`answer` role it defined. The remaining guides are `autolab-front`,
`bmining_director`, `interview`, `workplan_superdirector`,
`workrun_supercoder`.

The `workplan_superdirector` guide's one asset line ("add `[Asset]` at the
end of file") went with it. That is a deletion, not the Step 3 edit: leaving
a guide that teaches a marker no code reads would be worse than either
state.

`README.md` — the `assetplan-` row of the topic-prefix table, and `create-`
from the list of retired prefixes.

## Tests

`tests/test_mission.py` lost `test_strip_asset_marker_only_matches_the_tail`
(5 parametrized cases), `test_reconcile_labels_an_asset_sub_work`, the two
`next_work` label tests, `test_run_target_reads_the_asset_label_off_the_task`,
and the whole asset-ledger block (`asset_topic`/`asset_order_key` naming,
the three `asset_order` states, the `asset_plane` fixture).

`tests/test_zulip_listener.py` lost the asset state machine block (order,
in-progress, resign-into-prompt, missing-footer failure, the plain-task
negative, the two `find_asset_key` tests) and the whole "answering agforge"
block (`ASSET_TARGET`, `ASSET_CHANNEL`, `ASSET_TOPIC`, `FORGE_ID`,
`create_message`, `wire_create`, the mention-gate tests, the generation
tests, the answer-prompt test). The entrance test stopped patching
`handle_assetplan`.

**178 → 143 tests, all passing.** The 35 that went were the route's own; no
surviving test was weakened or rewritten to keep passing.

## Criterion 3, early

```
$ grep -rn 'agforge\|assetplan\|ASSET_\|\[Asset\]\|asset_gate\|mentions_us' src agent tests
(no output)
```

Clean already, ahead of Step 4's check. Two prose mentions of agforge that
were not routing — `mission.py`'s "the Plane client is shared with agforge"
and `role_run.py`'s "answers agforge's questions" — were reworded rather than
left, because a grep that has to be read with exceptions is not a proof.

The `[AUTO]` project marker and the `AUTO` label stay, per the plan. So does
`FORGEAUTO` on forge's side, which autolab never mentioned anyway.

## What is deliberately broken right now

Between this commit and Step 3, autolab can neither request an asset nor be
asked about one. That is intended: the route was removed before its
replacement exists so that nothing can silently fall back to it. Step 3 gives
the runs `tools/agents.md`, the Zulip identity and the guide lines that make
the ordinary path work.

Nothing is deployed yet — the push is to GitHub only, and the `--limit
agstudio` deploy comes at the end of Step 3 with the run-setup change, so the
live checkout is never left in this half state.
