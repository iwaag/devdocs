# agent_guide p3 — step 1: the three-arm comparison

Six Front probes were run under three compositions, from agfront's main
checkout (agfront `9f64b4b`, pyagag `aa2acce` installed):

| arm | flags | desk prompt (delegate-answer dry run) |
|---|---|---|
| before p1 | `--guides-rev 'ba28e90^' --no-shared` (= `1ef83f5`) | 27 221 chars |
| p1 | `--guides-rev 447bb03 --no-shared` | 19 500 |
| now | none | 20 596 |

That is 72 runs: 6 per arm for the two delegation probes and 3 per arm for
the others. The three arms ran in parallel, each arm's runs one after
another, and every trial had its own board and records root. Spent:
**$18.77**. The relay's cost gauge showed the same record counts and the
same day totals before and after, so no trial reached a live record.
Runner, outcomes and the two tally scripts are in
`pj-agdev/.local/agp3/` (ignored): `run_s1.sh`, `s1/<arm>/<probe>-<n>/`,
`tab.py`, `classify.py`.

## Results

"Pass" is the probe's rule. "Substance" is my reading of the tool calls
and reply behind each failing verdict (ex1's lesson: a verdict can look
right and be wrong, both ways).

| probe | arm | pass | substance | turns | cost |
|---|---|---|---|---|---|
| `aisvgs-sufficient` | before p1 | 1/3 | 3/3 | 10–12 | $0.77 |
| | p1 | 3/3 | 3/3 | 10–11 | $0.59 |
| | now | 3/3 | 3/3 | 5–17 | $0.88 |
| `growbox-thing` | before p1 | 2/3 | 3/3 | 9–14 | $0.80 |
| | p1 | 3/3 | 3/3 | 7–9 | $0.50 |
| | now | 3/3 | 3/3 | 9–12 | $0.58 |
| `forge-protoprey` | before p1 | 3/3 | 3/3 | 5–8 | $0.53 |
| | p1 | 3/3 | 3/3 | 6–11 | $0.43 |
| | now | 3/3 | 3/3 | 9–12 | $0.44 |
| `hold-release` | before p1 | 3/3 | 3/3, claims `clean` | 3–4 | $0.37 |
| | p1 | 3/3 | 3/3, claims `clean` | 3–7 | $0.40 |
| | now | 3/3 | 3/3, claims `clean` | 3–5 | $0.36 |
| `delegate-answer` | before p1 | **1/6** | 1/6 | 9–13 | $1.53 |
| | p1 | **2/6** | 2/6 | 8–18 | $1.41 |
| | now | **4/6** | 4/6 | 10–16 | $1.79 |
| `delegate-decision` | before p1 | 5/6 | 5/6 | 3–20 | $2.56 |
| | p1 | 5/6 | 5/6 | 11–23 | $2.42 |
| | now | 4/6 | 5/6 | 9–19 | $2.44 |

Four failing verdicts are passes in substance, each a spelling the rule
does not hold:

- `aisvgs-sufficient` before p1, runs 1 and 3: the replies found the
  study, rounds 1–2, the round-3 plan and the Developer's decision. They
  failed on `must_not: 教えてください`, which here is a closing courtesy
  ("進捗があれば教えてください"), not a request for the Developer to
  explain the study.
- `growbox-thing` before p1, run 1: a full, correct state of the study,
  but it never writes the literal `pj-growbox`.
- `delegate-decision` now, run 5: the whole loop in three servings, but it
  sent "12 時間/日" (with a space), which `sends_must` does not spell.

The rules were left as they are, so that the counts stay comparable with
ex1's.

## Delegation, by what the run did

The verdict does not say *why* a delegation run failed. Each run is
classified by its acts:

- **reached**: a send that the fixture's autolab would be served by, and
  the answer came back on a serving of its own;
- **wrong door**: a send that no listener is served by. That is a new
  topic in `#pj-growbox` without a `workplan-` prefix, with `--to autolab`
  and no mention;
- **none**: nothing sent.

| probe | arm | reached | wrong door | none |
|---|---|---|---|---|
| `delegate-answer` | before p1 | 1 | 2 | 3 |
| | p1 | 2 | 2 | 2 |
| | now | 4 | 1 | 1 |
| `delegate-decision` | before p1 | 5 | 0 | 1 |
| | p1 | 5 | 0 | 1 |
| | now | 5 | 0 | 1 |

What the runs that did not reach autolab did:

- **`delegate-answer`, none (6 runs across the arms)**: every one read
  `✔ workplan-growbox-germination-days` and answered "about 45 minutes, no
  stall on record" or "記録上は特に詰まった箇所はなく". Some gave reasons:
  - before p1, run 1: "この情報は既に確定した記録から読み取れたので、autolab
    に改めて聞き直してはいません（解決済みトピックへの再投稿は新しい作業を始めてしまうため）".
    That is fd-wr's sentence, read as a reason not to ask anyone at all;
  - before p1, run 5: "改めて autolab に聞き直す必要はありませんでした";
  - p1 run 5 and now run 2 point at the report file for "the details"
    instead of the agent.
- **`delegate-answer`, wrong door (5 runs)**: each run first answered what
  the record says, then asked autolab what the record lacks (a delegation
  in intent). But it asked in a topic of its own in `#pj-growbox`
  (`m20390-recap`, `germination-days-timeline`, …), with only `--to`. Four
  of the five had read `agentchat intro autolab-agstudio1` first. The
  fixture's autolab introduction names only one door, the `workplan-`
  topic for *work*. The live introduction also says "**Questions about me
  go in `autolab-agstudio1`** … ask about my work there and I answer"; the
  fixture dropped that sentence. So this shape is partly the fixture's (see
  step 2).
- **`delegate-decision`, none (one per arm)**:
  - p1 run 3 and now run 4 read that m20402 is running and replied "if
    autolab asks us to choose, I will answer the cheaper one". They left
    the Developer's "autolab に決めてもらって" unasked. That is as9's shape on
    running work;
  - before p1, run 5 proposed before acting ("この進め方でよければ、そのまま
    進めます"). That is the old guide's "propose before acting" branch,
    where the Developer had already said to go ahead.
- **Tried the ✔ topic first**: now runs 1 and 3 of `delegate-answer`
  first sent into `pj-growbox › workplan-growbox-germination-days`, the
  finished topic's bare name. `agentchat send` refused it (the overlay
  recorded nothing), and both then asked in a new topic that reached
  autolab. So fd-wr's sentence does not stop the attempt in every run. The
  tool's refusal does, and the refusal's text points at the introduction.

## The baseline's limit

`--guides-rev` restores the guide tree only. Every arm ran over the tools
installed now:

- pyagag `aa2acce`'s `agentchat --help` (the 117-line index) and every
  subcommand help;
- `read` taking several topics;
- the current reply and continuation sections;
- `agentchat send`'s refusal of a resolved topic.

So "before p1" measures **the old guides over the new tools**. That was
not changed here, for two reasons. The difference that matters is visible
in the guide text itself (step 2). And all three arms fail the delegation
in the same ways, so older help texts could only add failures of their own
to the before-p1 column.

## `aisvgs-sufficient` compared with run-0160

- run-0160 failed on a filesystem `find` that the harness refused, and then
  made no `agentchat` call.
- Before p1, 0 of 3 runs did that. All three began with the working
  directory (`ls -la && find . -maxdepth 3`, or the `tools/` files), and
  then read the board with 6–7 `agentchat` calls.
- Across all 24 before-p1 runs, 10 reached for the filesystem, one of them
  `find / -iname "strand-germination-days.md"`. None was refused, and every
  one went on to `agentchat`. p1 did this in 1 run of 24 and now in 0.
- So p1's "the board is Zulip, reached by `agentchat` — not the filesystem"
  shows in the calls. run-0160's dead end was not reproduced. One
  difference: today's `agentchat --help` (installed for every arm) is the
  index, which may make the board easier to find than it was for run-0160.

## Observed: the Developer's round-3 decision (p1 open finding 1)

The decision recorded only in `archsage-agstudio1 › study-aisvgs-round3`:

| arm | carried it |
|---|---|
| before p1 | 3/3 |
| p1 | 0/3 |
| now | 1/3 |

This is observed, not required, and it concerns p1's open finding 1, which
the Developer holds. It is reported here without a conclusion.
