# agent_guide p2 — step 6: a fixture board for repeatable trials

pyagag `9e33581`, agfront `d8bc699` on `agent-guide-p2`.

## The choice: a mirror store, not a disposable realm

The plan offered two ways: point a trial's `agentchat` at a mirror store
built from the fixture, or seed a disposable realm. The store was chosen:

- It needs no Zulip at all, so it cannot drift, and no trial post is made
  (p1's trials needed the Developer to approve each post by hand after the
  auto-mode classifier refused them).
- Step 5 already gives `agentchat` a mirror-backed client, so a run reads
  the fixture with the same commands it uses on the live board, through
  the same code path.
- A store marked `fixture=<name>` has **no live side**: every write, and
  every read the store cannot answer, fails with a line naming the
  fixture. A trial run cannot reach the real realm by accident.

## What the board holds

`agag.fixture.board` builds it in code, post by post, in the shapes this
board has (root notes, acknowledgements, `ag-post` lines, ✔ notices) with
words written for it; nothing was exported. `python -m agag.fixture build
<dir>` writes `<dir>/mirror.sqlite` from scratch, identical every time.

| channel | what it holds | for |
|---|---|---|
| `#agents` | six introductions (Front, autolab, forge, archsage with its sages, Observer, cagent) | `agentchat intro`, `tools/agents.md` |
| `#pj-aisvgs` | the plan, the setup, rounds 1 and 2 (✔, accepted, `main` at `f57eed1a27de`), the round-3 plan not started | the #15673 probe |
| `#routine-study-aisvgs` | the guide and two finished runs | same |
| `#archsage-agstudio1` | refresh topics (sagesync records), and `✔ study-aisvgs-round3`: **the Developer's decision to do round 3 by hand** | the fact p1's run-0169 missed |
| `#pj-growbox`, `#routine-study-growbox`, `#work-m20402` | two strands accepted (germination, food safety), the control-loop mission running, a run waiting on it | "the grow box thing" |
| `#pj-protoprey`, `#agforge-agstudio1` | v0.1.0 done, v0.2 waiting for the Developer's go-ahead; forge's hero sprite and meadow delivered, footstep sounds **not** delivered (no sound-effect toolset; forge asked which way), an unrelated birthday card | forge's past work; Observer's triage |
| `#pj-worldtrend` | a plan only | a study with nothing yet |
| `#front` | `✔ front-desk-20260926-1200`, one past request answered and resolved | a finished request |

## How a probe is run

```
python -m agag.fixture build <dir>                      # the board
python -m agag.fixture probes                           # every probe and its rule
cd agfront && .venv/bin/python -m agfront.trial <probe> --store <dir>/mirror.sqlite --out <out>
      [--guides <another guide tree> --no-shared]       # e.g. the live checkout's: the baseline
      [--dry-run]                                       # write the prompt, run nothing
python -m agag.fixture check <probe> <reply file>       # judge any reply
```

`agfront.trial` calls agfront's own `serve` with the fixture as the
client, so the evidence chatlog, `tools/agents.md` (harvested from the
fixture's `#agents`), `tools/runs.md` and the prompt are what a first desk
serving would get. Inside `agag.fixture.run.fixture_environment`, every
run started through `agag.agent` gets `AGENTCHAT_MIRROR` pointed at the
fixture. `--guides` and `--no-shared` reproduce the composition before this
phase (the live checkout's guides without pyagag's sections), so **the same
question against the same board can be asked with the old guide and the
new one**. The outcome — the marked reply, its verdict, the run record's
cost, turns and duration, and the tool calls from Claude Code's session
log — goes to `<out>/outcome.json` and `reply.md`.

The other agents' probes (archsage, autolab's planner, Observer's triage)
are served the same way from each agent's own composition; step 7 runs
them.

## The probes and their pass rules

Each rule is facts the reply must carry (any one spelling of each) and
words it must not carry; `judge` says which were met. Rules are about the
board's content, so one rule judges both guides.

| probe | role | asks | passes when the reply carries |
|---|---|---|---|
| `aisvgs-sufficient` | Front desk | #15673's wording, word for word | the study (`pj-aisvgs` / routine / sage), round 2, round 3, and the Developer's decision to do round 3 by hand; not "no record" or a request for the channel |
| `growbox-thing` | Front desk | "あのgrow boxのやつ、いまどこまで進んでる？" | `pj-growbox`, germination, food safety, the control loop (m20402); not a question back |
| `forge-protoprey` | Front desk | "forgeってprotoprey向けに何を納品したんだっけ？" | hero, meadow, the undelivered footsteps; not the birthday card |
| `archsage-loose-sage` | archsage | "the sprout-box sage" | `sage:growbox`; not "which sage" |
| `planner-other-project` | autolab planner in `pj-protoprey` | where the growbox study stands | growbox, the control loop, a finished strand; not "cannot see" |
| `triage-unopened` | Observer triage | `#pj-protoprey › workplan-protoprey-locations` (autolab asked the Developer; no answer) | `legit` |

## Checked

- `agentchat intro`, `channels --prefix pj-`, `topics`, `read --latest`
  and `agproject status growbox` answer from the fixture; `send` is
  refused ("this board is the fixture 'agent-guide-p2' … nothing is posted
  or read elsewhere"). One thing is not fixture-bound: `agproject status`
  also asks the host's Gitea about the study's repository, which is real.
- A dry run builds both compositions from the fixture: 20 630 characters
  with the new guides and sections, 19 534 with the live guides.
- **One real run**: `forge-protoprey` with the new guides — passed, 10
  turns, $0.16, 19 s. Its reads were `agentchat intro forge` (refused with
  the list of names), `channels --prefix pj-`, `intro agforge-agstudio1`,
  `topics pj-protoprey`, `agproject status protoprey`, `topics
  agforge-agstudio1`, three `read`s. The reply lists both deliveries with
  their message ids, keys and links, and the footsteps request as not
  delivered with forge's two options, and leaves out the birthday card.
- Tests: `tests/test_fixture.py` (8: the board is identical every time,
  the probes' facts are on it, six introductions and the reader identity,
  no post, a run started inside reads the fixture, rules and judging, the
  marked reply). pyagag **1153 passed**.
