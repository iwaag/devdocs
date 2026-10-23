# Challenge p4 — step 3: verification before deployment

The checks run against fixtures and temporary stores. The deployed
behaviour (ponderer's real first turn, the LAN entrance, Home and the
archive) is verified in step 4. WFE `61e547a` adds one focused test.
aghangout `5e92078` is unchanged since step 1.

## Required checks

| Project | Check | Result |
| --- | --- | --- |
| aghangout | `npm run check` (tsc, vite build, unit tests) | 31/31 |
| aghangout | `npm run browser-check` | Chromium 340/340, WebKit 340/340 |
| WFE | `npm run check` (tsc, every test, both builds) | 540/540 at `3516748`; `discussionReading.test.ts` 9/9 at `61e547a` |

## Behaviour matrix

| Plan item | Where it is checked | Result |
| --- | --- | --- |
| Titleless submission, focus in the composer | browser: *several projects* (no `.title-input`, `activeElement` is the composer, focus and words kept through 7 s of polling) | ✔ |
| Leading blank lines, multiline Japanese, full-width space | unit `titleFrom`; browser: 「会議室の見た目を どうしますか？」 from `\n\n  会議室の見た目を　  どうしますか？  \n…`; message stored as written | ✔ |
| Long text and Unicode truncation | unit: 100 kana → 60 graphemes + `…`; ZWJ emoji within 80 units, never split; exactly 60 kept. WFE `firstLine`: the same at 80 units | ✔ |
| Empty or whitespace-only words do not submit | browser: blank words create no room and leave no outbox entry; unit: `titleFrom` of blanks is `''` | ✔ |
| Failure after creation, before delivery, with attachments | browser *challenge entry*: room created, post fails, reload, 再試行 → one room, one message as written with ch-0002, one response request, the same title | ✔ |
| Challenge entry point | the same check: no pre-filled title; the title comes from the wish | ✔ |
| One-to-one unchanged | the browser one-to-one checks are unchanged and pass (WFE derives the title from the opening message) | ✔ |
| Conditional naming: untitled room named once | WFE `ifUntitled titles an untitled conversation once…` | ✔ |
| Manual edit before completion (another connection) | same test: the edit is kept, the overview is still published | ✔ |
| Manual edit during completion | the check reads inside the same `BEGIN IMMEDIATE` transaction as the write, so an edit is serialized before (kept, skip) or after (a later ordinary change). The test covers the "before" order across connections | ✔ by construction and test |
| Intentional reset to the original | same test (value null record wins); CLI test after `--no-title` | ✔ |
| Duplicate overview submission | same test: `duplicate: true`, nothing renamed | ✔ |
| Later overview updates | same test: a new revision is published, the title unchanged | ✔ |
| Title changes start no run | the counts (messages, requests, jobs, `updated`, state) are unchanged after automatic and manual title writes | ✔ |
| Unavailable facilitation keeps the room usable | new WFE test: no `conduct` workflow → the conductor request `failed`, the room is readable at once under its original title, and a later manual title starts nothing | ✔ |
| A skipped title does not block the overview | WFE tests and CLI output `Display title left as it is: …` after `Recorded: overview 3` | ✔ |
| Invalid title refused before writing | a two-line title publishes no overview; `ifUntitled` without a title is a 400 | ✔ |

## Left for the deployed check (step 4)

- the fallback title in the deployed room immediately after Send;
- ponderer's normal first turn giving the Japanese display title through
  `--if-untitled`;
- the same display title in the room, the aghangout list, WFE Home and the
  pj-ponderer archive manifest;
- the same from another LAN PC.
