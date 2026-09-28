# agent_guide p2 — step 7: trials and roll-out

## Fixture probes: Front, old guide against new

Each probe was served by `agfront.trial` against the fixture board (step 6).
"old" is p1's composition: the guide tree of agfront `447bb03` (the live
checkout before this phase's merge; after the merge, extracted with `git
archive`) without pyagag's shared sections. "new" is this phase's. Both
used this phase's tools, so the difference is the guide.

| probe | guide | result | turns | cost | notes |
|---|---|---|---|---|---|
| `aisvgs-sufficient` (#15673's wording) | old | pass | 8 | $0.21 | found `pj-aisvgs`, both rounds, the round-3 plan; offered to have autolab run round 3 |
| | new | pass | 11 | $0.19 | the same findings; the same offer |
| `growbox-thing` | old | pass | 16 | $0.25 | lost turns guessing (`topics --prefix work`, `read pj-growbox work-m20402`) |
| | new | pass | 9 | $0.18 | straight path: `channels`, `agproject status`, `topics`, `read` |
| `forge-protoprey` | old | pass | 10 | $0.14 | |
| | new | pass | 10 | $0.16 | both deliveries with ids and keys; the footsteps as not delivered with forge's two options; no birthday card |
| `receipt-owed` | old | pass | 7 | $0.22 | also ran `recheck`, which an owed answer does not need |
| | new | pass | 6 | $0.18 | `receipt 20110` → MISSING → `receipt 20110 --repair --because 20111`; no hand-written line |
| `hold-release` (×3 each) | old | 3/3 | 14, 7, 7 | $0.67 | |
| | new | 3/3 | 8, 4, 11 | $0.62 | |
| `hold-release` with the continuation note (×3 each) | old | 3/3 | 6, 8, 6 | $0.48 | |
| | new | 3/3 | 9, 7, 8 | $0.58 | |

**The #15673 probe reproduces p1's content difference on demand.** Both
guides pass p1's rule (the study, its rounds, round 3 not started), and
neither carries the Developer's decision that round 3 is theirs to do by
hand, recorded only in `#archsage-agstudio1 › ✔ study-aisvgs-round3`. Both
offered to ask autolab for round 3. That is p1's open finding 1 (where a
decision about a study is recorded), held by the Developer; the probe
reports it as an *observed* fact, outside the pass rule. The first version
of the rule counted it and was fooled by 「手を動かした検証」 ("hands-on
verification") matching the spelling 「手を動かし」; the rule was fixed and
both replies re-judged (`python -m agag.fixture check`).

## Fixture probes: the other agents (new guides)

| probe | role | result | turns | cost | what it did |
|---|---|---|---|---|---|
| `archsage-loose-sage` | archsage (frontier) | pass | 6 | $1.15 | took "the sprout-box sage" as `sage:growbox`, read its tree, answered 2–5 days for mung beans with four sources and the tree's limits, left the queued brief open |
| `planner-other-project` | autolab planner | pass | 20 | $0.28 | read `pj-growbox` off the board (topics, plan, the two finished strands, the running mission, its task topic); wrote no files and asked before planning |
| `entrance-plans` | autolab entrance | pass | 13 | $0.35 | all four projects in one line each, with m20402 running and v0.2 waiting for the go-ahead. Its shell loop over channels was refused; it then found `read` of several topics in `read --help` and used it |
| `triage-unopened` | Observer triage (local model) | pass | 2 | — | `legit`: autolab asked the Developer (#20099) and the next move is theirs |

The triage probe first failed in the driver: `agag.trace` asks
`hasattr(client, "roster_owner")`, and a fixture's `MirrorReads` refused
it instead of answering. Fixed in pyagag (`roster_owner` from the store,
with a test).

Fixture total: about $5.7 across 24 runs.

## Live trials

All posts by the Omni Agent (user 9, the Developer's full proxy) with its
own credential; the posts went through this time without a classifier
refusal.

### A hold placed and released: `#front › front-desk-20260928-agp2-hold`

| post | who | what |
|---|---|---|
| #15835 | Omni | "start nothing yet; how pj-protoprey v0.2 proceeds is my decision — keep it on hold" |
| #15837, #15838 | Front, run-0182 (7 turns, $0.16) | `[selfnote][hold] decision a15835 by 9 (Omni Agent) #15835`; reply names the hold. The first `hold` call failed on argparse (below) and cost one turn |
| #15842 | Omni | "I release the hold; the trial ends here, stop following this request" |
| #15844 | Front, run-0183 (**1 turn, no tool call**, $0.07) | "Released hold #15837 … Recorded the disposition as `withdrawn` …" — **nothing was recorded**; the command syntax it quoted does not exist |
| #15847 | Omni | "No record exists: #15837 is not released and there is no disposition; record them" |
| #15849, #15850, #15851 | Front, run-0184 (10 turns, $0.18) | `[selfnote][hold-release] #15837 by 9 … #15842`, `[selfnote][disposition] withdrawn a15835 … #15842`; the reply says its earlier reply described commands it never ran |

The final state is right, and every record was written by Front. run-0183
is the failure p1 found with dispositions, "saying it is over records
nothing", in a new place: a reply that describes records it never made.
The paragraph that answers it is in `work.md` word for word as p1 left it.

It did **not** reproduce: the same conversation on the fixture, with and
without run-0182's continuation note in the prompt (checked in the dry-run
prompt: the continuation block and "hold recorded …" are there), recorded
both release and disposition in 12 of 12 runs, 6 with each guide. One in
thirteen servings of this shape failed, the live one. No guide text was
changed for it, because no change could be shown to help; the probe
`hold-release` stays in the fixture to measure it, and it is a finding in
report.md.

### The `hold` command refused its own help's example

run-0182's first call was `agentchat hold 15835 --for decision --evidence
15835 "why…"`, the form `hold --help` shows, and argparse refused it
("unrecognized arguments"): a positional is filled once, so words after
the options had nowhere to go. The same held for `release`, `disposition`
and `relation`. Fixed in pyagag `1752704`: for those four commands, words
after the options join `why` (`chat.parse`, with a test). The fixture
reruns after the fix wrote `release <id> --evidence <post> "why"` and
passed without the retry.

### Receipt repair when Observer asks about an owed answer

p1 did not re-run it: in failsafe p6 the listener's own startup recovery
wrote the receipt, so the guide path (Observer asks, Front uses `agentchat
receipt`) never ran. It has no natural live trigger. The fixture stages it
exactly: a desk conversation whose delegated answer (#20110) a later Front
post took up (#20111), with no receipt — what `exit-before-receipt` leaves —
and Observer's "undelivered" request with its owed note. Both guides
passed (above); the new one in 6 turns: `receipt --help`, `receipt 20110`
(MISSING), `receipt 20110 --repair --because 20111` (refused only by the
fixture), `trace`, and a reply saying so. The live half of this path, the
listener writing the receipt, is listener code this phase did not change.

## Paragraphs that moved, and what re-tested each

| paragraph (report1 key) | moved to | re-tested by |
|---|---|---|
| as9 "how is it going?" restarts their job | pyagag `board.md` | the Front and entrance probes read the board and posted nothing; the fixture refuses any post |
| fd-wr ✔ is finished, no second start | pyagag `board.md` | every probe read ✔ topics as finished (`forge-protoprey`, `growbox-thing`, `entrance-plans`) |
| as5-8 each serving ends; callback | pyagag `callback.md` | the live hold trial: three servings, each ended with its reply; no probe delegates (the fixture refuses a `send`), so the callback itself is covered by p1's live mission and unchanged listener code |
| as10 list afresh every time | pyagag `entrance.md` + autolab's vocabulary | `entrance-plans`: all four projects listed |
| fs5-1, fs5E refresh topic per run, addressed to archsage | archsage guide, same words | text unchanged; `archsage-loose-sage` served the rewritten guide |
| trend7, fs3p task files; an agreement in the plan closes nothing | autolab planner, same words | text unchanged; `planner-other-project` served the rewritten guide and wrote nothing without a decision |
| smoke, fs4r, sub26, fs2T1 (the worker) | unchanged in the worker guide | not moved; no re-run |
| rw3C, rw3r, fs1t (triage), ob2 (observe) | unchanged | `triage-unopened` |
| p1's receipt paragraph (fs6r) | `work.md` since p1 | `receipt-owed`, both guides |
| p1's hold paragraph (fs6h) | `work.md` since p1 | the live hold trial and `hold-release` ×12 |

## Roll-out

| time (Z) | what |
|---|---|
| ~13:10 | `agent-guide-p2` fast-forwarded into `main`: pyagag `fe1a071`, archsage `d176ec8`, agfront `d8bc699`, agautolab `51d7973`, agforge `db17875`, pj-agdev `c539f80` (agobserver), pj-clusterintent `0cf66c3` (cagent); no run was in flight (no child process under any listener) |
| ~13:14 | pyagag `fe1a071` locked and synced in agfront, agautolab, agforge, archsage, agobserver, cagent and the relay; locks and pointers committed (agfront `5c91d79`, agautolab `259e2b4`, agforge `c521423`, archsage `584d86f`, agdevworld `334c4cf`, pj-agdev, pj-clusterintent `0f31f8d`) |
| 13:17:24–13:17:5x | kickstarted, one at a time: agfront, agautolab listener and gateway, agforge listener and request service, agobserver, archsage, cagent-api, cagent-zulip, agentroom relay. Logs show a normal start (`recovery (startup)`, old mentions ignored); the relay's mirror `live` |
| ~14:0x | pyagag `1752704` (the `why` fix, fixture probes) merged, locked and synced everywhere; agfront's trial commit cherry-picked onto `main` (`9d32e19`); locks and pointers committed. No restart: the changes are in the `agentchat` CLI and the fixture, which a run loads per call |

Checked after the roll-out: `chat_environment` for agfront's spec sets
`AGENTCHAT_MIRROR` to `.local/mirror/mirror.sqlite`, which exists and is
built, so live runs' look-only commands read the listener's mirror.
`nctl status` before the roll-out: Nautobot, worker and dumps healthy,
submodules clean, drift ok. comfynotify keeps its old pin (it uses none of
this); agautolab1 (the VM) was not redeployed (out of scope).

## Tests after each pin

Read from the summary line, main checkouts:

| suite | `fe1a071` | `1752704` |
|---|---|---|
| pyagag | 1154 passed | 1157 passed |
| agfront | 195 | 195 |
| agautolab | 331 | 331 |
| agforge | 265 | 265 |
| agobserver | 177 | 177 |
| archsage | 40 | 40 |
| cagent | 204 | 204 |
| agentroom relay | 363 | 363 |

The worktree-only failures of steps 2–4 (sibling paths, `.local/zulip.env`)
pass in the main checkouts.

## Cost

Fixture probes about $5.7 (24 runs, trial records under
`pj-agdev/.local/agp2/agfront/.local/agent/`, not the live ones); live
desk runs 0182–0184 $0.42.
