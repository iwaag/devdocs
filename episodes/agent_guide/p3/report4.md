# agent_guide p3 — step 4: p7's reader measurement, versioned

failsafe p7 chose its claims reader by running 13 cases through the host's
local model four times. The script and three reply samples stayed in
`pj-agdev/.local/failsafe-p7/`, all ignored. It is now in pyagag:

```
python -m agag.fixture reader [--runs 4] [--model <name>] [--url <ollama>] [--json <file>]
```

- **Where**: `agag/fixture/reader.py`, with the cases as data in
  `agag/fixture/reader_cases.json` (pyagag `051bb67`, tests in
  `tests/test_fixture_reader.py`).
- **What it reads with**: the listener's own reader. That is
  `agag.claims.OllamaReader` with `READER_SYSTEM`, given
  `reply_words(text)` (the words without the leading mention and the
  `ag-post` line). p7's script carried its own copy of the prompt and gave
  the model the raw reply. So a reading here is exactly what a listener's
  check would get, and a change to the reader's prompt is measured by the
  same command.
- **Which reader**: the one `~/.config/agag/claims.toml` names.
  `--model` and `--url` measure another one before the host is switched
  to it.
- **The three real replies** (#15838, #15844, #15851) became synthetic
  equivalents: the same shape and wording, fixture ids (#20835…#20850) and
  a fixture project (the pj-growbox lighting schedule). Nothing exported
  from the realm is committed.
- **A case is right** when the set of acts read equals the set wanted.
  Targets and quotes are kept in `--json` but not judged, because the judge
  (`agag.claims.judge`) matches targets against records, and this probe has
  no records.
- **A failed reading** is counted as an error, never as an empty (and so
  right-looking) list.

## The result against today's reader

`qwen3.8:27b-mxfp8` at the host's ollama, 13 cases × 4 runs:
**44 of 52 readings right, 0 errors, 0.29–3.68 s per reading**. Every case
gave the same answer all four times. JSON:
`pj-agdev/.local/agp3/reader-s4.json` (ignored).

| case | want | read (4/4) |
|---|---|---|
| hold-recorded | hold | hold |
| false-release (run-0183's shape) | release, disposition | release, disposition |
| repair | release, disposition | release, disposition |
| in-force, quoted, ja-none, offer | — | — |
| ja-send, send-en | send | send |
| ja-release | release, disposition | release, disposition |
| accept-ja | accept | accept |
| **restate** | — | release, disposition |
| **other-agent** | — | accept |

This is p7's measurement again, with one difference:

- **The two misses are p7's.** p7's table has the restatement and
  "autolab recorded the acceptance" over-read 4/4, and its step 1 rule 2
  ("already on record") absorbs both in the listener, because each names a
  target on record. The case file keeps "—" as their want, because that
  is what a better reader would answer. So 44/52 is the number to compare a
  new model against, not a failure.
- **p7's third miss is gone.** p7 read a `send` in #15851 from its
  `ag-post … re=15847` line, because the script gave the model the raw
  text. The listener has given only `reply_words` since p7. The probe now
  does the same, and the repair case reads clean.
- **Latency is p7's range**: 0.3–3.9 s then, 0.29–3.68 s now, with the
  model loaded.
