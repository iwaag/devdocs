# argue p5 — Step 3: generation, cache and isolated reads

## Result

| Repository | Revision | Change |
| --- | --- | --- |
| agdevworld | `1d77ef1` | `workflow_editor/server/dialogue.ts` (store, snapshots, keys, jobs, worker), routes in `server/api.ts`, worker start in `server/main.ts`, `wfe dialogue`, `docs/dialogue.md`, `test/dialogue.test.ts` |
| developer/pj-aghangout | `53386ff` | `renderer/render.ts` and `renderer/README.md`, renderer tests, `scripts/deploy.sh` installs `renderer/<commit>` + `renderer-current`, character export `6c7e9f3` |
| developer/pj-characters | `6c7e9f3` | archsage's style revised after the first rendering (below) |

Nothing is deployed yet: the service still runs `2eb1976`. The trial
below ran through the new code in a CLI process.

## How it works

- **Renderer (aghangout).**
  - `describe` gives the version (`dialogue-4`), the model (`sonnet`),
    the character export's commit, and a voice hash per mapped
    participant plus the neutral one.
  - `render` turns one `ag.dialogue-request.v1` into one
    `ag.dialogue-result.v1`.
  - The model call is Claude Code `-p`: no tools, no MCP, no setting
    sources, no session persistence, a JSON schema, working directory
    `$HOME`.
  - The prompt keeps stance and function:
    - a proposal, objection, question, request or result stays one;
    - claims are never upgraded and nothing is invented;
    - an essential qualification or question is kept;
    - it speaks only from the target message; context is for continuity;
    - 60–120 characters, past 160 only for an essential question.
- **Snapshot (WFE).** A request holds:
  - the target message;
  - the earlier messages it replies to, plus the four before it. These are
    originals only, never anything later, each cut at 1,500 characters;
  - the participants' kinds and project names.
- **Key.** The key is sha256 of renderer, model, voice and the request. A
  result records message, speaker, character, renderer, model, the
  characters commit, the source hash, the context ids with their hashes,
  the horizon, the text, usage and time.
- **Storage.** `pj-ponderer/.local/character-dialogue/dialogue.sqlite`,
  resolved from `discussions.archive` and refused unless Git ignores it.
- **Jobs.**
  - Each job is unique by key and leased for 5 minutes. A failure is
    retried with backoff; after `tries` the job is `failed`, with the
    reason.
  - A job is `dropped` if the renderer or the character changed before it
    ran.
  - Result and completion are one transaction.
  - The worker runs in the service process and follows new agent speech
    from a checkpoint taken at its first start.
- **Reads.**
  - `GET api/discussions/<id>/dialogue?lang=` returns states `original`,
    `ready`, `pending`, `running`, `failed` and `unrequested`, with
    `stale`. A read queues at most `window` (40) lines and never calls the
    model.
  - Retry and regenerate: `POST …/dialogue/retry`.
  - `GET api/dialogue`.
  - `wfe dialogue status|show|history|retry|work`.

## Verification

`test/dialogue.test.ts`: 8 checks, all pass. The full WFE suite passes
505/505.

| Check | Shown |
| --- | --- |
| storage | resolves to `<archive ws>/.local/character-dialogue`; a workspace that would commit it is refused, and the originals stay readable |
| identity | one render per line; reading again queues nothing; ready lines are never rendered again; another language is another line; the person's message is `original` |
| context | only the replied-to and four earlier originals; a later message never appears, even when named in `re`; cut at 1,500 |
| invalidation | a character change shows the line as `stale` and not as current; the old result is kept; an edited source is a new key |
| failure | retried, then `failed` with the renderer's reason; reading does not requeue it; `retry` requeues it, and a second retry is a no-op; regenerate keeps the current line until the new one is in; a person's message is refused |
| interruption | a running job is not claimed twice; an expired lease is reclaimed; a second completion of the same job writes nothing (one result) |
| follow | first call only sets the checkpoint; only later agent speech is queued |
| boundary | discussion view, archive publication (manifest/transcript) and a run copy of the archive project contain no line or folder; the archive workspace stays clean |

aghangout: `npm run check` (typecheck, build, 28 unit tests including 3 for
the renderer's prompt and voices) passes.

## The trial discussion, rendered early

The trial used p16-s3-room, its 14 agent messages, in Japanese.

1. **Prompt v3 against both models**, all 14 messages:
   - `sonnet` kept every stance, question and uncertainty, but ran long:
     up to 286 characters, with message numbers.
   - `haiku` was about ten times cheaper.
2. **One prompt change** (`dialogue-4`) after these failures:
   - an aim of 60–120 characters, past 160 only for an essential question;
   - drop reasons, ids, run names and message numbers.
3. **One character change**, in the characters project with
   `tools/characters add archsage --update`. The `sonnet` lines made
   archsage an elder throughout (「わしの考えを話すぞ…じゃ」), so its style
   became a child who uses an old-fashioned phrase only now and then. That
   is a data change, not code: `6c7e9f3`, refreshed into aghangout.
4. **Both models again on `dialogue-4`.** `haiku` broke the rules:
   - #139: 304 characters, still with file paths;
   - #134: misread the three room images as 「会議室3枚」;
   - #136: rendered `front` as 「正面の部屋」.

   **Default: `sonnet`.**
   - My first suspicion that `haiku` invented #150's 「約1日の見積もり」 was
     wrong: the original says "about a day … a judgement, not measured".
5. **Through WFE**: `wfe dialogue show p16-s3-room --queue`, then `wfe
   dialogue work` with concurrency 3. All 14 lines were ready:

   | Measure | Value |
   | --- | --- |
   | Wall time | 36.3 s |
   | Per line | mean 7.6 s, median 7.7 s, max 12.0 s |
   | Cost | $0.1033 in all, $0.0074 per line: list-price cost reported by Claude Code (`costBasis: list`); prompt caching made later lines cheaper |
   | Tokens | 2,675 output; cache read 39,305, cache creation 16,746 |
   | Line length | mean 157, max 226 characters, against a 1,920-character mean in the originals |

   Claude Code adds a small auxiliary `haiku` call to each run (about
   $0.0006). The first renderer build recorded that as the line's model, so
   these 14 results say `claude-haiku-5-5`. The renderer now records every
   model and names the costliest (`sonnet`) as the line's. Step 5 renders
   these lines again in any case.
6. **Live boundary.**
   - `pj-ponderer` `git status` is clean.
   - The archive clone has no `character-dialogue` path, and no line's
     text appears in the archive or in `discussion show --json`.

Evidence (ignored): `pj-agdev/.local/argue-p5/step3/early-trial/`:

- `req/`: the 14 requests;
- `out-sonnet`, `out-haiku`: prompt v3;
- `out-v4`, `out-v4h`: `dialogue-4`.

## Configuration on this host

`conversations.json` gained `dialogue`:

- `languages ja,en`, `follow ja`;
- `concurrency 3`, `window 40`, `tries 3`;
- `env.AGHANGOUT_CLAUDE`.

`renderer` points at the aghangout workspace for now. In Step 6 it moves to
the deployed `hangout/renderer-current/renderer/render.ts`. The cache key
does not include the path, so the trial's lines stay valid.
