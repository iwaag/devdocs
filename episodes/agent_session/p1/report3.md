# Agent session p1 — step 3: session ownership, identity and continuation

Date: 2026-10-14. Source: agdevworld `e7fbe73`, agautolab `600448a` + guide
`c08cb5d`, pyagag `4dde895`. Evidence (attempt logs, measurements, session
listings) is in the ignored notes `pj-agdev/.local/agent-session-p1/step3/`.

## Who owns what

| Layer | Owns |
| --- | --- |
| `agag.harness` (shared seam) | Harness-specific arguments and storage. `Session(new\|resume, id)` becomes `--session-id` or `--resume` (claude_code only, `SESSION_HARNESSES`); any other harness refuses a session (ValueError). `--resume`, `--continue` and `--session-id` cannot be smuggled through `extra_args`. `session_stored()` checks the claude store (`<config dir>/projects/*/<id>.jsonl`). `stream_session_id()` reads the init event. |
| WFE (`server/exec.ts`, `shared/run.ts`) | The workflow's lifecycle. Which session an attempt uses is decided at claim, from the frozen policy, the run's recorded `sessions[]` and the host's capability. Identity is recorded in `run.json`. Every rule is a reducer rule. |
| `agautolab.wfexec` | Launching and reporting: the capability offer, the store check before a resume, the prompt shape, and the attached id. |

## Identity

- **The id is minted by WFE** (a UUID) for explicit-id harnesses and
  recorded in `attempt.begin` `settings.session` *before launch*. Executor
  restarts and early crashes therefore never leave an unnamed conversation.
  For a harness that only reports its id later, the representation allows
  `id: null` until `attempt.session attached` supplies it; no such harness is
  enabled in this phase.
- **Always explicit.** A resume names the run's current session id. Nothing
  uses `--continue` or "the latest conversation". A new session must use an
  id the run has never used (reducer: `session-mismatch`). A resume must name
  the current session (`session-mismatch`). The harness itself refuses to
  create a session under an existing id.
- **Bound to the run and its configuration.** A session record keeps the
  agent, profile, backend, contract hash and guide hashes it was created
  under. At each claim WFE compares them with the attempt's freshly resolved
  settings. Any difference starts a **new** session, and the replacement reason
  lists exactly what changed (for example `configuration changed: guide
  agent/g.md changed`). An old conversation is never assumed to have learned
  new instructions. Another run never sees the id.
- **`run.json` keeps portable facts only.** It records id, policy, the
  attempts served, the configuration, `state` (open / replaced / retired), the
  id the harness reported (`attached`, `attachedIn`, `mismatch`), and why and
  by whom a session ended. Transcripts stay in the host's harness store.
- **Replacement paths:**
  - *Configuration change*: new session at claim, with the reason.
  - *Missing conversation*: the executor checks the store before a resume.
    If the conversation is absent, it reports `wfe exec session <attempt>
    --missing`. That is allowed only while the session was not attached in
    this attempt, so nothing has run in it. WFE records the replacement (old
    id, new id, host, reason), and the same attempt launches a new session
    with the full bootstrap. A repeated report returns the same replacement.
  - *Retirement*: `wfe run session <run> --retire --reason …` by a person, or
    by a recovering agent with `--by`. It is refused while an attempt is open
    or a process of the run lives. The next attempt starts new, with reason
    `retired: …`.

## Continuation

- **New session** (first attempt, `fresh`, or a replacement): common
  contract + project guides + brief, as before (about 19.9 k characters in
  the trial).
- **Resumed session**: a continuation of about 2.5 k characters. It contains:
  - the new attempt id;
  - a note that earlier attempt ids in the conversation have ended and their
    reports are refused;
  - a note that processes started earlier are gone;
  - a note that memory is context, not record;
  - the brief (why this attempt runs: answers, instructions, recovery,
    commits since the base, ready nodes, documentation steps);
  - the command to read the run.

  Contract and guides are not appended again. They are unchanged, because a
  change would have forced a new session.
- **Fencing is unchanged.** `WFE_ATTEMPT`, the process group, the log file
  and the time budget are renewed per attempt. A report carrying an old
  attempt id is refused (`attempt-stale`). The same fencing guards
  `attempt.session`.
- **Guide** (`wfrun/guide.md`, factual): the session policy decides which
  conversation a later attempt continues in. Records must stay sufficient for
  a new session. A resumed conversation remembers, but the record decides
  what was answered, accepted, held or completed.

## Evidence

**Automated** (all green):

- WFE 306/306: reducer rules, claim decisions, replacement (configuration,
  missing, retired), stale-attempt session reports, idempotent missing report.
- agautolab wfexec 13/13 against the real WFE CLI with a session-keeping
  stub:
  - `run` resumes one session across the wait;
  - an independent run gets its own session;
  - `fresh` gets a new session each attempt;
  - `run` on an incapable profile is refused;
  - a missing conversation is replaced before launch, never resumed blindly;
  - a changed guide gives a new session that carries the new guide text;
  - **early crash + executor restart**: the restarted executor stays idle
    while the agent lives, so there is one writer of the session. After the
    kill the run is held `unknown` with the intended id recorded but never
    attached. After the person's resume, the unstored conversation is
    replaced, not resumed.
- agautolab full suite 372 passed / 1 skipped. pyagag 1218.

**Real harness** (claude_code 2.1.283, `claude-sonnet-5`, private area, no
production touched). Workflow: talk "which language?" → do "write
greeting.txt". Each run answered once.

| Attempt | Run | Policy | Session | Mode | First request: write / read | Steps | Reported cost |
| --- | --- | --- | --- | --- | --- | --- | --- |
| amv7nkess | keep/run-001 | run | ba3db6df | new | 51.9 k / 0 (cold) | 5 | $0.280 |
| amv7nl4yd | keep/run-001 | run | ba3db6df | **resume** | 1.0 k / 55.3 k | 10 | $0.151 (delta) |
| amv7nma1a | keep/run-002 | run | 269b7d8a | new | 21.2 k / 31.6 k | 5 | $0.162 |
| amv7nnj8t | keep/run-002 | run | 269b7d8a | **resume** | 1.3 k / 55.7 k | 12 | $0.192 (delta) |
| amv7nmvg8 | anew/run-001 | fresh | 79da9cf7 | new | 20.2 k / 31.6 k | 6 | $0.178 |
| amv7nogsv | anew/run-001 | fresh | c1add7e8 | new | 14.8 k / 37.1 k | 11 | $0.227 |

- **One run, several attempts, one explicit session.** In keep/run-001 both
  attempts carry `ba3db6df`: in the record (`attempts: [amv7nkess,
  amv7nl4yd]`, `attached` equal to the intended id) and in the harness's init
  events. The transcript holds the bootstrap turn and then the 2.5 k
  continuation turn (`# This attempt: amv7nl4yd-c6cc80 …`). The resumed first
  request read the whole earlier conversation from cache (55 k read, 1 k
  written).
- **Another run is independent.** keep/run-002 has its own session.
- **Fresh** used a new session per attempt. The record shows the first
  session replaced ("fresh policy").
- All three runs completed correctly (こんにちは!, Bonjour !, Bonjour !).
  No report used a stale attempt id (no `attempt-stale` in any log).
- Trial cost: about $1.19 reported (client-side estimate on the
  subscription).
- These are single trials. The cost comparison belongs to step 5. Note that
  `total_cost_usd` of a resumed attempt is cumulative; the table shows
  deltas.

## Left for step 4

Observer's view of shared sessions, waits and holds under both policies,
interruption and recovery with the real harness, and missing storage in the
deployed path.
