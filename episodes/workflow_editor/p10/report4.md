# p10 step 4 — Detailed IO channels and direct links

## What was done (agdevworld)

- **Existing utilities first.** The attempt logs the run view's activity
  line reads (`server/activity.ts`: stream-json lines, append-only reading by
  offset) are the source. Observer's `logFacts` showed which events carry
  what. No redaction rule existed in the code base, so one was written here.
- **`server/io.ts`** (new):
  - **Log to entries**: the session start (id, model, version only — never
    tools, servers, paths or sockets), agent text, tool calls, tool results
    (named by their call), the session end (turns, duration, cost), and
    background-task notes.
  - **Never shown**: `thinking` blocks (reasoning is not public output),
    rate-limit and token bookkeeping.
  - **Redaction**: credential patterns, plus the exact values of the host's
    credential files listed in `zulip.json` `io.secretFiles`.
  - **Rendering**: mentions are broken; code fences are longer than any
    backtick run inside. Messages hold at most 3500 characters; bodies over
    900 are cut with a pointer to the whole entry.
- **Notifier** (`server/notify.ts`, store `user_version` 4):
  - one IO target and conversation per run (`runs.kind = io`): channel
    `wf-<project>.<workflow>.<run>.io` in the project's folder, topic `io`,
    the run uid in the description. The existing transport provisions,
    adopts and reports it without changes; its keys are shaped for that;
  - `io_attempts` keeps each attempt's offset, line, inode and the reader's
    open calls;
  - `ioScan()` runs in every `due()` pass (the transport's thread, outside
    the slot). It queues "Attempt … began" (why, backend), the entry
    messages, then "Attempt … ended — <the record's outcome>";
  - small increments wait about 15 s of quiet or 1500 characters;
  - a log that shrank or was replaced, and a missing log of an ended
    attempt, are each said as a gap;
  - IO deliveries are ordinary frozen, journaled, retried deliveries.
- **Links**:
  - The request receipt ("Request received") and the run topic's "Started"
    gain `Detailed IO:` beside `Progress:` and `Run:`. While the channel is
    not provisioned the receipt says "being set up; a follow-up message here
    gives its link", then "Detailed IO of run … is ready: <link>" follows,
    once.
  - Run view: the title band shows Conversation · Progress · Detailed IO in
    Simple and Detail. A new page `#/ws/<ws>/run/<wf>/<run>/io/<attempt>/<line>`
    shows one entry in full, redacted, backed by `GET …/io?attempt=&line=`.
  - `wfe run io <run> [--attempt --line]`: the channel, each attempt's
    relayed position and any gap, or one entry in full.
  - The parent's delegation messages (result, failure, cancellation) carry
    `Child: conversation · progress · detailed IO · run view`. The child's
    IO is never copied into the parent's channel.
- **Fixed on the way** (pre-existing, p6): from a request topic the `Run:`
  link used the request key (`…/run/make/r42`) instead of the run id.
  Production never showed it because its `viewBase` is null.
- **Contract**: `docs/io.md` (new); `runs.md` cross-reference.

## Completion evidence

`test/io.test.ts` (6 tests):

1. Entries are public output only. Reasoning text, the init event's cwd,
   servers and memory paths, rate-limit and accounting events never appear.
   A result names its tool.
2. Redaction covers env-file and JSON token values, Anthropic, GitHub,
   bearer, URL credentials and `password=`; ordinary words stay.
3. Long output is cut with a pointer. `@**Front**`, `@_**Observer**` and
   `@*team*` do not form mentions; nested fences survive; every message is
   ≤ 3500 characters with full line coverage.
4. The channel itself:
   - "Attempt … began", the entries in order, and a working link to the
     cut result's whole entry. "Started" names `Detailed IO: <channel>`.
   - **No replay** on a second pass, nor after a restart (a new store over
     the same files). Appended lines only are sent.
   - The ended boundary carries the record's outcome.
   - The IO channel is never among the intake's channels.
   - A **second attempt** gets its own boundary in the same channel.
   - A **truncated log** is a gap; a **missing log** of an ended attempt is
     a gap, said once, and the attempt is closed.
   - The status shows both attempts' positions and gap.
5. **Zulip outage**: every delivery fails, the IO stays owed (≥ 2 events)
   and the attempt is untouched. After the retry it is delivered. Without a
   remote view address, the whole entry is named as `wfe run io …` on the
   execution host.
6. **Request receipt**: it carries Progress, `Detailed IO: being set up…`
   (no premature link) and `Run: …/run/make/run-001`. The next pass sends
   "Detailed IO of run make/run-001 is ready: <channel>" in the request
   topic, once.

Suite: **332/332**, type check and build clean. The real Zulip channel, the
deployed receipt and navigation from a person's side are verified in step 6
on the deployed release.

## Decisions

- The IO channel is created for runs whose creation is at or after
  `io.since`. Nothing is backfilled for older logs.
- `viewBase` must be set in production `zulip.json` (it is `null` there
  now) for the receipt's `Run:` link and the whole-entry links. Step 6 sets
  it to the LAN address.

## Next

Step 5: the delegate inspector (project/workflow selection and validation),
the node and run views of delegations, and delegation-aware observation in
`wfe observe` and Observer.
