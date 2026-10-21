# argue p4 ex3 — Step 3: embedded graph interaction

Delivered in two commits:

- aghangout `00f599f`;
- WFE agdevworld `cc156f3`, a small change to the embedded graph on top
  of Step 2's `024aaaf`.

Neither is deployed yet; that is Step 4.

## What the reader sees

**Cards in the conversation** (`src/meeting.ts`; the logic is the pure
`src/workflows.ts`):

- **Under a message WFE names.** The card shows:
  - a relation word: 参照, 作成, 編集 or 実行;
  - the workflow's name, `project/workflow`, and whether it is a 定義 or
    a ラン（実行）;
  - the version: 下書き（未コミット）, コミット `abc1234` or 改訂 rN;
  - the change in words, built from WFE's comparison only:
    「結果レビュー」を追加、「画像を作る」の説明を変更; counts when there
    are many; 配置・見た目のみの変更 when nothing behavioural changed;
  - a small picture of the graph, with the marked nodes outlined in a
    pattern.

  A creation adds （公開・実行はされていません）.
- **An operation whose turn spoke but named no message** sits with that
  turn's first message, under 「このターンの操作（どの発言についてかは記録されていません）」.
  It is never attached to a message as if WFE had named it.
- **A turn that said nothing** gets its own row, 「発言のない操作」, with
  the run, attempt, invitation and time. It is placed in record order and
  is never made into a message.
- **The conductor's progression** folds into 「進行の更新 N件（…の進め方の記録）」,
  which unfolds to its records.
- **Several records under one message** are listed in order.
- **A version WFE could not read** says so on its card, in Japanese:
  「この版は表示できません（下書きの内容が記録されていません）」.

**One shared panel** (`src/workflowPanel.ts`; the ワークフロー tab, with a
count):

- It opens on **このターン時点** for the version WFE recorded. **現在** is
  a separate read. It is labelled 「現在（…時点に読んだもの。この議論で記録された版ではありません）」,
  and it says whether the target still has the recorded bytes.
- An edit has **変更後** and **変更前**:
  - added and changed nodes are marked ＋追加 and ～変更 (dashed and
    dotted outlines);
  - added connections are dashed;
  - the before view marks －削除 (struck through) and the removed
    connections;
  - the change list names each node, each property's before → after,
    each connection and the workflow fields.

  Differences are words and patterns, never colour alone.
- The source line reads 出典: the run, the invitation, the time and
  「#n へ移動」, which scrolls to the message and focuses it. Without a
  message it says 「発言との対応は記録されていません」.
- **State.** A run shows the state at the recorded point, as 「記録時点（ランの記録 n 番目、…）の状態」.
  A definition shows 「定義のため実行状態はありません」. 現在 shows the
  current state, labelled as such.
- **Nodes.** A node list (the accessible equivalent and the fallback)
  and node details. The details list only the messages WFE recorded
  about that node: one spoken for that node of that run, or named by a
  record that singles the node out.
- 「エディタで開く（現在の内容）」 opens the full editor as a secondary
  action.
- **The graph** is WFE's renderer, `/wfe/embed/graph.js`, imported on
  first use and drawn into a shadow root. If it cannot load, the panel
  says 「グラフを表示できませんでした…」 and the text lists carry
  everything.

**Behaviour:**

- Opening a card keeps the unsent draft. The chosen card stays where it
  was on screen while the panel takes its column.
- Another card switches the graph and the selection (`aria-current`,
  「表示中」).
- Enter on a card opens the panel and focuses it. Escape or ✕ closes it
  and returns focus to that card.
- A redraw (the data arriving, a toggle) keeps the keyboard's place.
- On a phone the panel is a full-screen opaque sheet.
- Reading never writes.

## How the WFE renderer is reused

| | |
| --- | --- |
| Boundary | `mountGraph(host, {workflow, effective, states, marks, edgeMarks, focus, selected, onSelect})` → `{update, select, reveal, fit, dispose}` |
| What is reused | the editor's `Canvas`, layout, icons, run-state overlay and stylesheet, unchanged except for the optional `marks` and `edgeMarks` (the editor never sets them) |
| Isolation | a shadow root, with `:root` → `:host`. No editor style reaches aghangout and no aghangout style reaches the graph. The editor's frame hint is hidden in the embed; Fit and zoom stay |
| What it does not have | no API client, navigation, editing or state of its own. Move and connect are no-ops, and dragging is disabled by `readonly` |
| Instances | one per room, switched between records with `update` |
| Large graphs | the initial frame centres on the marked nodes when the graph does not fit at a readable scale (Canvas `focus`), with −/+/Fit, wheel and pinch |

## Evidence

**Tests.**

| Run | Result |
| --- | --- |
| `npm run check` (typecheck, build, unit tests) | 24/24 (19 before; 5 new in `test/workflows.test.ts`: placement never by time; summaries from the comparison only; marks in words; recorded messages per node; versions never "published") |
| `npm run browser-check`, Chromium, WFE renderer served | **181/181** (127 before) |
| the same in WebKit | **181/181** |
| Chromium without the renderer (text fallback) | 177/177 |

Logs are in the ignored `pj-agdev/.local/argue-p4-ex3/step3/`.

**What the workflow checks prove** (fixture `fx-workflows`):

- **Placement:**
  - by WFE's record only;
  - a turn's operation with no named message is said as such;
  - a turn that said nothing is its own row, between the messages before
    and after it, with its run, attempt and invitation;
  - progression is folded;
  - three cards share one message.
- **Labels:**
  - a creation is a draft, never published or executed;
  - an edit names 「結果レビュー」を追加;
  - a layout-only edit says so;
  - an unavailable version says so;
  - card text contrast is ≥ 4.5:1.
- **The panel:**
  - it opens beside the conversation, at このターン時点・変更後;
  - the change list has properties before and after, and the removed
    connection;
  - the graph shows ＋ 追加 and ～ 変更 badges and two added edges;
  - the before view has no added node and one struck removed edge;
  - the draft and the card's screen position are unchanged.
- **Switching and nodes.** Another card switches the panel and the
  selection. A singled-out node shows its recorded message; another node
  claims none. The source message is one action away.
- **Execution.** The run's state is shown at the recorded point (seq 1),
  with no message recorded.
- **Unavailable versions.** Said, with no substitute: no graph and no
  node list.
- **Large graphs.** 40 nodes, all listed as text. Long names wrap. The
  first view centres on a node the record singles out, with zoom
  controls.
- **Pinned after later changes.** After `POST /fixture/workflow-later`
  and a reload, the edit still shows its recorded version (no
  「公開する」). 現在 shows it, labelled 「この議論で記録された版ではありません」
  and 「記録された版から変わっています」. このターン時点 returns.
- **Keyboard.** Enter opens; Escape closes and returns focus to the
  card. WebKit found a lost focus after the asynchronous redraw, now
  fixed.
- **Renderer missing.** The text lists remain.
- **No writes.** Posts, invitations and metadata writes are unchanged
  across all of it, so graph interaction triggers no edit and no
  dispatch.
- **Narrow, 390 px.** No sideways scroll. Cards and the progression line
  are ≥ 40 px tall. The panel is a fixed full-screen sheet and closes
  with ✕.
- **200% zoom.** Cards and the open panel need no sideways scroll.

## Screenshots (fixture)

- [cards in the conversation](shots/fixture-chromium-desktop-wf-cards.png)
- [edit, after, with WFE's graph](shots/fixture-chromium-desktop-wf-panel.png),
  and in [WebKit](shots/fixture-webkit-desktop-wf-panel.png)
- [edit, before (removed connection)](shots/fixture-chromium-desktop-wf-before.png)
- [現在 after a later change](shots/fixture-chromium-desktop-wf-current.png)
- [40-node graph, focused on the singled-out nodes](shots/fixture-chromium-desktop-wf-large.png)
- [renderer unavailable: text fallback](shots/fixture-webkit-desktop-wf-nograph.png)
- [phone: cards](shots/fixture-chromium-narrow-wf-cards.png) and
  [the sheet](shots/fixture-chromium-narrow-wf-panel.png)
- [200% zoom](shots/fixture-chromium-zoom200-wf-panel.png)

## Known limits after this step

- **English inside the graph.** The type labels inside the graph's
  cards are the editor's own vocabulary (DO, TALK, STUDY), as are the
  zoom control's "Fit" and WFE's raw reasons for a state that cannot
  be replayed. Every other label on the reading path is Japanese.
- **The fixture is a stand-in.** Its records are written by hand; the
  live WFE records and their comparison are exercised in Step 4.
- **The 再生表示 (player)** shows no workflow cards. The cards belong to
  the reading view.
