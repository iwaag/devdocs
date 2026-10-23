# Challenge p4 — step 1: no title before the wish (aghangout)

aghangout `5e92078` (on `main` in Gitea `developer/pj-aghangout`; deployed
in step 4).

## What changed

- **The new-room setup has no title field.** `app/src/room.ts` lost the title
  input, its required check (`aria-invalid`), its focus handling, its draft
  key (`<id>.title`) and its special case in the keyboard handler. The setup
  keeps who joins, 「Pondererに進行を頼む」 and 動作確認として作成.
- **Focus starts in the composer** for any room that does not exist yet.
  Before, a new several-project room focused the title, or the overview's
  heading in the reading view.
- **The instructions invite the wish.**
  - Composer: 「あなたの願いを、あなたの言葉でそのまま…（ponderer が最初に
    読みます。タイトルは自動で付きます）」.
  - Without ponderer: 「あなたの願いや最初の発言を…（タイトルは自動で付きます）」.
  - The empty overview and the empty dialogue explain that the title comes
    from the first line, and that ponderer, when asked to lead, gives a
    clearer one.
- **The original title is derived from the words** (`titleFrom` in
  `app/src/conversation.ts`):
  - the first nonempty line, with whitespace (full-width space included)
    collapsed to one space and trimmed;
  - at most 60 graphemes and 80 UTF-16 units (WFE's original-title limit),
    ending in `…` when shortened;
  - cut at a grapheme boundary (`Intl.Segmenter`), so a kanji, a surrogate
    pair or a ZWJ emoji is never split.

  The message is sent as typed (only the existing outer trim). The title is
  not applied to it.
- **The title is fixed when Send is pressed.** It is stored in the outbox entry
  (`create.title`) with the conversation id and the message token. A retry
  after a failure or a reload creates the same room with the same title, then
  posts the same token. A room that already exists is not created again
  (`exists`). An old outbox entry with an empty title falls back to
  `titleFrom(text)`.
- **Challenge entry point** (「この課題で議論を始める」, `main.ts`): it no
  longer pre-fills a title draft from the challenge. The challenge stays
  attached, and the room is named by the wish like every other room.
- **One-to-one rooms are unchanged.** Their first post names `with`, and WFE
  derives the title from the opening message, as before.
- **WFE's contract is unchanged**: creation still needs a nonempty title, and
  the client supplies it. The fixture now refuses an empty creation title
  as WFE does. It also lists what the page created
  (`GET /fixture/creates`).
- README: the 新しい議論 and challenge entry descriptions.

## Checks

- `npm run check` (tsc, vite build, unit tests): 31/31. Two new tests cover
  `titleFrom`:
  - leading blank lines, a tab and a full-width space, multiline text;
  - whitespace-only input gives `''`;
  - 100 hiragana give 60 graphemes ending in `…`;
  - repeated ZWJ family emoji fit 80 units with no lone surrogate and no
    broken sequence;
  - exactly 60 characters stay whole.
- `npm run browser-check`: **Chromium 340/340, WebKit 340/340**. Changed
  and added checks:
  - *challenge entry*: no title input. The room is created, then the first
    post fails (`/fixture/fail`). The room exists with no message. After a
    reload the wish waits in the outbox. 再試行 gives **one** room
    (`/fixture/creates`), **one** message exactly as written (two lines),
    carrying ch-0002, **one** response request, and the title
    "Can someone take this on?". The room is led by ponderer.
  - *several projects*:
    - no title input, and focus is in the composer;
    - the composer keeps focus and typed words across 7 s of polling (the
      old IME regression check, moved from the title to the composer);
    - whitespace-only words send nothing and create nothing;
    - a wish with leading blank lines, a full-width space and three lines
      gets the title 「会議室の見た目を どうしますか？」 and is stored as
      written;
    - the heading shows the title at once, and the lobby lists it first.
  - *nobody by default*: the room is titled by its first words.

The first run found one regression: the reading view focused the overview's
heading in a new room. `focus()` now goes to the composer whenever the room
does not exist yet.
