# Agent session p1 — final report

Braindump: can a workflow run be served as one harness session instead of
one-shot attempts, now that prompt caching is so effective? Either way, make
the choice a policy selectable in Detail, starting with the two extremes, and
experiment in RTS. Step reports: [report1](report1.md) … [report6](report6.md).

## Outcome

- **Both policies exist and work.** `execution.session: fresh | run` is a
  Detail setting (Workflow settings → Execution). It is frozen with each run
  and travels through validation, the definition digest, claim, CLI, API, the
  run view and Observer. A workflow that does not declare it uses `fresh`.
- **`run` keeps a run's conversation across attempts without keeping a
  process.** A question or hold still ends the attempt and frees the slot. The
  next attempt resumes the run's session by an explicit id. That id is minted
  by WFE and recorded in `attempt.begin` before launch, and then confirmed
  from the harness's init event.
- **The default stays `fresh`. The evidence did not show `run` saving
  money.** Two matched RTS pairs cost $8.07 for `run` against $5.99 for
  `fresh`: one loss, partly from extra work, and one tie after a 65-minute
  wait. The bootstrap a retained session saves is small, 16–22 k tokens
  written per continuation, because new sessions already hit the shared
  system-prompt cache. A retained session carries its whole history into every
  later request (up to 176 k tokens here), and it loses the cache across waits
  longer than the 1-hour TTL. Quality was equivalent within this small
  sample (one fresh run lost a requirement while one retained run kept it).
  This sample is two pairs, one task, one model.

## Capability findings (claude_code 2.1.283, the RTS profile)

| Finding | Detail |
| --- | --- |
| Explicit ids | `--session-id <uuid>` creates under a caller-chosen id; `--resume <id>` continues exactly that conversation. |
| Error cases | An id already in use is refused before any model call. A missing id fails with "No conversation found", exit 1, no tool run. |
| Ids are not scoped | An id is not scoped to a directory: a resume launched elsewhere found and continued it. |
| No writer lock | The CLI does **not** refuse a second concurrent writer. |
| Resume carries nothing implicitly | It does not replay old events into the new stream. It does not lock the model (a different `--model` is accepted and the cache is lost). It keeps the system prompt recorded on the first request until compaction. |
| Cost fields | `result.usage` is per invocation. `total_cost_usd` and `modelUsage` are cumulative for the session. A killed invocation saves no totals. |
| Compaction | Keeps the id and emits `compact_boundary` plus a synthetic summary event. Its usage is not in `result.usage`. |
| Other harnesses | codex (ephemeral), agy, gemini_cli, agcode: not supported. `run` is refused for their profiles and never run fresh instead. |

## Actual semantics

These are defined in agdevworld `workflow_editor/docs/sessions.md`.

- **Claim decides the session.** The inputs are the frozen policy, the run's
  `sessions[]` and the host's `--sessions-json`. `run` resumes the current
  session only under an identical configuration (agent, profile, backend,
  contract and guide hashes). Otherwise a new session starts, and the reason
  names what changed. The record keeps portable identities only; transcripts
  stay in the host's store.
- **Replacement paths:**
  - configuration change, at claim;
  - missing conversation, from the store check before launch (only while
    nothing ran in the session in this attempt);
  - `wfe run session <run> --retire`, by a person or a recovering agent,
    between attempts.

  A resume that launched and then failed is an interrupted attempt, not
  permission for a fresh retry.
- **Continuation prompts.** A new session gets the full bootstrap. A resumed
  one gets a continuation of about 2.5 k characters: the new attempt id, that
  earlier ids are ended and their processes gone, that memory is context and
  the record decides, the brief, and how to read the run. The wfrun guide
  states these facts.
- **Single writer.** One slot, attempt fencing (including `attempt.session`),
  and no launch while a process of the run lives. These matter because the
  harness does not enforce a single writer.
- **Observer.** It reads `session` / `process.session` from the record.
  Incidents stay per attempt. A retained session is no process and no stall
  timer. The recovery request names the sessions and how to retire one.
  Recovery budgets do not reset when a session is replaced.

## Failure-path evidence

Steps 3–4 covered these with the real harness, plus stub tests:

- an independent run keeps its own session;
- fresh starts a new session each attempt;
- wait A / run B / resume A;
- SIGKILL mid-attempt, then Observer finding (attempt episode), then
  recovery, then a resume in the same session; the agent checked record and
  git before acting and did not replay the killed command;
- the same under `fresh` (new session);
- a transcript removed from the store: replaced before launch and recorded;
- a human stop during a wait: the answer did not resume the run, Observer
  marked it `respected-hold`, the person's resume continued the session;
- cancellation;
- early crash plus executor restart: no second writer; the unstored session
  is replaced.

There was no duplicate writer, lost answer, false stall or bypassed hold.

## Versions

- Source and deployed:
  - agdevworld `dab1e54` (release current; previous `314c98b`);
  - agautolab `c08cb5d`;
  - pyagag `4dde895`;
  - pj-agdev `agobserver` `d1ce1d5`, with its lock at pyagag `4dde895`.
- Services restarted on 2026-10-14 at 17:05 JST: `com.agdev.wfe-service`,
  `com.agdev.agautolab-wfexec`, `com.agdev.agobserver`.
- Deployed check: agops `session-check/run-001`, 2 attempts in one session,
  completed.

## Remaining limits and next steps

- **Small sample.** Two pairs, one RTS task (lighter than expected: the
  feature already existed, so the correction was the real work), one model.
  Many short immediate continuations might favour `run`. Long implementation
  attempts and human-length waits do not.
- **What would make `run` cheap:** compaction or per-node session splitting
  (out of scope). Without one of them the carried history dominates.
- **Delegation.** Future delegate nodes keep their own run, wait identities
  and deadlines between execution intervals. No conversation ancestry is
  inferred. Delegate nodes, cross-agent conversations and model routing were
  not implemented.
- **Trial driver.** It answered pair 2's re-check with "お任せします", so those
  two runs completed without an acceptance decision. Both arms got the same
  input; this is noted in report5.
- **Pre-existing, not fixed.** Over plain LAN HTTP the editor is not a secure
  context, and workflow pages throw on `crypto.subtle` (approval digests). It
  predates this phase.
- **Deus Ex Machina note.** None: no in-system agent's work was done by the
  Omni Agent. The trial inputs were scripted and labelled as such.
