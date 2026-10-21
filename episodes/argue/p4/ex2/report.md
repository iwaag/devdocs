# argue p4 ex2 — readable discussion discovery and entry: final report

The Developer's screenshot showed a discussion list that discouraged
reading:

- truncated titles;
- technical last-message previews;
- equally loud cards;
- operational trials mixed in with ordinary topics.

Discussions should now be easy to choose, understand and join. The same
clarity should continue on the first screen inside a discussion.

**Delivered and deployed**:

- WFE agdevworld `5d6f511` (store v9);
- aghangout `c66bd02`;
- ponderer's guides `612a7b6`;
- the historical data classified and given overviews.

Everything was verified from agpc.

Step reports:

- [report1](report1.md): contracts, screens, fixtures, migration map;
- [report2](report2.md): WFE store, interfaces, migration, tests;
- [report3](report3.md): the list and room entry, browser checks,
  before/after;
- [report4](report4.md): deployment, the bounded check, LAN evidence,
  reading tasks, recovery.

## Outcomes

| Completion criterion | Outcome |
| --- | --- |
| Readable titles and purpose or outcome in discovery; checks in a separate, reachable view | One-column list on a plain surface. Each row shows its display title or full original title (wrapping, never cut), then 結論 or 目的 from the overview (or a labelled 依頼文の抜粋), then status in words, participants and time, then the action. 議論 is the default view; 動作確認 holds the 7 checks |
| Every discussion discoverable; 12 is no boundary | Both views have open/closed filters and counts, and 20 rows per step with 「さらに表示」. The list's place survives a visit to a room (address plus scroll). The fixture's 37 rows are all reachable. WFE pages the list server-side |
| Overview attributable, source-linked, explicit about coverage; missing or stale never shown as current | WFE refuses an overview part without a `[#n]` source, or with a source outside its coverage. Author, run, coverage and note are recorded, and the archive holds every revision. Readers see 「ここまでの要約・以降に発言N件」 and 「この概要は…による解釈で、発言の原文ではありません」. A missing overview shows the brief, labelled as the request. Every `[#n]` opens the original message in one action |
| Reading and metadata trigger no agent work; status reflects WFE evidence, including closed rooms with outstanding work | Metadata and overview writes touch no message, invitation, run, state or ordering (tests; fixture write counters; live counts 247/70/575 unchanged by the migration). Status comes from obligations, open questions with their receiver, failed invitations and held runs. 「あなたの返答待ち」 appears only when the receiver is the person, and an agent requester is named. A closed room says 「未完了の作業 N件」 |
| The room explains purpose, known outcome and the next recorded need, with direct access to speech and participation | Overview-first rooms: 目的, 結論, 未解決 and 次に必要なこと (with 返信する), then the whole verbatim conversation and the unchanged composer. The visual novel is 「再生表示」. 進行, 招待, メンバー, 記録 and 詳細 remain as panels |
| Desktop, narrow, keyboard, Japanese text; WFE and aghangout tests | WFE 488/488. aghangout 19 unit tests and 127 browser checks in both Chromium and WebKit, covering filters, paging, overflow, coverage, sources, status, metadata, keyboard, contrast (all ≥ 4.5:1), touch targets, 390 px and 200% zoom |
| Deployed, checked from another LAN machine, reviewable; docs, guides, pins; no duplicate legacy path | Deployed. agpc passed 36/36. Review links are in [report4](report4.md#review-links-from-any-lan-machine). README_DEV, WFE `docs/discussions.md`, the aghangout README, ponderer's guides and the pj-agdev pin are updated. The last-message preview, the 12-row slice, the History panel and the grid are removed, not kept beside the new paths |

### The bounded check (`argue-p4-ex2-check`, category `check`)

- forge answered through its ordinary discussion run.
- ponderer published a Japanese, source-linked overview **without being
  asked** to. The guide fact was enough.
- It did not announce the conclusion its own decision said it was making.
  That is recorded as an annotation (see Limitations).

## Objective checks and unmeasured claims

**Checked**: on the deployed system, from agpc, the five reading tasks
were each answered from the first screen, with no panel opened and no
truncated text:

- the topic;
- its purpose;
- its outcome;
- what remains unresolved;
- where to reply.

On the old screen, the topic was cut at one line and indistinguishable
from two sibling cards. Purpose and unresolved points were absent, and
the outcome appeared only as a last-message fragment.

**Not measured**: whether the Developer now finds the UI more inviting.
No user study was done, and this report does not claim it.

## Remaining limitations

- **Overviews go stale without a prompt.** Nothing revises an overview
  automatically, by design: no model runs on reads, and there is no
  mandatory per-message synthesis.
  - Readers see how much came after, and ponderer's guide says when to
    revise.
  - Whether ponderer keeps them current across longer discussions is
    observed only in one short check.
- **Six historical overviews are Omni's, not ponderer's.** They are
  attributed to `Omni Agent (for Developer)`, and nothing marks them
  further.
  > Deus Ex Machina note: did the historical category classification and
  > the initial overviews and display titles of six discussions for agent
  > ponderer — handoff candidate.
- **The check's conclusion was not announced.** In `argue-p4-ex2-check`,
  ponderer recorded "concluding" as a decision and wrote the outcome into
  the overview, but posted no conclusion. This echoes p4's "intention not
  carried forward" pattern. It is recorded as annotation 3 and was not
  fixed here.
- **Partly English panels.** The panels' inner texts (進行, 招待, 記録
  details) and the personal-conversation rooms are still largely English.
  The labels and everything on the reading path are Japanese.
- **Few rows to page.** With 6 ordinary and 7 check discussions on the
  live system, 「さらに表示」 is exercised only on the fixture (37 rows),
  not live. The live paging route was verified through the API from agpc.
- **No stored reference screenshot.** The Developer's screenshot is not in
  the repository. The "before" comparison uses my capture of the same
  deployed screen.

## Review links

- The list: <http://agstudio.local:8093/hangout/>
- Checks view: <http://agstudio.local:8093/hangout/?list=check>
- An entry (outcome and unresolved):
  <http://agstudio.local:8093/hangout/?discussion=argue-p4-trial>
- Sources across languages:
  <http://agstudio.local:8093/hangout/?discussion=p16-s4-provenance>
- The bounded check:
  <http://agstudio.local:8093/hangout/?discussion=argue-p4-ex2-check>
- Screenshots: [shots/](shots/), with before, after-fixture,
  after-deployed and agpc captures.
