# Challenge p2 — Step 6: verifying migration and behavior

The deployed releases are WFE agdevworld `21a094e` (store v12) and aghangout
`9604695`. Step 5 records them; step 7 records the cutover.

## Repository checks

| Check | Result |
| --- | --- |
| WFE `tsc` + `npm test` (agdevworld `21a094e`) | 533 pass, 0 fail |
| aghangout `npm run check` (`2596519`; `9604695` changes only the guide) | build and 28 unit tests pass |
| aghangout browser checks (step 5) | 285 pass in Chromium and WebKit |
| Slack bridge (step 2) | 7/7 |

The scenario list of plan step 6 maps to WFE tests (report2, *Checks*) and to
archive tests (report3, *Checks*). The migration was rehearsed on a copy of
the live store (report2, report3). Its result matched the rehearsal: no
request was left pending and nothing was replayed.

## Deployed end-to-end check

The check was driven through the real aghangout and WFE. It wrote as the
explicitly named test actor **"Omni Agent (challenge p2 check)"** in the check
room `20261023-083618-chp2ck` (category `check`, now closed). Evidence and the
script are in the ignored local notes (`.local/challenge-p2/step6/`).

| Scenario | Observed |
| --- | --- |
| One-to-one message, no conductor | The composer shows → Forge. The message carries ch-0001 pinned at `8176fd6`. One `converse` request (165) went to forge with receipt `conversation:<id>:request:165`, and no facilitation was created. |
| Pinned challenge in the run input | `forge/converse/run-009` `input.attachments[0]` = ch-0001 at the same commit. Forge replied naming it. |
| Repeated submission | The same token posted twice gave the same message #317 and the same request 166 (`duplicate: true`). |
| A member joins | archsage joined through ＋ 参加. The id was kept and no message was lost. No request was made and no job was started. The draft survived, and the default addressee stayed forge. |
| Asking ponderer to lead | 「Pondererに進行を頼む」 made the next words (#320, with ch-0001) the facilitation brief in the same conversation. Earlier messages, request 165 and its run were kept. `ponderer/conduct/run-022` received ch-0001 in its input and announced the challenge id, citing forge's earlier answer (#318). It asked nobody, as the brief said. |
| Ending facilitation | 進行を終える recorded the facilitation as `ended`. The composer returned to → Forge, and the conductor run was not cancelled (it completed). |
| Close and archive | The room closed with nothing cancelled. The archive was published at `conversations/20261023-083618-chp2ck` (pj-ponderer `30d2c09d97ef`). |
| Restore from a fresh clone | A new clone of pj-ponderer has manifest `ag.conversation.v1` with path, default addressee and the started-and-ended facilitation. Each of the 6 messages appears exactly once. The first message is verbatim, with its author and pinned ch-0001. The conductor request and the request receipts are present. |
| Lists | The check room is not in the ordinary conversation list. |

The first run stopped after the join because the script toggled the
already-open 課題 panel shut. After the script was fixed, the same room was
resumed from the facilitation phase (`RESUME=<id>`), so the evidence covers
one room end to end.

## LAN check from another computer

agpc (Ubuntu, system Chrome) ran the script read-only (`READ_ONLY=1
CONV=20261023-083618-chp2ck`) against `http://agstudio.local:8093` and passed
**7/7**:

- the facilitation mark is shown;
- the ch-0001 attachments are shown;
- the archive is current at `conversations/<id>`;
- the old `?discussion=20261023-011814-vv689m` link resolves to `?conv=`;
- the check room is absent from the ordinary list;
- the room fits a phone;
- no page errors.

## Archives of every conversation

`wfe conversation archive --status` reports 63 rooms, all `current`. 43 are
under `conversations/`: the 42 backfilled direct rooms plus the check room.
20 are under `discussions/`, unchanged. The backfill of direct history is
therefore published live, not only in the rehearsal.

## Store after the check

- **Requests:** none pending or started. 151 answered, 13 unanswered,
  1 withdrawn, 2 failed.
- **Failed and withdrawn requests:** all predate p2 and involve the retired
  lowerpoly: `p16-s3-room` #15, `p15-s6-unavail` #144, and `p4-s5-a` #55
  (withdrawn, closed room).
- **Facilitations:** 20 active (migrated discussions) and 1 ended (the check
  room).

## Findings

- **Composer during load.** A `?conv=` room opened without a partner shows
  宛先を選んでください until its first read returns, because `recipients()`
  returns `none` while the view is null. It then shows the real route
  (→ Forge). Nothing is sent wrongly: Enter only focuses the recipient
  choice. This is a known limit, not fixed in this phase.
- **agpc browser.** The playwright browser cache on agpc is older than its
  playwright-core. The LAN check runs with `CHROME=/usr/bin/google-chrome`.
