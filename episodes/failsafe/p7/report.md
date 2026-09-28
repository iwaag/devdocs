# failsafe p7 — report: a reply that claims an act it never did

## Outcome

Every delivered reply of every agent is now checked against the records
its serving wrote. A mismatch is noticed in the serving's own
conversation, and the agent that claimed the act is served once, with the
notice as the trigger, to record it or correct the reply. A second
mismatch goes to the owners through Observer. No person has to compare a
reply with the records. That comparison is what the Omni Agent did by hand
in #15847.

Live on pyagag `aa2acce` since 2026-09-28 18:08Z. On the second live trial
the listener caught a false reply in 5 s. Front was served once with the
notice, recorded both acts on the person's own post, and the claim was
settled `recorded` 30 s after the false reply. Nobody was asked anything.

Step reports:

- `report1.md` — the claim contract;
- `report2.md` — how a claim is known;
- `report3.md` — what a mismatch does;
- `report4.md` — implementation;
- `report5.md` — trials and roll-out.

## The claim contract (report1)

- **The acts and their records.** `hold` → `[hold]`, `release` →
  `[hold-release]`, `disposition` → `[disposition]`/`[disposition-reversed]`,
  `relation` → `[relation]` (or a root note with `rel=`), `accept` →
  `[acceptance]`/`[state] accepted|done`, `reserve` → `[approval]`,
  `receipt` → `[receipt]` (or a run-written `[served]`), `send` → a post in
  another conversation.
- **A serving's records are its window**: the agent's bot's posts between
  the serving's ack (or its start) and its reply. A listener serves one
  conversation at a time and a run posts with the agent's credential, so
  the window is exact whatever the harness. It is read off the listener's
  mirror, with no transcript and no Zulip call. Memo posts are excluded.
  run-0183's window (#15843 → #15844) held nothing, while run-0182's and
  run-0184's held exactly their records.
- **A claim is true** when the window holds a record of its kind, or when
  one is already on record: naming its target, in the target's
  conversation, or (no target) in the reply's conversation. A restatement
  therefore raises nothing.
- **The proxy case**: the check looks at who posted the record (the
  claiming agent) and at its kind and target. The `by 9 … for 8` authority
  is the tools' check (`acts_for`). Pinned by a test.

## The option chosen (report2)

**A local reader extracts, code judges.** The host's local model
(`qwen3.8:27b-mxfp8`) gets only the reply's words and lists the acts it
claims as `{act, target, where, quote}`. The comparison with the records
is deterministic.

- **Option 1 (the reply declares its acts) was rejected.** A declaration
  is the agent's own account of what it did, which is what failed, and it
  would need guide text that p2 could not show helps.
- **Option 3 (a decision trigger without a record) was rejected.** Knowing
  a trigger is a decision means reading prose in the Developer's language.
  Its cheap proxy fires on questions, and every false alarm buys a paid
  run.

Measured before building: 13 cases × 4 runs, 0.3–3.9 s per reply,
deterministic. The one systematic misreading (the `ag-post` machine line
read as a send) is why the reader gets the words only.

## What a mismatch does (report3)

- `[selfnote][claim] {json}` in the reply's conversation (what was
  claimed, quoted; what is missing; what was found), and the owner's own
  `[selfnote][start] #<claim> for <requester>`. The notice is the trigger;
  the stale decision is not.
- Every conversational prompt carries the open claims
  (`prompt_with_guide`, no guide text): the quote, the record missing, the
  tool that makes it, and "do it now or say plainly it was wrong".
- The answering serving settles it: `recorded` when the records now exist,
  `corrected` when the reply no longer claims them. A repeat is attempt 2,
  with no start note: one mismatch buys one serving.
- Trace and panel show an open claim as owed (`! claim #…`, "owed now"; a
  card reads `waiting` with the claim as its reason). The trace's `claim`
  candidate takes an escalated claim, or one nothing answered within
  600 s, to Observer, which reports it to the owners at once and asks
  nobody. It recovers only on the settlement record.

## Trials (report5)

| trial | runs | result | cost |
|---|---|---|---|
| deterministic reproduction: run-0183's output through agfront's desk serving on the fixture, real reader | 2 servings, no model | mismatch (release #15837, disposition), claim and start written, notice only in serving 2's prompt, settled `corrected` | $0 |
| same, over the real listener on the fake realm (pyagag test) | — | the repair's `trigger_id` is the start note; its records settle it `recorded` | — |
| false alarms: still in force, quoted command, restatement, Japanese status (stub replies, real reader) | 4 | 4 clean, none served again | $0 |
| fixture `hold-release`, model (Front desk, `claude-sonnet-5`) | 6 | probe 6/6. Claims: 5 clean (release + disposition read, both in the window), 1 skipped (an unusable mark; the kit, unlike the listener, runs no repair) | $0.77 |
| fixture, run-0183 replayed then the model answers the notice | 3 × 2 | 3/3: the notice made Front run `release` and `disposition`, settled `recorded` | $0.45 |
| live 1, correct path (`front-desk-20260929-p7-claims`) | run-0185, run-0186 | hold, then release + disposition: 2 checks, both clean, 4–5 s after each reply | $0.27 |
| live 2, repair path (`front-desk-20260929-p7-false`, `faults/false-claim`) | run-0187, the fault serving, run-0188 | claim #15877 at 18:12:45; the repair serving triggered by start #15878; records #15880/#15881; `[claim-settled] #15877 recorded #15882` at 18:13:15; trace and panel open, then clean; Observer no incident | $0.23 |

**False-alarm rate on the fixture: 0 of 5 checked correct replies** (plus
0 of 4 written false-alarm replies, and 0 of 3 repair replies). Live:
**0 of 4** correct replies. Total model cost of the phase's trials: about
$1.72.

Suites after the pin: pyagag 1187, agobserver 180, agfront 198 (199 with
the fault's test), agautolab 334, agforge 265, archsage 41, cagent 204,
relay 363.

## README_DEV

- New section *A reply that claims an act it never did*, after *Work
  relations and dispositions*, which sits beside receipts, holds and
  dispositions. It covers the window, the acts table, the reader and the
  host file, what a mismatch does, how it is shown, the tools, trial aids
  and limits.
- The Observer section has the `claim` kind.
- *Trying a guide change* describes the claims probe and `--replies`.
- Host notes are in `pj-agdev/.local/devenv.md` (*failsafe p7 on this
  host*).

## Limits and what is left open

- **Detection is only as good as the reader.** A claim it does not extract
  is missed, and a test pins that. The measured misses so far are
  over-readings, which the "already on record" rule absorbs. No real false
  claim has yet met the live reader; its rate on real traffic is
  unmeasured, and so is the rate of false claims itself (p2: 1 in 13 on
  one shape).
- **What is checked.** Acts that leave no realm record (files, commits,
  commands) are not checked. autolab's task replies are literal sections,
  not model replies, and are not read (its planner's are).
- **Who hears about it.** Observer reports only claims inside requests
  that came through `#front`. A claim elsewhere is recorded and shown by
  the trace, but nobody is told.
- **Load.** The reader shares the host's model with Observer's evaluations
  and triage. A busy model delays a check; it never skips one (`unchecked`,
  retried).
- **Pins.** comfynotify keeps its pin (no listener, no reply), and
  agautolab1 (VM) was not redeployed.

## Interventions (Deus Ex Machina note)

- **The Omni Agent as the Developer's stand-in**: the four trial posts
  (#15856, #15861, #15869, #15874). The trial conversations are left open
  as the record; nothing was resolved for Front.
- **Arming `faults/false-claim`** for live trial 2 is a trial aid, not work
  of Front's.
- **The host's reader file** (`~/.config/agag/claims.toml`) was written by
  the Omni Agent. It is operator configuration.
- **Handoff delivered.** What the Omni Agent did by hand in the incident
  (#15847: "no record exists; record them") is what the listener now does
  by itself. No new handoff candidate came up in this phase.

## Where it lives

- pyagag:
  - `e39956c` (`agag.claims`, the listener's check, trace, progress, agent
    wiring);
  - `aa2acce` (`check_served` shared by the listener and the trial kit; the
    claims probe; `--replies`).
- agfront `81184e7` (trial driver), `9f64b4b` (`faults/false-claim`).
- pj-agdev `4356863` (agobserver `claim` incidents), `99fdd63` and
  `621fcea` (pointers).
- pj-clusterintent `6afea1d` (cagent), `7b09068` (pin).
- Pins: agfront `81efd04`, agautolab `60e666f`, agforge `60e88fb`, archsage
  `11e2d40`, relay `33f3da2`.
- Kit (ignored): `pj-agdev/.local/failsafe-p7/`.
