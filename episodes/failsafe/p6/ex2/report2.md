# failsafe p6 ex2 — step 2: the proxy's own posting identity

## Posting as the Omni Agent

- **The last refusal that pushed the proxy onto another credential is
  gone** (pyagag `df0b635`).
  - `holds.place` refused a hold whose evidence was the recorder's own
    post.
  - That is right for an agent, whose own words are not a person's hold.
  - It is wrong for a proxy, whose words *are* the decision. A proxy
    recording its own words now holds, as itself.
  - An ordinary agent is still refused.
- **Acceptance.** The Omni Agent can record on its own credential, with its
  own post or in person, and the record says `for 8 (Developer)`.
- **Operator instructions** (`pj-agdev/.local/devenv.md`):
  - The backfill note "Front's credential, because the Omni Agent's own
    credential is refused on its own stand-in posts" now says to use the
    Omni Agent's own credential and post.
  - It also says never to switch to `developer.env` to get past an
    authority refusal.
- **The Omni Agent's own harness memory** is updated to match:
  - `mission-acceptance-is-a-record`;
  - `front-confirm-rule-stand-in`.
- **Human UI posts stay the Developer's.** The agent room and the relay
  write as the Developer: the Front Desk, routine starts, the completion
  door. They are the human's UI. No tracked code defaults a posting path
  to `developer.env`, and no tracked default was changed.

## Checks retained

- **Acceptance follows the result actually reviewed.** Evidence older than
  the shown result (`+shown=`/`after=`) is refused for the proxy as for
  anybody (p5/p6 tests, unchanged).
- **A worker never approves its own work.**
  - `test_the_worker_never_accepts_its_own_work_even_with_a_proxy_on_record`
    covers this, with a proxy configured.
  - An ordinary agent's post answers, releases and accepts nothing for the
    Developer (`archsage` in the tests).

## ex1's wording

- `ex1/report.md` and `ex1/report5.md` said "the Developer in person" for
  #15580.
- The post came through the API client (`Python-urllib`). Only the
  Developer **account** is evidenced.
- The wording now says "Developer-account post", with a note that p6 ex2
  corrected it. The conversation itself, and Front's #15606 record, are
  left as they are.
