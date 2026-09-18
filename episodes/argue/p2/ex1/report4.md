# Step 4 — deployment, the live exercise, and the human session

Date: 2026-09-18 JST (14:02–14:25Z). Plan: [plan.md](plan.md) step 4.

## Deployment (agstudio)

- **State**: `nctl drift` → `converged=46`, 0 diffs, before and after. No
  desired-state change: no new account, no new service.
- **Web image** rebuilt (`docker compose up --build -d web`), `:8090` → 200;
  the served `frontDesk-*.js` chunk carries the new strings.
- **Relay** `com.agdev.agentroom` restarted with `launchctl kickstart -k` on
  agdevworld `69f07e9`: mirror `live (resumed queue)`, no resync;
  `/argues/7217/close-plan` answers on :8094 with `zulip_calls: 0`.
- **No listener change**: Front's optional "let's stop here" was not done.
  agfront stays on p2's code (pyagag `7beeec5`). agautolab1 (the VM) untouched.
- **Pins**: pj-agdev `0e45161` points at agdevworld `69f07e9`.

## Suites

| Suite | Result |
|---|---|
| agentroom `uv run pytest -q` | **301 passed** (295 + 6 new) |
| agdevworld `tsc && vite build` | clean |
| agfront | not touched, not run |

## Live exercise on the deployed relay (Omni Agent, not the human)

`#argue › argue-p2ex1-close-smoke` (anchor 7259), opened through
`POST /argues` as the Developer and labelled as a check that invites nobody.

1. Front served it once (run `argue/0018`, $0.065) and its reply was
   rendered automatically (`present/0011`, $0.043).
2. **Close from the room's door**: `close-plan` → one action, `ready`;
   `POST /argues/7259/close` with that fingerprint → `applied`, the topic is
   `✔`, confirmed by the mirror, **nothing written** into the argue.
3. **Front's listener did not stir on the ✔.** Its log between the close
   (14:19:08Z) and the resume (14:20:56Z) holds only the renderer's job for
   the earlier reply; no `serving`, no run record. The Notification Bot's
   "resolved" line is a system notice (pyagag `0a33830`), the move into the
   `✔ ` name is ignored by intake — both as reasoned in step 1, now observed.
4. **Resume**: one post through `/argues/7259/post` → `resumed: true`; the
   listener served it at once (`argue/0019`, $0.053; then `present/0012`,
   $0.040); the deployed room showed `answered · being rendered…` with the
   paid button dimmed under `renderer: rendering 1 post…` (`s4-b`).
5. Closed again the same way: `applied`, `✔`.

Cost of the exercise: **$0.20** (2 argue runs $0.12, 2 renders $0.08).

## The human session

**Pending for the human.** What this exercise can say instead:

- **A human-led argue already happened this morning, on p2's code**,
  before this exercise — `argue-20260918-124701` (12:47–13:15 JST, the
  Developer, in Japanese, 13 posts). It went the whole way: desire recorded,
  archsage's council, a study opened (`pj-worldtrend`), a sage consulted, a
  correction from the human folded in, and a "thank you" at the end. That is
  the session p2 left pending; it is done, and it produced exactly the three
  problems this exercise's braindump lists (see below).
- What is left for a human on the *new* code is the plan's list: open
  `/?view=argue`, start an argue, read a reply in both views, re-voice once
  through the confirm, reply from the composer (Japanese IME, Shift+Enter,
  a long draft, a wheel over it), close it with `finish ✔`, resume it with a
  post. The room is deployed at `http://localhost:8090/?view=argue`.
  `argue-20260918-124701` is still open and can be the one closed.

### Cost of the morning session (run JSONs)

| Role | Runs | Cost |
|---|---:|---:|
| Front `argue` (Sonnet 5) | 7 | $0.74 |
| Front `present` (Sonnet 5) | 8 | $0.53 |
| archsage council (Fable 5.1) | 1 | $0.96 |
| `sage:worldtrend` | 1 | $0.13 |
| autolab `superdirector` (setup) | 1 | $0.46 |
| **Total** | 18 | **$2.83** |

Presentation was 19 % of the session. Two of the eight renders were 38 s
apart (13:08:03 and 13:08:41), and the argue carries **two `[render]`
requests and one failed rendering**: the human pressed `reinterpret ⟳`
twice on a resolved-then-resumed argue — the braindump's item 2, observed
in the record.

### Observations p2 asked for

- **Front's pace**: Front replied 11–20 s after each human post; the
  council answered 2.5 min later. The human's second post came 15 min after
  the first, the third 4 min after Front's fifth reply — the human was never
  behind; Front was waiting.
- **Courtesy**: the closing "ありがとう、良いargueだったよ" bought a Front run
  ($0.088) that said the argue was over — and could not end it. Front had
  resolved the argue itself at 7241 (outcome: study); the human's correction
  at 7247 un-resolved it; Front then said "nothing left open" and left it
  open. Step 1's door is the answer to that shape.
- **Preface**: Front's replies in the morning session carry no smoke-test
  preface (there was nothing to preface); the smoke argue's replies do, as in
  p2, because the opening post said so.
- **Un-resolve on post** is also what a human's *correction* looks like:
  a post after Front's outcome. That is the intended behaviour (the
  discussion resumed; the study stayed), and the human closed nothing
  afterwards because they could not.
