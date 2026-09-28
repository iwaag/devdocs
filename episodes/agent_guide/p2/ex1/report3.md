# agent_guide p2 ex1 — step 3: the fixture answers a delegation

pyagag `4b52e93` → `517c44c` → `67325ac` → `174b74e`; agfront `aa77689`;
pinned in agautolab `090446d`, archsage `6a5c6c4`, pj-agdev `cbd93d5`.

## How it works

**The overlay.** A probe with a responder script is served on the trial's
own copy of the board, `<out>/overlay/mirror.sqlite`, marked with the
script (`agag.fixture.responder.make_overlay`, a SQLite backup of the fixture
store). The store the trial started from is never opened for writing: a test
compares its hash before and after a send. The plan asked for the send to be
recorded "in the trial's own overlay (not the fixture store)". A full copy
was chosen over a layer on top of the store because every read the store
answers (topics, histories, notes, root notes) then covers the new posts with
no change to `MirrorReads`.

**What an overlay takes.** On an overlay, and only there:

- `send_to_channel` records the post as the board's reader would have made
  it: the root note `agentchat send` writes first, then the message.
- `ensure_subscribed` is a no-op.
- `users()` answers from the board (`fixture_users` meta, so `send --to
  autolab-agstudio1` resolves). This also works on a plain fixture, where it
  was refused before.

Every other write (reactions, resolves, hold/release/disposition records)
is still refused, so the board is the same between runs.

**The script.** A probe's `script` is a tuple of `Canned(agent, text,
topics=())` lines. A post the served agent sends *addresses* an agent when it
mentions it (`@**autolab-agstudio1**`), is posted in its channel, is under
one of its topic prefixes (`workplan-`, `workrun-` for autolab), or asks it
with `to=<id>`. Each such post takes that agent's next unused line.
Selfnotes never do. `{ask}` in the line is the id of the post it answers,
`{asker}` the asker's mention. The line is posted by that agent in the same
topic **after the serving is over** (`responder.deliver`), never during it.

**The callback.** `Trial.converse(serve, calls)` plays the conversation
the way Front's listener does:

1. Post the person's message in home (on the overlay).
2. Serve it.
3. Post the marked reply home, addressed to the person.
4. Deliver the scripted answers.
5. If one names the served agent, serve home again, with the topic that
   called as `extra_threads`. That is what
   `agfront.zulip_listener`'s mention route does: it serves home and places
   the calling topic beside the chatlog.

This repeats up to four servings. Judging covers the whole conversation:

- `must`/`must_not`: the last serving's reply;
- the tool rules: every serving's calls;
- `sends_must`: the posts the served agent sent;
- `servings_min`: the answer counts only if it came back on a serving of
  its own.

`outcome.json` keeps every serving's reply and calls, the posts sent, and the
summed cost of all the servings' records.

## The two probes

| probe | the person asks | the script | passes when |
|---|---|---|---|
| `delegate-answer` | how long autolab's finished mission m20390 took and whether it got stuck, "ask autolab" | autolab: 41 minutes, stalled 12 minutes on a paywalled source replaced by a preprint | the callback serving (≥ 2 servings) carries the 41 minutes and the paywall/preprint, and a `send` was made |
| `delegate-decision` | have autolab decide the grow lights' hours for m20402; "if asked to choose, take the cheaper one" | autolab: (1) a `response_request … ask=decision` to Front, 16 h or 12 h; (2) after the answer: set to 12 h, 06:00–18:00, `b41d0e7` | a sent post carries the 12-hour choice, the last of ≥ 3 servings reports what autolab set, and none reports 16 h |

## Tried free first, then with the model

**The stub.** The plan's hint was the `stub` profile. Its `fake` harness
needs a command configured in the host overlay, so the test does the same
thing more directly. agfront's `test_a_delegation_is_answered_and_served_again`
patches `run_harness` with a stub that runs the **real** `agentchat send` in
the run's own environment, then prints a marked reply.
`delegate-decision` came back with three servings. The second serving's
thread held autolab's decision question and the third's held its final
answer. The sends were recorded, and the only rule missed was the tool-call
rule (a stub leaves no session log). Two driver bugs were found and fixed
this way before any paid run (below).

**Real runs** (Front desk, new guides, main checkout, `--out
pj-agdev/.local/agp2ex1/s3-*`):

| run | probe | result | servings | turns | cost | what happened |
|---|---|---|---|---|---|---|
| 1 | first `delegate-answer`: the pump schedule of the **running** m20402 | fail | 1 | 10 | $0.32 | read the board (the mission is running, 9 of 14 sources read) and **asked nobody**: "作業自体は動いているので今追加で聞き直すことはせず". `board.md`'s as9 sentence ("a 'how is it going?' starts their job again") at work, over the Developer's explicit "autolab に確認して". The probe was changed, not the guide (below) |
| 2 | `delegate-answer` (m20390) | pass, **for the wrong reason** | 2 | 15 | $0.34 | the responder then answered at once: Front's first serving sent, read the answer in the same serving and reported it; the callback serving only repeated it. Fixed: answers wait until the serving ends |
| 3 | `delegate-answer` | fail | 1 | 9 | $0.17 | read the ✔ mission topic, answered from it ("約45分… 途中の詰まりの記録はありません"), did not post into the ✔ topic (fd-wr: ✔ is finished) and asked nobody else |
| 4 | `delegate-decision` | **pass** | 3 | 18 | $0.47 | 1: two refused `send`s (no `--to`; then a topic belonging to the routine run), then a new `workplan-growbox-light-hours` with `--to autolab-agstudio1`. 2: autolab's question arrived; Front answered it there with `send --intent report --re 20126 "12時間/日でお願いします…"` and told the Developer. 3: reported 12 h/day, 06:00–18:00, `b41d0e7` |

The callback path is now exercised end to end on the fixture: a serving
ends, the answer arrives, the conversation is served again and continues.
p1 could show that only with a live mission.

## Driver defects found on the way

1. **An immediate answer tests nothing** (run 2). The responder first
   posted its line inside `send`, so the asking run could read it before
   it ended. Now it is queued and delivered after the serving
   (`responder.deliver`), as a real answer arrives.
2. **An answer served twice** (the stub test). After the fix above, the
   loop still looked for callbacks above a mark taken before delivery. It
   served the last answer again, a fourth serving. Callbacks are now the
   scripted posts after the serving's own reply.

## Observations (no guide changed)

- Asked to "ask autolab", Front **did not ask** in two of three
  `delegate-answer`-shaped runs. It read the board and answered from it
  instead. For running work (run 1) it cited the reason that is the as9
  sentence in pyagag `board.md`. For finished work (run 3) it read the ✔
  topic and said the board holds no record of a stall, which is true, but
  it was not what the Developer asked. Both are honest replies. Neither
  delegates. The probe measures this now. Step 5 repeats it to get a rate
  instead of one run.
- The first `delegate-decision` send lacked `--to` and was refused by
  `agentchat` itself ("--intent response_request needs --to"). The run
  recovered from the error text.

Step 3 cost: $1.30 (four runs).

## Tests

- pyagag `tests/test_fixture_responder.py` (5): a send is recorded and
  answered on the overlay only, and the fixture's hash is unchanged; only a
  post addressing the agent takes a line, in order, once; other writes
  stay refused; `agentchat send --to <name>` on the overlay writes the root
  note, then the message, then the answer; scripted probes are judged over
  sends and servings. `test_read_many`'s fixture now answers `users()`.
- agfront `tests/test_trial.py` (3, the stub loop included).
- Driver tests of agautolab, agobserver and archsage pass on this pin.
  Full suites are run in step 5.

## Correction (step 5)

The addressing rule above counted `to=<id>` in a post's `ag-post` line.
A listener is served only by a mention or by a topic it owns, and a step-5
run passed on a question the real autolab would never have seen. Since
pyagag `3a16479` only a mention, the agent's channel or its topic prefix
takes a script line (report5).
