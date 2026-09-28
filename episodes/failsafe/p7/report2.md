# failsafe p7 — step 2: how a claim is known

## Decision

**Option 2, narrowed: a local reader extracts the claims and code compares
them with the records.** Options 1 and 3 are not used.

- A reader on the host's local model gets the reply's words and nothing
  else. It returns the acts the reply says its author did now, each as
  `{act, target, where, quote}` over step 1's vocabulary.
- Code compares each extracted act with the records as step 1 describes:
  the serving's window first, then a record already made. Only a claim
  with no record is a mismatch.

The model reads and code judges. The model's task stays small (extraction
in the conversation's own language). The comparison, which decides whether
anything happens, is deterministic and testable without a model.

## Why not the other two

- **Option 1 (the reply declares its acts).** A declaration is the agent's
  own account of what it did, and the agent's own account is what failed:
  run-0183 described commands it never ran. The agent would have to write
  the declaration, which needs guide text. agent_guide p2 could not show
  that a sentence helps against a 1-in-13 event, and the terms say the
  check belongs to the system. A reply that claims in prose and declares
  nothing would still need the reader, so a declaration adds a second path
  and no coverage.
- **Option 3 (structural: a decision trigger and no record).** Knowing
  that a trigger *is* a decision (a release, a cancel, "stop following
  this") means reading prose in the Developer's language. The cheap proxy,
  "the holder spoke while a hold is in force and nothing was recorded",
  also fires on "how is it going?". Each false alarm buys a paid serving
  of the agent. It also misses every claim on another trigger, including
  the costlier one: a send that never happened.

## The reader, measured

The probe is a direct call to ollama's chat API: one system prompt, the
reply as the user message, JSON format, temperature 0, thinking off. Model
`qwen3.8:27b-mxfp8`, the one Observer's triage and watch evaluations
already keep loaded. The kit is `pj-agdev/.local/failsafe-p7/reader_probe.py`
(ignored).

Cases: the three incident replies (#15838, #15844, #15851, from Front's
mirror) and ten written for the shapes the plan names:

- a false-alarm pair: "still in force" and a quoted command offered, not
  run;
- a restatement of records that exist;
- an offer ("shall I?");
- another agent's act;
- sends in English and Japanese;
- a release, disposition and acceptance in Japanese;
- a Japanese status reply with no act.

| case | expected acts | extracted, 3 runs + 1 verbose |
|---|---|---|
| #15844 (the false claim) | release, disposition | release (#15837), disposition — 4/4 |
| #15838 (the real hold) | hold | hold — 4/4 |
| #15851 (the repair) | release, disposition | release, disposition, **send** (from `ag-post … re=15847`) — 4/4 |
| still in force | — | — 4/4 |
| quoted command, "shall I?" | — | — 4/4 |
| restatement "recorded as withdrawn (#15850), released at #15849" | — | release (#15837), disposition (#15850) — 4/4 |
| offer | — | — 4/4 |
| "autolab recorded the acceptance of m20402 at #20510" | — | accept (20402) — 4/4 |
| send, English / Japanese | send | send — 4/4 each |
| release + disposition, Japanese | release, disposition | both — 4/4 |
| acceptance, Japanese | accept | accept — 4/4 |
| status, Japanese | — | — 4/4 |

Latency: 0.3–3.9 s per reply while the model was loaded, 8.5 s for the
first call. The reader was deterministic across runs: every case gave the
same answer all four times.

What the misreadings teach:

- **The machine line must not reach the reader.** #15851's extra `send`
  quoted `ag-post intent=report re=15847 end=15848`. The reader gets the
  run's reply body: the journal's reply text without the handoff mention,
  the `ag-post` line or the listener's notices.
- **Over-extraction is absorbed by the comparison when the world agrees.**
  The restatement and "autolab recorded…" are acts the reader should not
  have listed. Each names a target that is on record: hold #15837 released
  at #15849, disposition #15850, m20402's acceptance. Step 1's rule 2
  ("already on record") finds them, so no notice is sent. A target is
  matched against the record's own id **and** every id the record names
  (`a<unit>`, `#<hold>`, `#<evidence>`, `upto=`). The reader named the
  disposition by its own id (#15850) in one case and by the unit (#15835)
  in another.
- **What remains a false-alarm risk**: an over-extracted act with no record
  anywhere, for example a hypothetical act read as done. None of the ten
  written cases produced one after the machine line was removed. The
  step-5 fixture measures it on the probe's shape.

## What the check does with the reader's answer

| reader | outcome |
|---|---|
| no acts | `clean`: nothing written |
| acts, every one found (in the window or already on record) | `clean`: nothing written; the journal keeps what was matched to what |
| an act with no record | `mismatch`: step 3 |
| unreachable, timed out, or unreadable JSON | `unchecked`, kept on the serving record and tried again. After the last try it stays `unchecked`, counted in the listener's status file and logged. It is never read as clean |

A serving whose reply is the listener's failure line, or empty, is not
read: there is no reply of the agent's to hold to account. The reader
runs after the delivery is confirmed. The reply is already posted, so a
slow or failed reader delays nothing the person sees.

## Configuration

The reader's endpoint is a host fact, like the proxy relation: one file,
`~/.config/agag/claims.toml` (or `$XDG_CONFIG_HOME/agag/claims.toml`;
`AGAG_CLAIMS_CONFIG` names another). Every listener on the host reads it,
so Front, autolab, forge, archsage and cagent need no change to their
`agents.toml`:

```toml
[reader]
url = "http://<host>:11434"      # ollama
model = "qwen3.8:27b-mxfp8"
timeout = 60
```

A missing file turns the check off. Every serving is then `unchecked`
with "no reader configured", visible in the status file. A test double
replaces the reader in pyagag's tests, so no test needs a model.

## Missed detections, visible in tests

Tests will state the known limits:

- a claim the reader does not extract is not detected (a stub reader that
  returns nothing on a false claim leaves the serving `clean`);
- a reader that fails leaves the serving `unchecked`, never `clean`;
- a send claim whose target cannot be matched to a conversation is
  judged only by whether the window holds any send at all.
