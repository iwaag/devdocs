# argue p4 ex2 — Step 1: contracts, screens and migration map

## Starting point

### Revisions and state

| Component | Revision inspected | Where |
| --- | --- | --- |
| WFE (agdevworld) | `153bdad`, store v8 | `pj-agdev/agdevworld/workflow_editor` (`server/discussions.ts`, `server/discussionArchive.ts`, `server/api.ts`, `cli/discussion.ts`, `docs/discussions.md`) |
| aghangout | `6210d30` (served) | `.local/agdev/pj-aghangout/app/src` (`lobby.ts`, `meeting.ts`, `discussion.ts`, `api.ts`) |
| ponderer | `5d85cd1` | `.local/agdev/pj-ponderer/agent/{guide,conduct,synthesize}.md` |

- Nautobot/nctl: I refreshed the agstudio observation (`nctl observe
  agstudio`, operation `01M5EG4TZ5HBBFN1HVF62V1JKD`, success). `nctl drift
  --host agstudio` then reported converged.
- WFE holds 12 discussions: 2 open and 10 closed. None has an open
  invitation.
- Before screenshots are in `pj-agdev/.local/argue-p4-ex2/step1/shots/`
  (ignored): the lobby, plus rooms `argue-p4-trial` and
  `p16-s4-provenance`, at 1400×900 and 390×844.

### What the code does today

The plan's evidence holds, with these details:

- **Lobby list** (`lobby.ts`):
  - `d.discussions.slice(0, 12)` in a three-column grid over the lobby
    artwork;
  - each card has a purple left border and shows, in order: the title (one
    line, ellipsis), state, member names, counts and relative time, then
    `<speaker>: <first line of the last message, 90 chars>`;
  - the 12 stored rows exactly fill the grid. Discussion 13 would not
    appear.
- **List API** (`GET api/discussions`):
  - returns every discussion, each with `last` (300 chars of the newest
    message), `activity` (latest waiting or latest activity), `open`
    (open invitations) and archive status;
  - there is no category, no overview and no person-facing status.
  - `archive.status()` runs `git remote get-url` per row, which is cheap
    but repeated.
- **Room** (`meeting.ts`):
  - opens on a visual-novel line player positioned at the reader's last
    seen line;
  - a new reader of a closed room therefore starts at message 1 and steps
    through ◀ ▶. The conclusion is only reached by playback or through
    History;
  - the title is in the header, and the purpose is never stated;
  - Progress, Invitations, Members and Records are panels;
  - the conductor (ponderer) occupies a seat and appears in every member
    list.
- **Status sources that already exist in WFE**:
  - `invitations` (open, failed);
  - `asking` (open questions; `openAsking` checks each WFE question's own
    record);
  - `activities` (waiting asks and talks);
  - `obligations()`: open invitations and awaited results, questions with
    owner and receiver, runs still working (with `situation`), and open
    delegations. It works for closed rooms too ("recorded only: the
    discussion is closed").
  - The talk activity records its addressee as the requester: either a
    person id or a run.

### Existing interpretation and provenance conventions reused here

- **Synthesis messages** (`kind: synthesis`) and `summary.md` in
  pj-ponderer cite `#<message>`. The summary also states the archive
  commit and horizon it rests on (`agent/synthesize.md`).
- **Annotations** are records outside the speech, with an author, `re`
  and an optional run, and are archived as their own record type.
- **The archive** keeps append-only transcript records, a manifest, and a
  horizon per record type. A new record type needs a horizon field and a
  `covers()` term.

The overview follows these conventions:

- it is a WFE record, not a message;
- every statement cites messages;
- it records its author and run, the message horizon it covers and its
  time;
- it is archived as its own record type with its own horizon.

Reading a list never needs Git or a model call.

## Screen specification

### List (lobby section 「議論」)

The surface is quiet and opaque: there is no artwork behind the reading
list and no per-item colored stripe. The partner cards above it stay as
they are, because they belong to the personal conversations.

```text
議論                                                      [＋ 新しい議論]
[ 議論 6 ] [ 動作確認 6 ]        表示: (進行中 2) (終了 4) (すべて 6)

┌──────────────────────────────────────────────────────────────────────┐
│ forgeが作った素材の来歴はどこに置くか                                 │  ← title, up to 2 lines (wraps)
│ 結論: forgeの納品記録を正とし、配布側リポジトリは参照のみ [#145]       │  ← overview outcome or purpose, 2 lines
│ ここまでの要約・以降に発言2件                                         │  ← coverage (only when stale)
│ ● 進行中 · あなたの返答待ち                                            │  ← status (text first, color second)
│ forge, archsage · 発言7 · 16時間前                     [議論を読む →] │  ← metadata, action
└──────────────────────────────────────────────────────────────────────┘
                    [ さらに表示（残り N 件） ]
```

- **Order within a row**: title, then overview, then status, then
  metadata, then the action.
  - The whole row is one link to `?discussion=<id>&from=<list state>`.
  - The button text 「議論を読む」 becomes 「続きを読む」 when the room has
    unread messages for this browser, and 「返答する」 when the person is
    awaited.
- **Title**:
  - `displayTitle` when one is recorded, else the original title;
  - it is never shortened by string rules;
  - it can use two lines at desktop widths and wraps further on narrow
    screens;
  - the full original title, with the id, is in the room's 詳細.
- **Overview line**: the first available of:
  1. the overview outcome (結論), then its purpose (目的);
  2. 依頼文の抜粋 (a labeled excerpt of the brief);
  3. 「概要はまだありません」.
- **Status labels**, derived from WFE (see *Status* below):
  - room: 進行中 / 終了;
  - 作業中 (a run or invitation is working);
  - あなたの返答待ち (the receiver is the configured person);
  - `<name>の返答待ち` (an agent requester or another person);
  - 未完了の作業 N件 (a closed room that still owes something);
  - 失敗 N件.
- **Metadata**:
  - domain participants only; the conductor is shown in the room's 詳細;
  - message count;
  - update time.
- **Views**:
  - 議論 (the default) and 動作確認, each with its count;
  - within a view: 進行中 / 終了 / すべて (default すべて), with counts;
  - 20 rows, then 「さらに表示」;
  - view, filter, the number of rows shown and the scroll position are
    kept in the address (`?list=check&state=open&n=40`) and in
    sessionStorage, so returning from a room restores them.

### Room entry

```text
‹ 議論一覧   forgeが作った素材の来歴はどこに置くか        ● 進行中  [詳細]
┌ 概要 ─────────────────────────────────────────────────────────────────┐
│ 目的  forgeが複数プロジェクト向けに作る素材の来歴を…  [#140]          │
│ 結論  …                                                  [#145 #147]  │
│ 未解決 ・…                                                    [#146]  │
│ 次に必要なこと  あなたの返答待ち（#146 archsageの質問）  [返答する]    │  ← from WFE, not from the overview
│ 要約: ponderer-agent · 2026-10-21 · #147までを要約 · 以降に発言2件     │
└───────────────────────────────────────────────────────────────────────┘
 会話（原文）                                                             ← all messages, scrollable, newest at the bottom
   #140 Omni Agent (for Developer) → 全員 …
   …
 [ To: 全員 | forge | archsage ]  [ 返信先 #146 ✕ ]
 [ メッセージ…                                               ] [送信]
 その他: 進行 | 招待 | メンバー | 記録 | 再生表示
```

- **The overview block** shows:
  - purpose, outcome, unresolved points and the next recorded need;
  - every `[#n]` citation links to that message in the conversation,
    which is highlighted and focused;
  - each citation is one action to the original text;
  - stale coverage is a visible line;
  - with no overview, it shows the labeled brief excerpt, and no
    conclusion is ever invented.
- **The conversation** opens as the full transcript (original speech,
  verbatim), scrolled to the newest message. Syntheses and announcements
  carry their labels.
- **Character presentation stays available as 「再生表示」**: the current
  seats and line player behind a toggle that the browser remembers.
  Nothing about speakers or attribution changes.
- **The composer** (To:, reply-to, drafts, outbox, IME) is unchanged in
  behaviour and is reachable by one Tab from the overview.
- **詳細** shows:
  - original title and id;
  - category (with correction);
  - display title (with correction);
  - conductor and requester;
  - members.

### Screen fixtures (synthetic, aghangout fixture server)

All ids start with `fx-` and all titles carry the word 「(fixture)」 or
「フィクスチャ」. They exist only in `app/fixture/server.ts`.

| Fixture | Exercises |
| --- | --- |
| `fx-long-ja` | an 80-character Japanese title, with no spaces, wrapping on two lines on desktop and four on a phone; overview current |
| `fx-mixed` | an English title, a Japanese overview citing English messages (`[#n]` opens the English original) |
| `fx-similar-1`, `fx-similar-2` | two titles that share their first 30 characters and differ only at the end (1回目/2回目) |
| `fx-no-overview` | no overview: 「依頼文の抜粋」 from the first person message |
| `fx-empty` | no overview and no message: the honest empty state |
| `fx-stale` | overview through #n, three messages after it: 「ここまでの要約・以降に発言3件」 |
| `fx-await-you` | a talk question to the configured person: 「あなたの返答待ち」 and 「返答する」 |
| `fx-await-agent` | a talk question to an agent requester (front run): 「Frontの返答待ち」, never 「あなたの」 |
| `fx-failed` | a failed invitation in an open room: 「失敗 1件」 with WFE's reason |
| `fx-closed-owed` | a closed room with a working delegated run: 「終了 · 未完了の作業 1件」 |
| `fx-check-*` | three operational checks: only in 動作確認 |
| `fx-old-01…25` | 25 older closed discussions: rows 21–25 appear after 「さらに表示」 |

## Data contracts (WFE)

### Category and display title: store v9

- `conversations.category`, either `discussion` or `check`. A discussion
  gets `discussion` when nothing is said.
- `conversations.display_title`, either null or a short title for
  readers.
- `discussion_meta`: an append-only history of changes (`id, conv, field,
  value, by, run, at, note`), for the archive and for review.
- **Writes**:
  - creation: `wfe discussion create --category check`, or the
    `category` field of `POST api/discussions`;
  - correction: `wfe discussion meta <id> [--category …] [--title …|--no-title] --note …`
    or `POST api/discussions/<id>/meta {category?, displayTitle?, note?}`.
- **Metadata writes touch only these fields and their history.** They
  never:
  - change `updated`, so the list order stays put;
  - write a message;
  - change an invitation;
  - reopen anything.

### Overview: the `overviews` table, append-only revisions

```text
overviews(id, conv, author, run, purpose, outcome, unresolved JSON[string],
          through INTEGER, sources JSON[int], note, at, token UNIQUE(conv, token))
```

- **Citations** are written inline as `[#n]`. WFE parses them and
  validates that:
  - every cited message is in the discussion and is `≤ through`;
  - the purpose cites at least one message;
  - the outcome and each unresolved point cite at least one message each.

  `sources` is the union of the citations.
- **`through`** defaults to the newest message, and must be a message of
  the discussion.
- **The current overview** is the latest revision. Earlier revisions stay,
  in the store and in the archive.
- **Writers**:
  - from an executing attempt, a run of the facilitator's or the archive
    project's workspace (ponderer); its agent and run are recorded;
  - outside an attempt, any `--author` (the person, Omni). Omni-written
    ones carry that author.
- **Interface**: `wfe discussion overview [<id>] --purpose … [--outcome …]
  [--unresolved …]… [--through <msg>] [--title …] --note …`,
  `--show`, `--file <json>`; `POST api/discussions/<id>/overview`.
- It never starts a run or sends a message.

### Archive

- New transcript record types are `meta` and `overview`, with horizon
  fields `meta` and `overview`.
- The manifest gains `category`, `displayTitle` and `overview` (the
  current one).
- A pre-v9 manifest's missing horizons count as 0, as `activity` and
  `annotation` did in v7.

### List read model: `GET api/discussions`

- **Query**: `?category=discussion|check&state=open|closed|all&limit=&offset=`.
  Without `category`, every discussion is returned (WFE Home).
- **Response**: `counts {discussion:{open,closed}, check:{open,closed}}`,
  `total` (rows matching the filter), and each row with:
  - `category`, `displayTitle`;
  - `overview`, compact: `{purpose, outcome, unresolved, author, run, at,
    through, after}`, where `after` is the number of messages newer than
    `through`;
  - `brief`, when there is no overview: `{message, author, text≤400}`,
    taken from the first person message, else the first announce;
  - `status` (below);
  - `participants`: members minus the person and the facilitator.
- Heavy fields (`status`, overview, brief) are computed only for the rows
  of the requested page, and nothing is fetched per transcript by the
  client. `last` is removed: the client no longer previews the last
  message.

### Status (derived from WFE's records, shared by list and room)

```text
status: {
  room: open|closed,
  working: n,      // obligations of kind invitation/result/run/delegation still open
  awaiting: [{ message, question, receiver: {kind: person|run, id, name}, you: bool }],
  failed: [{ invitation, participant, error }],   // failed invitations not later answered
  owed: n,         // all obligations (questions included)
}
```

- `awaiting` comes from `openAsking`. The receiver is:
  - for a talk activity's question, the requester (a person id or a run);
  - for any other open question, the configured person.
- `you` is true only when the receiver is a person whose id is the
  configured person.
- A room that is open with `working = 0` is "進行中" without "作業中". A
  closed room with `owed > 0` says "未完了の作業".

## Affected components

| Owner | Change |
| --- | --- |
| **WFE** | store v9 (category, display title, meta history, overviews); `create --category`; `meta` and `overview` CLI and HTTP; list filtering and paging, counts, status, overview and brief; room view `overview`, `status`, `category`, `displayTitle`; archive record types and manifest fields; `docs/discussions.md`; tests; Home card uses `displayTitle` and lists ordinary discussions |
| **aghangout** | lobby: discussion list replaced (single column, views, filters, paging, restored state); room: overview header, transcript view as default, player behind 「再生表示」, details with category and title correction, Japanese labels; fixture server and browser checks; README |
| **ponderer** | `agent/guide.md` (the tool, the overview as an interpretation it owns) and `agent/conduct.md` / `synthesize.md`: when an overview is written. The overview is not a step every discussion must pass. |
| **automated creators** | Omni's fixture scripts use `--category check`. No in-system creator makes fixtures today; Front creates through ponderer, and its discussions are ordinary. |

## Migration map (to be applied in Step 2)

The evidence is each discussion's opening brief (`step1/discussions/*.txt`)
and the episode reports. A prefix is only a hint.

| id | Category | Evidence | Display title (recorded explicitly) |
| --- | --- | --- | --- |
| `p16-s3-room` | discussion | substantive design question (#124, meeting-room scene) asked for forge's and archsage's views; used as a p16 trial | aghangoutの会議室シーンをどう作るか |
| `p16-s4-provenance` | discussion | substantive rule question (#140); its conclusion became forge's `knowledge/asset-provenance.md` (p4 ex1) | forgeが作った素材の来歴はどこに置くか |
| `argue-p3-trial` | discussion | the Developer's experiment goal, verbatim (#166); one match played | RTSにJevを取り入れたAI対戦実験（1回目・実行委員会方式） |
| `argue-p3-ex1-trial` | discussion | same goal, guided by Omni hints (#181) | RTSにJevを取り入れたAI対戦実験（2回目・Omniの助言つき） |
| `argue-p4-trial` | discussion | same goal, conducted by ponderer (#234) | RTSにJevを取り入れたAI対戦実験（3回目・ponderer進行） |
| `argue-p4-ex1-trial` | discussion | a substantive choice (three sources to read next), agreed protocol (#243) | 議論研究から次に読む3件を選ぶ |
| `p3-s2-fixture` | check | "[System check by the Omni Agent, not a request of the Developer]" (#161) | — |
| `p4-s3-check` | check | "[Check — … a small discussion, not an experiment]" (#208) | — |
| `p4-s5-a` | check | "[Fixture — … a check of the system, not an experiment]" (#214) | — |
| `archsage-topic-line` | check | argue p4 report5: fixture for the requesting-agent path, closed by Omni after the check | — |
| `forge-media-gap` | check | argue p4 report5: rerun of the same fixture path | — |
| `20261020-233458-r93d5v` | check | "[Trial …] A one-line check" from agpc (#157), the LAN check of p16 step 6; continued in p4 ex1 as a cutover check | — |

- I found no uncertain case. If one had been uncertain, it would have
  stayed `discussion` until its purpose was established.
- Checks keep their original titles; their prefixes already say what they
  are.
- **Overviews**: I prepare them in Step 2 for the six ordinary discussions
  from their transcripts, as Omni (author `Omni Agent (for Developer)`).
  I record the handoff note there.

### Browsing all historical rows

- the 議論 and 動作確認 views with すべて, plus 「さらに表示」 until the end;
- `wfe discussion list [--category …] [--state …]`;
- the archive in pj-ponderer (`discussions/`).

Nothing is dropped after row 12.
