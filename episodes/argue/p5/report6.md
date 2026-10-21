# argue p5 — Step 6: deploy, verify and report

## Deployed

| Component | Revision | How |
| --- | --- | --- |
| WFE (agdevworld) | `35b3474` (from `2eb1976`; `1d77ef1` in between) | `scripts/wfe-release.sh deploy`: the release build ran the full suite, 505/505, healthy. Conversation store unchanged (v10). New dialogue store v2 in pj-ponderer's ignored `.local/` |
| aghangout | `97806f5` (from `96d1b99`) | `scripts/deploy.sh`: `npm run check` passed; release plus `renderer-current` (`dialogue-9`, `sonnet`, characters `aa1e69f`) |
| characters | `aa1e69f` | new managed project, Gitea `developer/pj-characters` |
| pj-agdev | `8abde75` | pins agdevworld `35b3474` |
| `conversations.json` | `dialogue` added | ignored local configuration |

- **Backup.** Before the first deployment, all SQLite stores and JSON
  configuration were backed up (path in
  `pj-agdev/.local/argue-p5/backup-path.txt`).
- **Recovery is code only.** Activate WFE `2eb1976` and aghangout
  `96d1b99`. The dialogue store is disposable. Steps are in
  `pj-agdev/.local/argue-p5/notes.md`.

## Checks

- **WFE.**
  - Full suite: 505/505 at `1d77ef1`, again in each release build.
  - `test/dialogue.test.ts`: 8/8 at `35b3474`, including the lease-holder
    case.
- **aghangout.**
  - `npm run check`: 29 unit tests.
  - `npm run browser-check`: all pass in Chromium on `97806f5`, and in
    WebKit (212) on the UI commit `b3e0e2c`. The UI has not changed since.
- **characters.** `tests/test_characters.py` 7/7, and `tools/characters
  validate`.
- **Boundary, live.**
  - archive publications (`argue-p5-async` current through #255) carry no
    line text;
  - `discussion show --json` carries none;
  - ponderer's run copies have no dialogue folder, and the new forge and
    archsage run copies hold no line;
  - `pj-ponderer` is clean.

## From another LAN PC (agpc, Chrome)

`lan-check-p5.mjs` (ignored, `pj-agdev/.local/argue-p5/step6/`; copied to
agpc `~/p15-check/`). It ran twice:

- **The full run** passed every check except one, whose expectation was
  wrong: it required 9 portraits where 8 is correct.
- **The rerun**, with exact counts and without posting again, passed
  15/15.

Shown from agpc:

- WFE's dialogue reports `dialogue-9` (`sonnet`), characters `aa1e69f`.
- **p16-s3-room**:
  - 14 short Japanese lines; the person's 8 messages as written;
  - portraits on forge's and archsage's 8 lines, initials on the 6 lines
    of argue and aghangout;
  - the 3 workflow cards stay beside the speech;
  - #126 opens its English original in place;
  - replying from it names #126;
  - citations work;
  - English lines contain no Japanese;
  - 原文 shows all 22 messages as written.
- **Generation failure.** Two English lines of `argue-p5-async` had really
  failed: for this check the renderer was pointed at a missing binary,
  three tries each, then the setting was restored. They said so and showed
  their originals. Retry from agpc rendered each in English, #254 in the
  first run and #255 in the second.
- **New message.** In the first run, agpc posted a question to forge in
  `argue-p5-lan` (#256).

  | Event | After posting |
  | --- | --- |
  | forge's reply arrived as written | 10.8 s |
  | it became its short line, without a reload | 6.3 s later |

  The line: 「好むのは、用途と形式が明確な具体的依頼だ。…応えられない点は率直に告げ、需要として記録に残す。」
- **A 390 px phone**: no sideways scrolling, and a tap opens the original.
- No page errors.

Screenshots from agpc: `pj-agdev/.local/argue-p5/step6/agpc-shots/`. The
check discussions `argue-p5-async` and `argue-p5-lan` are closed.

## Service state

- **nctl drift.** After deployment: 46 converged, 2 drifting, identical to
  Step 1.
  - `agobserver` `service_missing`, pre-existing.
  - `cagent-snake-e2e` `service_missing` on agautolab1, present at Step 1
    and unrelated.
  - `aghangout` and `agdevworld-web` are converged.
- **Nautobot.** No change. The dialogue worker runs inside the existing WFE
  service process, and the renderer is a file beside aghangout's release;
  no service, endpoint or placement was added.

## Documentation

- `devdocs/README_DEV.md`: *Character dialogue (`argue` p5)*.
- agdevworld `workflow_editor/docs/dialogue.md` (and a pointer in
  `docs/discussions.md`).
- aghangout:
  - README (the short lines, `cast.json`, `public/cast/`);
  - `renderer/README.md`;
  - the guide's facts.
- pj-characters: README, `docs/format.md`, `HISTORY.md`, the agent guide.
- Local, ignored:
  - `pj-agdev/.local/devenv.md` (*argue p5*);
  - `pj-agdev/.local/argue-p5/notes.md` (revisions, configuration,
    recovery, checks).
