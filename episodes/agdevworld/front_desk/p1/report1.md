# Step 1 — the Front Desk scene and its input

`/?view=frontdesk` in agdevworld, linked from the dashboard toolbar. A
dedicated `FrontDeskScene` on its own Phaser game (agdevworld `516fc97`):
not an eighth `PanelGridScene` view, because there is no grid, no ⇄ cycle and
no chat overlay — the frame is the whole window.

## What is on screen

- **Background** `bg.png` fills the frame (cover, centred); **portrait**
  `agfront.jpg` stands in the lower left with its aspect ratio preserved,
  sized from the frame. Both are the supplied images, unchanged, copied to
  `public/frontdesk/`.
- **Dialogue** beside the portrait: the speaker, the latest reply, a
  `1/3` page label with ◀ ▶ (wheel over the box turns pages too), and link
  chips for every URL the reply carries plus "open in Zulip" when the relay
  supplies the topic URL.
- **Prompt bar** along the bottom: the draft, a grapheme counter against the
  relay's limit, `Send ⏎ · buys a run`. Enter sends; Shift+Enter is a newline.
- **History** panel on the right, toggled by a button (Escape closes it):
  every post with sender and time, acks as a dim "Front received it" line,
  wheel-scrolled, with chips for the recent conversations and a "new
  conversation" button. The draft lives in the textarea and is untouched by
  opening, scrolling or closing the panel.
- **Status** in the dialogue header: `posting…`, `sent · waiting for Front`,
  `📩 Front received it and is working`, `answered 3s ago`, `✔ resolved`,
  `✖ not sent — <reason>`, `? uncertain — <note>`; and a caption line that
  turns amber `UNKNOWN — <reason>; last known history` when the relay cannot
  be read, with the bar disabled and the last history kept.

`src/frontDeskState.ts` carries the relay contract step 3 implements
(`GET /frontdesk`, `GET /frontdesk/<id>`, `POST /frontdesk/<id>/post` with a
per-submit token) and a `&demo=1` source that answers from a script in the
Front Desk voice, so the frame could be checked without a run or a relay.

## The two input findings

**Phaser's word wrap breaks emoji and Japanese.** `Text.advancedWordWrap`
splits an over-long "word" with `split('')`, per UTF-16 unit. A Japanese
sentence is one word to it (no spaces) and an emoji is two units, so a reply
in this voice is exactly the text it halves. `src/textLayout.ts` wraps with
`Intl.Segmenter` — words where the language has them, graphemes when a word
is wider than a line, never inside a grapheme, with the common kinsoku
characters kept off line starts — and hands Phaser explicit lines with wrap
off. Pages are line-chunks of that.

**The composition-confirming Enter is the event's word, not ours.** The first
version kept a `composing` flag from `compositionstart`/`compositionend`. A
CDP probe (`Input.imeSetComposition` then Enter) never delivered
`compositionend`, and the flag would have blocked every later send. Now the
keydown handler trusts `isComposing` / keyCode 229, and the flag is refreshed
from each `InputEvent.isComposing` instead.

## Checked in the browser (headless Chrome over CDP, `.local/deskshot.mjs`)

| check | result |
|---|---|
| typed `やっほー！今日もよろしく🙋‍♀️✨ test`, Enter | sent; bar cleared; ack shown as status, reply arrived in the dialogue |
| composition `にほんご` + Enter | **not** sent, draft kept; the next real Enter sent |
| synthetic keydown Enter with `isComposing` / keyCode 229 | nothing sent |
| long reply (≈330 graphemes, 5 emoji clusters incl. ZWJ, 2 markdown links) | one page at 1600×1000, three pages at 900×600, no split emoji, three link chips |
| history open / scroll / close with a draft in the bar | draft `下書きを保持` intact |
| 1600×1000 → 900×600 → 1280×800 | relaid out; portrait aspect kept; chips wrap to two rows |
| relay without the route (today's) | amber UNKNOWN caption, `chat is not available right now`, history panel says history unknown |

Screenshots are in agdevworld `.local/shots/frontdesk/` (ignored).

## Left for later steps

- The relay routes named above do not exist yet (step 3); against today's
  relay the scene correctly reports unknown.
- Animation and responsive polish were left discretionary and minimal: no
  tweens, one layout function.
