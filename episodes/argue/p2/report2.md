# Step 2 — Front's discussion runs and character-rendering runs are separate

Date: 2026-09-18 JST. Plan: [plan.md](plan.md) step 2. The running Front
listener is still on the old code; nothing was posted to the realm. One paid
probe of the new role was made from a shell (below). Deployment is step 5.

## What changed

### The discussion carries no character

- The Front Desk role is **`desk`** (was `character_talk`): guide
  `agent/guides/desk/guide.md` is the old guide with every character part
  removed — no "who you are", no two voices, no `ag-dialogue` contract — and
  one sentence added: the conversation is the substantive record, how it is
  shown is somebody else's work. It replies in the developer's language.
- `zulip_listener.serve` no longer pins a settings revision, writes
  `characters.md` / `settings.json`, stamps `settings_revision` into the run
  record or parses the reply. A desk reply is posted as written, like a
  `front` reply. `agfront.dialogue` kept only the validator
  (`parse_block`, `split_reply`); `finish_reply`, `render`, `strip_preface`
  and the `ag-dialogue-error` fence are deleted with their tests.
- The argue role never had a character; a test now pins that too. Human
  input, the human desire note and the argue anchor are untouched.

### `agfront.present` — the presentation role

One run: recorded speech in, one `ag-dialogue` block out. Its role in
`agents.toml` is `present` (own profile, Sonnet 5 through claude_code) with
`allowed_tools = "Read,Glob,Grep"` — no `agentchat`, no shell, nothing to
write with; it is run without `AGENTCHAT_HOME`. Guide:
`agent/guides/present/guide.md` (faithfulness is the whole job: claims,
numbers, names, links, uncertainty, disagreement and questions survive; only
transport is dropped).

| Thing | Decision |
|---|---|
| Input | A snapshot written once per job: `sources.md` (the posts to re-voice, `[speaker #id]`, each with its character id or "no character — skip"), `context.md` (up to 12 earlier posts, bounded, context only), `characters.md` (only the lore of the characters this job needs). The same text is in the prompt. |
| Speakers | An account with an `#agents` introduction is an agent, by roster `agent` name → manifest `agents`. A post under a `**[sage:<name>]**` header is the logical speaker `sage:<name>` → manifest `senders`. `archsage` and `sage:arxiv` are different speakers on one account. Anybody without an introduction is a human: shown verbatim, never re-voiced. |
| Missing character | Not an error: the posting layer adds a `plain` turn (`character: null`, the speaker's label, the source id) and the view shows the post as written. The model is told to skip it. |
| The check | `parse_block` (shape, known characters, sizes, mentions reduced to names, sources may name only a `message_id`) plus: every turn cites a post, every cited post is one of the job's, the turn's character is the cited speaker's character, every post with a character got a turn. Any failure is a `PresentError` → the job retries; nothing partial is saved. |
| The record | `ag.memo-dialogue.v1`, one fenced `ag-memo` JSON per memo post: `job`, `source {anchor, channel, topic, messages, fingerprint}`, `settings_revision`, `renderer` (`agfront.present/1`), `part`/`parts`, `turns` (each with `speaker`). Split into parts under 8,500 characters, because Zulip truncates silently. |

### `agfront.render` — the job mechanism

A worker thread started by `agfront.listener.main` beside the listener, on
the **same mirror** (one reader of the realm per process, as the Observer
does), with its own checkpoint, its own client and its own store,
`.local/render/render.sqlite`. It never calls a discussion handler.
`AGFRONT_RENDER=0` or log-only mode leaves it off.

| Requirement | How |
|---|---|
| Trigger | New **agent speech** in a source — `#front › front-desk-…` or `#argue › argue-…` — on the change feed: Front's replies and every specialist's contribution. Acks, selfnotes, Zulip notices and human posts are not rendered. |
| Source identity | An argue's `[selfnote][argue]` id; a desk conversation's first post. Survives a rename and a resolve. |
| Coalescing | Speech marks the source *dirty*; a job is planned after 45 s of quiet, or up to 10 min while a serving's ack is the newest post (a reply is coming). One job is at most 6 posts. |
| No backfill | The first start sets the checkpoint to the present: nothing already in the realm is rendered unasked. A truncated feed resumes from now, never from the index. |
| Job identity | `j` + digest of anchor, message ids, content fingerprint, settings revision, renderer version. Planning the same thing twice is one row. |
| Reuse / no duplicate | Before any run the memo is searched for the job's record; a complete one finishes the job. A job that was `running` at a restart is also checked against Zulip itself, since the mirror may be one post behind. The validated result is kept as `jobs/<job>/result.json` **before** posting, so a failed or half-done post re-posts and does not re-run. |
| Failure | 3 attempts, backoff 60 s × 2ⁿ, then `failed` plus one `failed` record in the memo. No restart, reader or timer re-arms it. The discussion is untouched. |
| Another interpretation | `[selfnote][render] <settings revision>` in the source, by a non-agent account (the relay will write it as the Developer in step 3). Every agent post without a complete, fingerprint-matching result at that revision becomes jobs (newest 60), failed ones are re-armed, earlier results stay. Requests are remembered by message id; a revision that is not retained is answered with a `refused` record and no run. |
| Edited / deleted source | The record's fingerprint no longer matches the source, which a reader can see (stale). Nothing regenerates by itself; a new request renders the changed post because its job id is new. A source that changed between planning and running fails the job rather than rendering something else. |
| Memo | `#memo › <source topic>-s<anchor>`, opened with `[selfnote][memosource] <anchor>`. No root note, no mention route, no `threads/` — step 1. |
| Records | `.local/agent/present/run-NNNN.json`, with `render_job`, `source_anchor`, `source`, `source_messages`, `settings_revision`, `renderer` — so presentation cost is countable apart from `desk` / `argue` / `front`. |

## Verification

`tests/test_render.py` (11 tests, mirrored `FakeRealm`, injected run and
clock) and the rewritten desk tests:

| Plan item | Test |
|---|---|
| Discussion runs receive no character settings or memo text | `test_a_front_desk_run_receives_no_character_settings_at_all` (a settings tree full of marked lore is synced; the prompt and every file of the workspace are searched for it), `test_the_desk_guide_exists_and_carries_no_character`, the argue serving test, `test_a_desk_reply_is_posted_as_written_even_when_it_carries_a_block`. Memos cannot reach `threads/`: no root note is written there and the note readers skip memo channels (step 1). |
| Rendering receives the intended sources | `test_a_burst_of_agent_speech_is_one_job_with_the_intended_sources`: Front's reply, autolab's contribution and a `sage:arxiv` post are one job; the prompt holds their text, the human's post as context and only the needed lore; the ack and the selfnote are absent; the record's anchor, ids, fingerprint, revision and renderer are asserted; the sage is a `plain` turn. |
| A rendering failure leaves the discussion usable | `test_a_failing_renderer_is_bounded_and_touches_nothing_but_the_memo`: three attempts, then `failed`, one `failed` record, and the set of non-memo messages is unchanged. |
| Retry / restart does not duplicate | `…healed_without_a_second_run` (store says `running`, `result.json` gone, a new `Renderer`: recognized from the memo, 0 runs, 1 record), `…reuses_the_validated_result…` (the post fails once: 1 run, 1 record), `…plan_nothing_twice` (the feed replayed from 0). |
| Another settings revision is a distinct interpretation | `test_another_settings_revision_…`: same source ids, two records, two job ids, only revision B's lore in the second prompt; an agent's request is ignored; repeat, restart and replay buy nothing; an unknown revision is refused. |
| Edited source | `test_an_edited_source_is_identifiable_and_rendered_again_only_on_request`. |

agfront: **179 passed**.

**One paid probe** of the real role and guide, settings revision
`2c21078ae237`, a synthetic desk exchange (Front's proposal with an
uncertainty, autolab's plan with a commit id, topic names and an unverified
token): exit 0, one turn, **$0.175**, three valid turns. The commit id, both
`workrun-` names, `HTTP 401` and both uncertainties survived; autolab's long
post became two turns of the same character. A rendering is currently about
the price of a small discussion run — moving `[profiles.present]` to a
cheaper model is one line and touches no discussion role.

## Commits

| Repository | Commit |
|---|---|
| agfront | `03bf127` |

## Notes for the next steps

- Status the relay can read from the mirror alone: a `result`, `failed` or
  `refused` record. *Pending* is derived — agent speech with no record at the
  shown revision; the renderer posts nothing when it plans a job.
- The `memo` channel does not exist yet; `ensure_subscribed` would create it
  with the bot's rights, so step 5 creates it with the provisioner first.
- The settings manifest does not register `archsage` yet (step 4); until then
  its posts are `plain` turns, which is the designed fallback.
