# Failsafe p7 — a reply that claims an act it never did

## Goal and scope

A serving can reply that it recorded something — a hold released, a
disposition, an acceptance, a request sent to another agent — without
having done it. Today nothing in the system notices; the Omni Agent did,
once. Make the system notice, within the serving's own conversation, and
bring the act about or the claim corrected, without a person having to
compare the reply with the records.

Read `devdocs/episodes/agent_guide/p2/report7.md` § *A hold placed and
released* (the incident) and `../p6/report.md`, `../p6/ex2/report.md`
(holds, dispositions, the proxy). `devpolicy/terms.md` for Shackle and
Tool Giving: the check belongs to the system, not to more guide text. p2
tried the guide side and could not show a sentence helps against a
1-in-13 event.

Private experimental, breaking-change phase: no backward compatibility.
The implementer chooses where the check lives, what it compares, and what
it does on a mismatch. Keep the check cheap enough to run on every serving
that could make such a claim.

## The incident

`#front › front-desk-20260928-agp2-hold`, agfront `desk`
(`claude-sonnet-5`).

| post | who | what |
|---|---|---|
| #15835 | Omni (the Developer's proxy) | keep pj-protoprey v0.2 on hold |
| #15837 | Front, run-0182 | `[selfnote][hold] decision a15835 …` (real) |
| #15842 | Omni | "I release the hold; stop following this request" |
| #15844 | Front, run-0183: **1 turn, no tool call**, $0.07 | "Released hold #15837 … Recorded the disposition as `withdrawn` …" — nothing recorded; the syntax it quoted does not exist |
| #15847 | Omni | no record exists; record them |
| #15849–#15851 | Front, run-0184 | the real `hold-release` and `disposition` notes |

Observer saw only the records: a hold in force, so it would never have
asked. The fixture probe `hold-release` (pyagag `agag.fixture.probes`)
recorded both acts 12 of 12 times, so the event is rare and model-side.

## step1 — define the claims and where their evidence is

- List the acts a reply can claim and the record that proves each. Start
  with: hold, release, disposition, relation, accept, reserve, receipt
  repair (pyagag `holds.py`, `dispositions.py`, `relations.py`,
  `acceptance.py`, `receipt.py`), and `agentchat send` to another agent
  (a post in that agent's conversation). The last is the same failure
  class — Front's guide asks it to report where it posted — and probably
  the costlier one.
- For each, say how a checker finds the record a given serving wrote:
  most are selfnotes by the agent's own bot in the home conversation,
  after the trigger and before the reply (`Serving.trigger_id`,
  `delivered_id` in the run record's `outcome`). `send` lands elsewhere;
  the transcript's tool calls or the mirror's index by sender and time can
  find it.
- Write this as `report1.md`. It is the checker's contract.

Hints: the run record (`<instance>/.local/agent/<role>/run-NNNN.json`) has
`reply`, `num_turns` and `outcome` (`serving_id`, `delivered_id`, `end`),
but not tool calls. Tool calls are in the harness transcript;
`agfront.trial._tool_calls` already parses a claude_code transcript.
Other harnesses (codex, agy, gemini) write theirs differently; the
records themselves are harness-independent, so prefer checking records
over parsing transcripts.

## step2 — decide how a claim is known

Options, not a decision:

1. **The reply declares it.** The reply mark (`<ag-reply …>`, `agag.reply`)
   gains an optional list of the acts the reply reports, by record kind and
   target. The listener checks each against the records before or after
   delivery. Deterministic and harness-independent; misses a reply that
   claims in prose and declares nothing, so it pairs with option 2 or 3.
2. **A reader judges it.** A cheap model reads the reply and the list of
   records the serving wrote, and answers whether the reply claims an act
   missing from the list. Observer's triage already runs a local model
   (report7: "local model"), and replies follow the language of the conversation
   (the Developer often writes Japanese), so regexes over
   prose will not do.
3. **A structural signal.** A serving whose trigger is a decision from a
   decision holder (release, cancel, accept, "stop following this") and
   that wrote no record of the matching kind. Cheap, no reading of prose,
   catches run-0183 exactly, misses claims on other triggers.

Mixing them is fine. Whatever is chosen, a missed detection should be
visible in a test, not only in production.

## step3 — what happens on a mismatch

- The owner of the act is the agent that claimed it. The natural repair is
  to serve that agent again in the same conversation with the mismatch
  stated (what was claimed, what records exist), so it records the act or
  corrects its reply — what Omni's #15847 did by hand.
- A post that serves an agent costs a run and becomes its trigger
  (MEMORY: bot posts re-serve and misdirect). That is wanted here; make
  sure the notice is the trigger, not the stale decision, and that one
  mismatch buys one serving, not a loop (a second mismatch on the repair
  serving goes to Observer's review path or to the person).
- A notice that must not serve anyone (only for the record) is a
  selfnote; use one if the check only records.
- Observer already owns "work stopped" asks and review topics
  (`agobserver/{health,review,notify}.py`); if the check lives in the
  listener, Observer may still be the one that asks. Implementer decides.
- Trace and the progress panel should show an unresolved claim mismatch
  as open, the way they show an owed receipt.

## step4 — implement

- The listener's post-delivery hook is `Listener._after_delivery`
  (`pyagag/src/agag/listen.py`, ~line 1054); it already writes the served
  mark and receipts from the serving record, so it is where a serving's
  facts are at hand. Every agent's listener is `agag.listen`, so one
  implementation covers Front, autolab, forge, archsage and cagent.
- If option 1 is chosen, `agag.reply` parses the mark and `REPLY_GUIDE`
  describes it to every conversational role in one place; that is the
  only guide text this phase should need.
- Check the proxy case from p6 ex2: a release by Omni with the Developer's
  authority is recorded as Omni's words; the checker must accept that
  record as the release.

## step5 — trials

- **Deterministic reproduction**: a serving that replies with run-0183's
  text and makes no tool call. The stub profile (`harness = "fake"`,
  `[profiles.stub]` in `agfront/agents.toml`) or a test double of the
  harness gives it without a model. Pass: the mismatch is detected and the
  repair serving is triggered by the notice.
- **Fixture**: the `hold-release` probe on the fixture board
  (`python -m agag.fixture`, `agfront.trial`; agent_guide p2 ex1 may have
  moved the drivers). Extend it so the checker runs; pass: no false alarm
  on the 12-of-12 correct runs' shape.
- **False alarms**: a reply that mentions an existing record ("the hold
  #15837 is still in force") or quotes a command without claiming to have
  run it. Pass: no notice.
- **Live**: the incident's conversation shape in a fresh `front-desk-`
  topic, with a hold placed and released by the proxy. A real false claim
  is rare; the live trial proves the correct path stays quiet and cheap.
- Run every consumer suite after the pyagag pin (pyagag, agfront,
  agautolab, agforge, agobserver, archsage, cagent, agentroom); read the
  summary line. Deploy as README_DEV § *How a guide is put together*
  describes: branch, merge, pin, `uv sync`, restart listeners one at a
  time, check logs. Service state through Nautobot or `nctl status`
  (`pj-clusterintent/nctl`).

## step6 — report

- `report.md`: the claim contract, the option(s) chosen and why, what a
  mismatch does, trial results with run numbers and cost, false-alarm
  rate on the fixture, and README_DEV updated where it describes receipts
  and holds. Note anything Omni did for an in-system agent as a handoff
  candidate.
