# Challenge p2 — Step 5: one room in aghangout

aghangout `2596519` adds the room; `9604695` adds only its guide line (step
4). The work was done in a separate clone so that it did not collide with
aghangout-agent's live workspace.

## What changed

- **One Room** (`app/src/room.ts`) replaces the chat room (`room.ts`) and the
  meeting room (`meeting.ts`, deleted). There is one message list, one
  composer, one panel set, one challenge panel and one conversation list.
  - **Shape is presentation only.**
    - One-to-one: the partner's room, the partner's portrait, the player
      first.
    - Several projects, or led by ponderer: the members at the table and the
      page to read first (overview, verbatim transcript, character dialogue,
      workflow cards).
    - The room changing shape (a member joins, ponderer starts leading)
      redraws the stage. The draft, attachments, outbox, place and work stay
      as they were.
  - **Who receives the words is shown before sending**, as WFE routes them:
    the participants chosen, the run whose question is being answered,
    ponderer while it leads, else the default addressee. With nobody, the
    composer says 宛先を選んでください and does not send.
  - **Direct actions:**
    - ＋ 参加 invites a project; joining starts nothing.
    - 「ponderer に進行を頼む」 makes the next words the brief of an explicit
      facilitation start.
    - 進行を終える stops facilitation; no work stops.
    - 再試行 on a failed request tries the same recipient again.
    - 再開 runs a held continuation.
  - **Panels:** 進行, 依頼 (requests, with role and workflow), 作業 (runs and
    where they open), メンバー (default addressee, leading, close and
    reopen), 会話 (the partner's other conversations), 記録 (archive state
    and path), 詳細, 課題, ワークフロー.
  - The transcript marks where ponderer began leading, and from which
    message.
- **The lobby:**
  - The partners are the projects WFE offers one-to-one, each card with its
    newest one-to-one conversation.
  - One list (会話 / 動作確認) covers every conversation and shows each
    row's shape.
  - 「＋ 複数のプロジェクトで話す」 opens a new room with title, members and
    ponderer leading (on by default).
- **Addresses.** `?conv=<id>` is the room. `?with=<project>` is that
  partner's one-to-one room. An old `?discussion=<id>` link resolves to
  `?conv=<id>`, and citations `<id> #n` keep identifying their records.
- **Client and fixture:**
  - `api.ts` uses only `api/conversations`;
  - `conversation.ts` holds pure helpers, including whom words reach,
    invitable projects and waiting requests;
  - the fixture emulates the unified contract.

## Checks

- `npm run check` passes: build, plus 28 unit tests.
- Browser checks: **285 pass in Chromium and in WebKit**, with WFE's graph
  renderer. Without the renderer, 281 pass.
- New checks:
  - joining keeps the id, draft and attachment, and changes shape without
    losing a message;
  - asking ponderer to lead starts facilitation with the brief (heading,
    進行役 preview, transcript marker, synthesis); ending it returns the
    words to the default addressee;
  - with no default addressee, sending is disabled until someone is chosen;
  - a held continuation's 再開 calls resume and clears it;
  - an old `?discussion=` link resolves.
- Three bugs found by the checks and fixed:
  1. A new one-to-one room's first message, sent before the first read
     returned, lacked `with` and was refused.
  2. The status line (retry, resume) was hidden in the read view.
  3. ponderer was listed twice on a led room's row.

## Where it stands (deployed 2026-10-23 17:22 JST)

The cutover was done with the classifier's permission; step 7 records it in
full. Deployed:

| Component | Version |
| --- | --- |
| WFE | `21a094e` (store v12; the migration matched the rehearsal: no request pending) |
| `conversations.json` | new shape (ponderer offered for one-to-one) |
| aghangout | `9604695` |
| Slack | `21a094e` (`with: front`) |
| guides | agautolab `ba9e2c1`; pj-front `66492de`, pj-forge `415b1a8`, pj-archsage `31c8908`, pj-aghangout `9604695`, pj-ponderer `ef7ac65` |
| executor and Slack | running again |

The 10 direct trial conversations were classified `check` by "Omni Agent
(challenge p2)". The listing counts conversations 16 and checks 46.

**Not done yet** (step 6 onward, paused here at the Developer's request):

- the deployed end-to-end check with a named test actor and a Challenge
  attachment;
- confirming the archive backfill of the direct conversations in
  pj-ponderer;
- the agpc worker pins;
- the LAN check from another computer;
- step reports 6–7, `report.md` and README_DEV.

The Developer can look now at `http://agstudio.local:8093/hangout/`.
