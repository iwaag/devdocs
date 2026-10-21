# argue p4 ex2 — Step 3: the list and the room entry (aghangout)

aghangout `c71c1b5` and `c66bd02` (from `6210d30`), on WFE `8485dcd` plus
`5d6f511` (list rows name their `newest` message, so the client can tell
unread without the transcript). Step 4 deploys them.

## Before and after

All screenshots are in [shots/](shots/).

- **Before**: the deployed system on 2026-10-21.
  - [lobby](shots/before-desktop-lobby.png): a three-column grid over the
    artwork, titles cut at one line, last-message previews, the 12-row
    limit;
  - [narrow lobby](shots/before-narrow-lobby.png);
  - [room](shots/before-desktop-room-argue-p4-trial.png): opens on a
    line-by-line player, with the purpose not stated.
- **After**: the fixture server's synthetic discussions. Step 4 adds
  deployed shots.
  - [list](shots/after-fixture-desktop-list.png) and
    [list with status words](shots/after-fixture-desktop-list-status.png);
  - [narrow list](shots/after-fixture-narrow-list.png);
  - [entry](shots/after-fixture-desktop-entry.png) and
    [narrow entry](shots/after-fixture-narrow-entry.png).

## What changed

### The list (`src/discussionList.ts`, `src/reading.ts`)

- **Surface**: the partner cards stay in the lobby artwork (`.lobby-hero`).
  The discussions sit below on an opaque surface. The purple stripe on each
  item is gone.
- **Layout**: one column. On phones the partner cards shrink to name and
  face, so the list starts on the first screen after scrolling past them.
- **Row order**:
  1. **Title**: the display title, else the original. It is never
     shortened and wraps freely; the full original title and the id are
     in 詳細.
  2. **Summary line**:
     - `結論` (the overview's outcome) or `目的` (its purpose), with
       citations removed in the list;
     - else `依頼文の抜粋` (the brief);
     - else 「概要はまだありません。」.

     It is clamped to 2 lines (3 on phones), and the full text is at the
     top of the room. A stale overview adds 「ここまでの要約・以降に発言N件」.
  3. **Status words**: 進行中/終了, あなたの返答待ち, `<名前>の返答待ち`,
     作業中 N件, 未完了の作業 N件, 失敗 N件 and 停止中の作業 N件.
     - Colour only repeats the words.
     - 「あなたの」 appears only when WFE's `awaiting.you` is true.
  4. **Metadata**: project participants (not the conductor), 発言 N and
     更新 …前.
  5. **Action**:
     - 返答する when the person is awaited;
     - 続きを読む when this browser has shown less than `newest`;
     - else 議論を読む.

  The whole row is one link (`?discussion=<id>`), so it works with Enter,
  middle-click and open-in-new-tab.
- **Views and filters**:
  - 議論 (default) and 動作確認 (with a note saying what checks are);
  - 進行中 / 終了 / すべて, each with its count from WFE's `counts`;
  - 20 rows, then 「さらに表示（残りN件）」.
- **Kept place**: view, filter and row count are in the address
  (`?list=check&state=open&n=40`) and in sessionStorage with the scroll
  position.
  - Browser back and the room's 「‹ 議論一覧」 restore all of them.
  - The list component survives the lobby's 15-second refresh. Before,
    the refresh rebuilt the lobby and lost the scroll position.
- **Reads**: one list read per view change and refresh, page only. No
  transcript is read, and no model is called.

### The room entry (`src/meeting.ts`)

- **Default**: the reading view. Focus goes to the overview's heading. The
  overview shows:
  - title and status words, plus a 動作確認 marker for checks;
  - **目的 / 結論 / 未解決**, where every `[#n]` is a button that scrolls
    to, highlights and focuses message n in the conversation (one action
    to the original). A missing outcome is said as such;
  - who wrote it, when, how far it covers, and that it is an
    interpretation, not speech;
  - without an overview: 「概要（目的・結論）はまだ書かれていません。以下は依頼文の抜粋で、結論ではありません。」,
    the brief, and a link to its message;
  - **次に必要なこと** from WFE's status: whose answer (with 返信する for
    the person), work in progress, closed but still owed, failures and
    stopped runs. Otherwise it says WFE records nothing awaited or owed;
  - 「最新の発言へ」 and 「前回以降の発言 N件」.
- **会話（原文）**: every message, verbatim, under the participant WFE
  recorded, with its agent, kind label (要約（解釈）/質問/結果…), what it
  answers and 返信する. Lines since the last visit are marked, and a
  question to the person is marked あなたへの質問です. New messages are
  appended without disturbing the reader's place.
- **再生表示**: the previous visual novel (seats, dialogue box, ← →),
  unchanged in behaviour. The choice is kept in localStorage.
- **Panels**:
  - 進行 / 招待 / メンバー / 記録 as before. The old *History* panel is
    removed, because the reading view replaces it;
  - **詳細**: original title, id, creator, conductor, requester and
    participants, plus the category and display title. A correction needs
    a reason and calls `POST …/meta` only. The change history and the
    overview revisions are listed.
- **Composer**: To:, reply-to, drafts, outbox with Retry/Edit and the IME
  guard are unchanged in logic. The labels are now Japanese (宛先, 全員,
  送信, 「#n への返信」).
- **New discussions** can be created as 動作確認.

## Checks

### Tests and checks

- **`npm run check`**: `tsc`, the vite build and 19 unit tests.
  - 6 are new in `test/reading.test.ts`: the list state round-trip;
    display versus original title; citation parsing and stripping;
    outcome/purpose/brief/empty and stale coverage; "あなたの" only for
    the person, with agent requesters named and closed rooms still owing;
    the action word.
- **`npm run browser-check`**: 127 checks, all passed in both **Chromium**
  and **WebKit**. Logs are in
  `.local/argue-p4-ex2/step3/browser-{chromium,webkit}.log`.

### Existing flows, kept

- posting, replies, questions, the failed post plus reload plus Retry as
  one message, drafts, IME, partner switching, an unavailable partner and
  WFE being down;
- in discussions:
  - setup that requires a title;
  - the facilitator, participants and synthesis, now read in the
    transcript;
  - the player with seats, the speaker and ←;
  - addressed posts, invitations, records and progress, where a decision
    source opens the message.

### New reading checks

| Area | Checks |
| --- | --- |
| Surface and layout | no artwork and an opaque background behind the list; one column; title → summary → status → meta order; nothing essential hidden in a `title` tooltip |
| Titles | an 80-character Japanese title is complete, takes a second line on desktop and three or more on a phone, never ellipsised; `fx-similar-1/2` keep 1回目 and 2回目 visible |
| Overview | an outcome without citations in the list; a labelled brief; 「概要はまだありません」; stale 「以降に発言3件」 in the list and in the room |
| Status | あなたの返答待ち plus 返答する; Frontの返答待ち never "あなたの"; 失敗 1件; 終了 plus 未完了の作業 1件; the conductor not listed |
| Views | counts; the open filter shows only open rooms; checks only in 動作確認; 20 rows then さらに表示 reaching all 36 to 37 rows, so nothing is lost after the twelfth |
| Navigation | a row opens with Enter; focus on the title; the filter, scroll position and back button restore the list's place |
| Sources | a citation focuses the cited message (English original under a Japanese overview); a decision's source opens from 進行 |
| Participation | 返信する from 次に必要なこと names the question and focuses the composer; the question is marked in the conversation |
| Metadata | a correction without a reason is refused in the page; with one it is recorded. **Fixture writes**: posts and invitations unchanged, meta +1 |
| Keyboard | Tab (WebKit: Option+Tab, Safari's link navigation) reaches the rows with a visible focus outline; filters work with Enter and keep focus |
| Contrast | 4.5:1 or more on every list text, labels and status words (8.7–15.4:1), and on the room's overview metadata, message text and headers |
| Touch | view, filter and new-discussion controls are at least 40 px on a phone |
| Narrow (390 px) | no sideways scroll in the list, entry or room; the reading area sits above the composer |
| 200% zoom (640 px wide) | no sideways scroll in the lobby or room; long titles wrap; the overview is readable |

### Platform note

WebKit does not focus a button on click, and Safari's Tab skips links. The
checks therefore drive filters with the keyboard (focus plus Enter) and
use Option+Tab on WebKit. These are platform conventions, not a defect of
the page.
