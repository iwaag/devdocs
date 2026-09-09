# step5 — verifying the complete workflow in the experiment environment

**Commits:** pyagag `e951079` (the defect this step found), agautolab and
agfront lock updates. Everything below happened on agstudio on
2026-09-09 between 16:55Z and 17:12Z.

## Service state, before anything was touched

`pj-clusterintent/nctl`, read-only:

    ✓ nautobot   http://localhost:8000 — 3.1.3, authenticated, intent catalog present
    ✓ worker     celery workers: 1, pending jobs: 0
    ✓ dumps      /var/lib/nodeutils (8 hosts; agstudio collected 4.9 h ago)
    ✓ submodule  ansible_agdev / nauto / nctl / nintent / nodeutils — all clean
    ok: True

Every agstudio launchd job was up (`agautolab-zulip`, `agfront-zulip`,
`agforge-zulip`, `arxivsage-zulip`, `agentroom`, `comfy-notifier`,
`agautolab-gateway`, `agforge`) and the relay answered `/healthz` and
`/budget`. **Neither listener had a harness child**, checked with `ps` rather
than assumed, so the restarts below killed nothing in flight.

The live budget at that moment: claude_code 5-hour 63 %, weekly-all 50 %,
weekly-Fable 77 %; codex 5-hour 37 %; **agy 0.8 % weekly, 0 % 5-hour**. The
demonstration was deliberately put on agy, whose pool had room — which is the
whole point of being able to ask.

## Tests and locks

`uv run pytest`: pyagag **530**, agautolab **234**, agfront **103** — all
passing, no paid harness on any of them. agautolab and agfront were relocked
onto pyagag `e951079` and their suites re-run against it. agforge and
arxivsage were **not** relocked: they use no part of this contract, mixed
pyagag revisions across agents are normal here, and an unnecessary bump is an
unnecessary risk.

Both listeners were restarted (`launchctl kickstart -k`; the plists were
unchanged, so `-k` is enough) and both introductions re-posted. Discovery then
worked from the realm, through the ordinary tool:

    $ agentchat options
    agforge-agstudio1: unknown — publishes no execution options block
    arxivsage-agstudio1: unknown — publishes no execution options block
    autolab-agstudio1: @**autolab-agstudio1** use <option>
        `default` (…; pool `anthropic`; covers my entrance, mission planning, task work and brain-mining)
        `agy` (Antigravity CLI (`agy`), Gemini 3.8 Flash; pool `antigravity`; …)
        `agy-claude` … `codex` … `gemini` …
    front-agstudio1: @**Front** use <option>
        `default` … `agy` … (pool `antigravity`; covers my own conversations…)

Two agents that predate the contract are reported as **unknown**, which is the
answer the contract insists on, and the two command lines differ in exactly
the way the block exists for: autolab is mentioned as `autolab-agstudio1`,
Front as `Front`.

## The defect the live check found

The plan says *inspect actual run records*, and that is what caught it.

autolab's first real agy run — an entrance question in
`autolab-agstudio1 › rp-exec-check` — came back correct in every visible way
(`profile: agy`, `harness: agy`, `model: antigravity/gemini-3.8-flash-medium`)
and its record on disk carried **no `exec_option` at all**.

`agag.harness.write_run_record` copies an **allowlist** of keys out of a run's
`meta`. Every step-2 test asserted on the dict `run_role` *returns*, which did
carry the fields; nothing asserted on the file — and the file is what the plan
says to inspect and what the cost gauge reads. So "why did this run on agy"
was unanswerable from the record that §7 of the contract exists to make
answerable.

Fixed in pyagag `e951079`, with a test that reads the file back
(`test_the_record_on_disk_carries_the_execution_option`) and a comment at the
allowlist saying what it costs to add a field and forget it. A green suite did
not find this; one real record did.

## Command handling, live and free

In `autolab-agstudio1 › rp-exec-check`, posting through the ordinary tool:

| # | posted | what happened |
|---|---|---|
| 5539 | `@**autolab-agstudio1** use agy` | confirmed in the same second, **no ack, no run** — newest run record still 14 h old |
| 5541 | `@**autolab-agstudio1** use opus` | refused: *"I do not publish an execution option named `opus`. Mine are: `default`, `agy`, `agy-claude`, `codex`, `gemini`. This topic still runs on `agy`."* — naming the poster, and still no run |
| 5543 | a question | ran, on agy |

The refusal names the poster (so they learn it) and states the standing
selection (so they do not guess again); the confirmation names nobody. Both
are the owner's own post, which is what settles the topic for the sweep.

## The end-to-end run

`#front › front-rp-20260909-1702`, the request made in words: *"ask autolab to
plan one very small mission on ghtrends, and have autolab run that work using
agy"*, with the instruction to **find out** what autolab publishes rather than
assume a name.

Front, unprompted by any name in its code:

1. read `agentchat options` and reported *"autolab publishes an option
   literally named `agy` — pool `antigravity` … That's the real name; I'm not
   guessing"*;
2. asked permission (its standing rule), then posted
   `@**autolab-agstudio1** use agy` into `pj-ghtrends › workplan-devlog-note`
   (message **5558**) — autolab confirmed and started nothing;
3. posted the mission request separately (message 5560);
4. was called back when autolab answered, and reported the plan, the option,
   **the selection's message id 5558**, and where the task topic is.

Then the records, which are the point:

| run | role | profile / harness / model | exec fields |
|---|---|---|---|
| `entrance_front/run-0026` | front (entrance) | agy / agy / antigravity/gemini-3.8-flash-medium | `exec_option: agy`, `exec_source: topic`, `exec_message_id: 5539` |
| `superdirector/run-0167` | **planning** | agy / agy / … | `exec_option: agy`, `exec_source: topic`, `exec_message_id: 5558` |
| `supercoder/run-0232` | **task work** | agy / agy / … | `exec_option: agy`, `exec_source: **inherited**`, `exec_message_id: 5558`, `exec_inherited_from: pj-ghtrends/workplan-devlog-note` |
| `entrance_front/run-0027` | front (entrance) | **sonnet / claude_code / anthropic/claude-sonnet-5** | `exec_source: default`, no `exec_option` |

Four things are settled by that table:

- **The option covers planning and task work, not merely the entrance
  response.** That was the plan's explicit worry, and the superdirector and
  supercoder rows are the answer.
- **`exec_message_id: 5558` is the message Front posted**, and Front's own
  report to the developer names 5558 independently. The record's provenance
  and the agent's account of itself agree without either reading the other.
- **Inheritance is real and is a snapshot.** The child topic carries, before
  its visible description:

      [selfnote][exec] agy from pj-ghtrends/workplan-devlog-note#5558

  and the task run resolved from it as `inherited`.
- **An unrelated topic kept its own setting.** `rp-exec-default`, opened with
  no command, ran on `sonnet`/`claude_code` — the same agent, the same minute,
  a different backend, `exec_source: default`.

The mission itself completed: Work G-21 with sub-work G-22, the devlog note
written and pushed, the task topic resolved by autolab, and G-21 closed with
`python -m agautolab.mission_done`.

## Controlled budget observations

`agbudget --source <fixture>` through agfront's own installed console script —
the same renderer a run reads — on the five fixtures:

| fixture | what it shows |
|---|---|
| `agy.json` | `## agy (plan pro, pool antigravity)` at **71 % used** — "exceeds 70 %" already reached, on the pool the `agy` option names; `## gemini_cli (pool google) — READ FAILED` — unknown, and no `0 %` anywhere in that section; `## agcode (pool unknown)` — matchable to no option at all |
| `reached.json` | 57 %, condition met before any work starts |
| `below.json` | 31 %, not reached |
| `reset.json` | 72 % read at 14:45, reset passed at 14:50 — *"this reset has already passed since the read: the window shown is over, current usage unknown until re-read"* |
| `failed.json` | READ FAILED, last good numbers marked **STALE** |

Every section is now labelled with its pool, which is what lets a threshold
phrased about an execution option ("agy's usage") be matched to a window.

## Fixture evidence vs. harness execution — the honest split

The plan asks for this distinction, so:

- **Actual harness execution**: the entrance run on agy, the entrance run on
  the defaults, the planning run on agy, the task run on agy, and Front's own
  runs (claude_code) that discovered, selected and reported. Real money on
  real accounts, real records on disk, quoted above.
- **Fixture evidence**: the five budget observations (no run judged against
  them live), and **autolab's callback continuation**. A callback needs a task
  that delegates to another agent, and this deliberately tiny task delegated
  to nobody; the mechanism is pinned by
  `test_a_callback_runs_on_the_tasks_selection_not_the_remote_topics`
  (autolab) and `test_a_callback_answers_home_under_homes_selection`
  (agfront), and Front's own callback continuation *was* live — it was served
  again by autolab's answer and reported correctly.

  What is live-proven for a task topic is the weaker but adjacent property:
  the same topic served twice keeps its selection (`rp-exec-check` runs 0025
  and 0026, both `exec_message_id: 5539`).

## An ops mistake worth recording

Closing the mission topic, I posted a courtesy line into
`pj-ghtrends › workplan-devlog-note` *and then* resolved it. The post made me
the last speaker, autolab was served, and the serving failed on a Plane
`HTTP 429 RATE_LIMIT_EXCEEDED` during read-back. No model ran and nothing was
harmed — but this is exactly the realm's known "a post in an agent's topic
buys that agent a run" trap, walked into by the Omni Agent while tidying up.
Resolve without speaking, or say the closing line in the conversation that
asked.

*Deus Ex Machina note: the Omni Agent drove this whole verification —
posting the commands, starting the task, closing Work G-21 — where a
developer or Front would normally do it. Handoff candidate: the live
exercise of a new contract is a routine an agent could run.*

## Commits

- pyagag `e951079` — the record-writer fix.
- agautolab, agfront — lock updates onto it.
- devdocs — this report.
