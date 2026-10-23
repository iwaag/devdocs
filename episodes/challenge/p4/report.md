# Challenge p4 — final report: titles from the person's wish

**A person starts a conversation by writing what they want and pressing
Send. Nobody asks them to name it.**

- The wish's first line becomes the title at once, so the room is
  recognizable in the list straight away.
- When ponderer leads the room, its normal first turn gives a short
  Japanese display title.
- If a person has already titled the room, or reset the title to the
  original, their choice stays.

## Review it (from any LAN computer)

1. Open `http://agstudio.local:8093/hangout/` and press
   「＋ 複数のプロジェクトで話す」. There is no title field, and the cursor
   is already in the message box.
   - Write a wish and send it, and the room shows its first line as the
     title.
   - Leave 「Pondererに進行を頼む」 on, and within about a minute the
     heading changes to ponderer's Japanese title.
2. To see the result without sending anything, open the 動作確認 view
   (`?list=check&state=all`):
   - `?conv=20261023-225438-sjbp91`: the original title was 「家族で週末に
     遊べる、30分くらいで終わる短いボードゲームの案を一緒に考えたい」.
     ponderer named it 「家族で遊ぶ30分の短いボードゲーム案」. 詳細 shows
     both, with the history (ponderer-agent, run-023).
   - `?conv=20261023-225540-6hcyrl`: the person titled it 「人が先に付けた題
     （p4確認）」 before ponderer answered. ponderer's attempt to title it
     was skipped, and the history has only the person's record.
3. From an agent: `wfe conversation help` (`--if-untitled` under
   `meta`/`overview`).

## Releases

| Component | Release |
| --- | --- |
| aghangout | `5e92078` (previous `4175292`) |
| WFE | agdevworld `61e547a` (code `3516748`, tests `36f7f5f`/`61e547a`; previous `a252289`; no store change) |
| ponderer guide | pj-ponderer `380442b` |
| agpc worker | WFE `61e547a`, executor `1b65813` (nctl) |
| pj-agdev | `25314c7` |

## Steps

| Step | Report | Outcome |
| --- | --- | --- |
| 1 aghangout | [report1](report1.md) | No title field, composer first. The title comes from the first nonempty line, grapheme-safe, and is fixed in the outbox for retries. The challenge entry point no longer pre-fills a title. Chromium/WebKit 340/340 |
| 2 Conductor titles | [report2](report2.md) | `--if-untitled` / `ifUntitled`: checked and written in one write transaction. A person's edit or reset wins, a skip never blocks the overview, and duplicate or later overviews rename nothing. ponderer's guide fact. WFE 540/540 |
| 3 Verification | [report3](report3.md) | The plan's behaviour matrix, plus an unavailable-facilitator test |
| 4 Deployment | [report4](report4.md) | Cutover, agpc re-pinned, drift unchanged. LAN check 24/24 on studio and 5/5 on agpc. Archive manifests match |

## Completion criteria

| Criterion | Status |
| --- | --- |
| Starting a discussion requires no human title input | **Met.** There is no title field, on studio and on agpc. The browser checks cover new rooms, group rooms and the challenge entry point. |
| Every submitted wish produces a recognizable fallback title immediately | **Met.** The first nonempty line, readable 3.5 s after Send in the deployed check. Long, multiline, Japanese and emoji text are covered. Blank words create nothing. |
| A normal first conductor turn supplies a useful Japanese display title through the existing metadata mechanism | **Met.** run-023 gave 「家族で遊ぶ30分の短いボードゲーム案」 as a `discussion_meta` record. The same title is in the room, the list, WFE Home and the archive. |
| Manual title changes survive automatic completion and retries | **Met.** Live (room B) and in tests: an edit from another connection, a reset to the original, duplicate and later overviews. A retry after a failed first post keeps one room and one message. |
| The deployed LAN interface demonstrates it, with checks and recovery recorded | **Met.** report4. Recovery is the release switch alone, since the store is unchanged. |

## Limitations

- A room nobody leads (one-to-one, or several projects without ponderer)
  keeps its first-line title until a person edits it in 詳細.
- The display title arrives only when ponderer's first turn runs. If that
  turn fails or is delayed, the original title stays and remains usable.
- Old local drafts of a title (`<id>.title`) are no longer read.
