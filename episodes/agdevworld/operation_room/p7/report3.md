# Step 3 — dashboard: session = topic

`routineState.ts` follows the Step 2 payload: `RoutineRow.latest_topic`,
`runs`, `open_runs` replace `fire_topic`; `RoutineSession` gains `topic`,
`stamp`, `previous`, `answer`, `history` and `chat`, loses `start_id`,
`end_id` and `note`; `SessionResolution.state` is `resolved | open`;
`SessionHistory` is `runs`, `open_runs`, `session_limit`, `deep_runs`,
`hidden_resolved`, `note`; the detail's `chat_log` is gone and
`latest_topic` is beside `latest_fire`; `startSession` returns `topic` and
`previous`.

Session cards are titled by the run's stamp (`Run 2026-09-07T05:00Z`, the
topic name as tooltip), carry the fire id and age, origin, the run's answer
state and linked-conversation count, a resolution chip that is only ever
`✔ resolved` or `○ open` while the relay is live (`? unknown (relay)` when
it is not — relay health keeps the word, sessions lose it), a window
annotation when the run's history is bounded, and the fire's own
`↤ previous run <stamp>` pointer: a link that selects that run when it is
listed, plain text saying why otherwise. The history line under the cards
states runs held, open runs, hidden resolved and the relay's deep-read rule.

The chat panel shows the selected run's topic whole (`ChatPanelHandle.show`
replaces `highlight`; there is no span to infer), names the topic in its
header and posts into it. A ✔'d run is shown but not written into: the
composer is disabled and the note says Front does not sweep a resolved
topic, so un-resolve it in Zulip or start a new session. The graph's root
card is the run topic. New session opens a run topic: the pending card is
`Run <stamp>` from the topic the relay returned, the result line names the
topic and the previous run it points to, and the pending card is replaced
when the event queue lists the topic. `detailPopup.ts` shows the latest run
and run counts in place of the fire topic. `worldViews.ts` needed no change:
the unmanaged panel shows the newest run on `update`.

Verified over CDP on the dev server (`:5173`) against the live relay with
the run list injected through `fetch` (no run topic exists on the realm
until Step 4) — `.local/p7/step3/a.js`, screenshots `sessions.png` and
`narrow.png` (390 px): run titles and chips, no `unknown` on a live card,
the bounded annotation, the history line, the chat header/posts/composer on
an open run, the graph root, the previous-run link selecting the older
resolved run with the composer refused and the reason shown, host
observation latest-only on the older run, Show resolved off hiding the
resolved run with selection falling back and the empty state naming the
filter in both the list and the chat, and a mocked start producing the
pending card, the result line and the listed run. `npm run build` passes
with the existing Phaser chunk advisory. The web image is not rebuilt yet
(Step 4).
