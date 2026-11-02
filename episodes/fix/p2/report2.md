# Step 2: the UI operation discoverable, the status truthful

Date: 2026-11-02 (JST). Owner: managed project `aghangout` (workspace `pj-agdev/.local/agdev/pj-aghangout`).

## What was done (aghangout `COMMIT`)

- **Header.** 会話を終了（作業は止まりません） / 会話を再開 sit in the room header beside the other room actions (`room.ts` `renderActions`), for an open and for a closed room; the members panel points there instead of holding the button. The button's title says what closing does not do; the hint is hidden on phones, where the button keeps a 2.5 rem height.
- **Success from the record.** `roomAct()` calls `POST …/close` or `…/reopen` with the reason `aghangout: 会話の見出しの「会話を終了」` (or 再開), requires WFE's answer to say the new state, then re-reads the room in full. The status line says 会話を終了しました（WFE の記録）, or that the room was already closed, or 終了を記録できませんでした：<reason> with 再試行. Nothing is said as done before the answer.
- **Batch close.** 選んで終了 in the list head (`conversationList.ts`) puts a checkbox on every row and a bar with the count, 選択した会話を終了 and the wording about work. Each chosen id is closed with its own call (`aghangout: 一覧でまとめて終了`); `closeReport()` (`reading.ts`) accounts for every id from WFE's answer — closed, already closed, withdrawn requests, or the failure with its reason. Failed ids stay selected and the button reads 失敗した会話を再試行; closed ids are released, so a retry sends nothing for them. The list is read again afterwards.
- **Words.** Filters read すべて / 未終了 / 終了. `statusWords()` says 未終了 or 終了 first, then the specific indicators as before (あなたの返答待ち, <名前>の返答待ち, 作業中, 失敗, 停止中の作業, 保留中の続き, and for a closed room 未完了の作業), and 待機中 only for an open room with none of them. The header word is 未終了 / 終了. No persisted state was added: everything is read from WFE's `status` and `conversation.state`.
- **API.** `setOpen(id, open, reason?)` returns WFE's answer (`RoomChange`: `already`, `conversation.state`, `withdrawn`, `status`).
- **Fixture and checks.** The fixture answers close/reopen like WFE (state, `already`, withdrawn pending/failed requests, `status`), records every room write with its reason (`/fixture/writes` → `rooms`), and refuses the next N closes of a named room (`POST /fixture/fail-close`). Four rooms `fx-close-*` exist for these checks only.
- README: the room's header, the list's 選んで終了, the status words.

## Checks

PENDING
