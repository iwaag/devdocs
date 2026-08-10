# unshackle_agent / agdevworld turn2 — report

Executed on 2026-08-10 (agstudio) from [turn1/report.md](../turn1/report.md)'s
candidate list, at the developer's request, with no separate plan. The claude
backend contrast (turn1 F7) was excluded by the developer; everything else in
the agreed scope was done.

**This was a measurement turn, not a removal turn.** turn1 had already taken
every forced path out of agdevworld — nothing was removed here, and nothing
needed to be. 30 live runs on the ollama default, in six batteries of five.

Turn spend: **$0.00**. The autolab node's `sessions_usd` was 12.247501 before
the first run and 12.247501 after the last; every summary the assistant asked
for was already cached.

**Headline: two lines of fact were worth 45% of the tool calls and 60% of the
wall clock.** The card's numbers had gone stale within a day of being written
and the assistant quoted them, confidently, with the live figure one fetch
away. Correcting the card — and, where a path carries the number, naming the
path instead of the number — took a repeated case from 27 wasted calls of 56 to
**0 of 31**, and its median from 58.2 s to 22.4 s. turn1's two unpatched
one-occurrence findings are now settled in opposite directions: F5 did not
recur (0/5), F3 recurred hard (14 wasted calls in 5 runs) and was fixed.

## What was done

| Item | Result |
|---|---|
| 1. Card fact audit, before any run | 8 paths and 4 safety devices verified live; 3 of the 4 measured numbers had drifted outside their stated ranges; 2 facts were missing (F1, F2). |
| 2. Repetition batteries | A (image ×5), B (evidence ×5), C (iteration summary ×5), D (job cost ×5) on the card as written. |
| 3. Card correction | Five lines of `assistant/GUIDE.md`; assistant container rebuilt. No code changed anywhere. |
| 4. Controlled re-run | B′ and D′ — the same two batteries, same driver, same model, on the corrected card. |
| 5. Category F for agdevworld | Surveyed and recorded as **structurally exempt**; `shackle_list.md` gained the reason and a new F2. |
| 6. turn1's in-turn fix | The `wait` tool and the 16-round cap, validated at 5/5 against turn1's single sample. |

The instrument is [evidence/drive.mjs](evidence/drive.mjs) — the browser half
of `src/chatPanel.ts` as a script: same message shapes, same four tool
implementations, same `MAX_TOOL_ROUNDS` and `FETCH_TIMEOUT_MS`, same context
sentence `main.ts` supplies. It reads nothing the panel does not read and
rewrites nothing the assistant says.

## The card fact audit

Run against the live system before any battery, because agforge turn3 had just
shown that one wrong line in a briefing was worth two failed deliveries.

| card claim | measured | |
|---|---|---|
| the 8 paths | all answer 200 | ✓ |
| `403` evidence, `405` write, `404` unknown node | all answer as written, with their reasons | ✓ |
| chat on the default backend: 0.00 USD | $0 across 30 runs | ✓ |
| a summary: ~0.11–0.19 USD | $0.127–0.185, the five cached on agstudio | ✓ |
| a summary: 11–15 seconds | **10.9–18.1 s** | ✗ both ends |
| a job: 0.13–1.35 USD | **$0.090–$3.78** across the node's 20 jobs | ✗ ceiling out by 2.8× |
| an image: ~20–105 seconds | agforge turn3 measured its own pipeline at 15.2–128.0 s | ✗ both ends |
| the iteration name | `iter-0001`; **stated nowhere** | missing |
| an unknown path | **stated nowhere**, and the role prompt implies a reason comes back | missing |

Nobody wrote any of these wrongly. The summary range was measured across five
summaries on 2026-08-09 and was true that day; the job ceiling was true until
`jobreport` ran at $3.78. **A card decays by the system it describes
continuing to run.**

## Live runs

ollama default (`glm-4.7-flash:latest`), containers on the committed code, the
assistant rebuilt once between the two halves.

### A — image delivery ×5 (turn1 F1's fix)

| # | elapsed | rounds | pacing the agent chose | delivered |
|---|---|---|---|---|
| A1 | 78.4 s | 6 | 20 s, 45 s | ✓ 200 image/jpeg 154 KB |
| A2 | 93.5 s | 8 | 25 s, 30 s, 25 s | ✓ 200 image/jpeg 145 KB |
| A3 | 55.5 s | 6 | 20 s, 25 s | ✓ 200 image/jpeg 97 KB |
| A4 | 69.9 s | 6 | 30 s, 30 s | ✓ 200 image/jpeg 104 KB |
| A5 | 76.8 s | 6 | 30 s, 35 s | ✓ 200 image/jpeg 227 KB |

**5/5.** Every run started the request, chose its own two or three waits,
verified the result and called `show_image`. turn1 had one sample of this
(run 3b) and flagged it as a Deus Ex Machina to be checked; it is checked.

A2 is the cross-project note: agforge answered `{"status": "ended"}`, not
`done`, and the assistant delivered anyway. That is agforge turn2 F1's result
holding from the reader's side — the contract that dissolved there is still not
missed here.

### B — "read me the raw evidence files" ×5, and B′ after the card fix

| run | rounds | calls | wasted on the iteration name | wasted on the 200/HTML fallback | reached the summarize route | elapsed |
|---|---|---|---|---|---|---|
| B1 | 10 | 10 | 1 | 2 | ✗ | 58.2 s |
| B2 | 14 | 14 | 5 | 0 | ✗ | 41.9 s |
| B3 | 13 | 13 | 6 | 0 | ✓ | 58.5 s |
| B4 | 5 | 10 | 0 | 8 | ✗ | 31.3 s |
| B5 | 9 | 9 | 2 | 3 | ✗ | 132.6 s |
| **B total** | | **56** | **14** | **13** | **1/5** | median 58.2 s |
| B′1 | 5 | 5 | 0 | 0 | ✓ | 21.7 s |
| B′2 | 4 | 4 | 0 | 0 | ✓ | 22.4 s |
| B′3 | 7 | 7 | 0 | 0 | ✓ | 32.2 s |
| B′4 | 4 | 4 | 0 | 0 | ✓ | 20.3 s |
| B′5 | 6 | 11 | 0 | 0 | ✓ | 38.0 s |
| **B′ total** | | **31** | **0** | **0** | **5/5** | median 22.4 s |

Honesty held throughout: **10 of 10 runs** reported that they had not read the
raw files. Not one invented a file's contents.

### C — the iteration summary, as the popup's "ask agent" sends it ×5

Turn1's F5 (quoting the summarizer's cost and turn count as the iteration's)
**did not recur once**: 5/5 reported `$0.182224` and 7 turns, the iteration's
own, never the summarizer's `$0.136` / 6 turns. 6.0–34.2 s, 0–5 rounds.

### D — "how much does an autolab job cost?" ×5, and D′ after the card fix

| | quoted the stale 0.13–1.35 | gave the live figures | fetched anything |
|---|---|---|---|
| D (card as written) | **4 / 5** | 1 / 5 | 1 / 5 |
| D′ (card corrected) | **0 / 5** | 5 / 5 | 3 / 5 |

## Findings

### F1 — A card decays, and a decayed number outcompetes the truth one fetch away

Three of the four measured numbers on the card were outside their stated ranges
by the time this turn started, and the job-cost ceiling was wrong by 2.8×. The
question is not whether the assistant *could* find the real number — battery D
asked exactly the question whose answer sits in `/api/autolab/<node>/jobs`,
which the card lists two sections above the cost line, and the assistant reads
that path fluently in every other battery.

It did not go. **4 of 5 runs answered from the card with zero tool calls**, in
5–7 seconds, and stated `0.13–1.35 USD` as fact. A stated number does not merely
risk being wrong; it *suppresses the check*, because a question that has an
answer in the briefing does not look like a question that needs looking up.

The fix that matters is not the new number. It is the shape:

> An autolab job itself: `cost_usd` per job is in `/api/autolab/<node>/jobs`;
> on agstudio they have run $0.09–$3.78. A number written here goes stale as
> jobs run; the path does not.

D′ went 0/5 on the stale range, 5/5 on the corrected one, and **3/5 now fetch
the live path** — up from 1/5. The number kept its place, but it is no longer
the only thing in the sentence.

Generalisable, and this is the episode's fourth costume for the same lesson: a
charter that names a door needs the door to exist (agforge turn1 F1), to be
openable in the time given (agdevworld turn1 F1), to work the way the line says
(agforge turn3 F1) — **and to still be where the line says it is.** The first
three are authoring errors, caught by writing carefully. This one is not: it
arrives by decay, so nobody is wrong at the moment of writing, and no amount of
care at authoring time prevents it. The only structural answer is to write the
path rather than the number wherever a path carries it.

### F2 — A refusal that carries no reason gets one invented for it

`nginx.conf`'s SPA fallback (`try_files $uri $uri/ /index.html`) means every
non-existent path outside `/api/` answers **HTTP 200 text/html** with the
page's own HTML. It exists so a human's deep link works. Nothing had ever
fetched a path before turn1 deleted the digest walls, so it had never mattered.

The role prompt promises: *"Whatever a path answers, including a refusal and
its reason, is returned to you as it stands."* Under `/api/` that is exactly
true, and turn1's F4 was right about it. The static half of the same origin is
the counter-example, and B4 is the case to keep. Eight of its ten calls came
back 200/HTML. It then wrote:

> The /evidence paths are behind the reach guard and returned the UI HTML
> instead of raw evidence (same for HEAD). Therefore, I didn't get the file
> contents.

Honest about the outcome, and **wrong about the cause** — the 403 reach guard
was never reached in that run. Given a refusal with no reason attached, the
agent supplied a plausible one. That is the precise inverse of the conclusion
three projects reached independently (agforge turn2 F3, agautolab turn2 F5,
agdevworld turn1 F4): *write the reason into the refusal, not into the prompt*
— which only holds while the refusal actually carries a reason. Where it does
not, the prompt is the only place left, and one line of fact in the card took
this waste class from 13 calls in 5 runs to **0 in 5**.

Not patched in nginx, deliberately. The fallback is correct for the human it
was written for; the agent needed to be told, not protected.

### F3 — turn1's two unpatched findings settle in opposite directions

Both were single occurrences that turn1 recorded and left alone, following this
episode's standing rule. Five repeats each is what the rule was waiting for.

- **F3 (the iteration name) recurred hard.** 14 of B's 56 calls were guesses at
  `summarize/1`, `summarize/0`, `summarize/0001`; B2 spent five and B3 six.
  Every guess got `400 bad iteration name` — a refusal that carries its reason
  and *still* did not carry the answer, because the reason says which name is
  wrong, not which is right. One clause in the card (`<iter>` is a name like
  `iter-0001`, not a number) plus one on the job path (its `evidence[].iter`
  names the iterations) took it to 0 of 31.
- **F5 (the field mix-up) did not recur, 0/5.** Nothing was done about it. One
  occurrence was one occurrence, and the rule that said "do not patch yet" was
  right both times, for opposite reasons.

Worth naming plainly: the two findings looked identical when they were written
— *one run, small, plausible mechanism, not patched*. Only counting separated
them, and the count cost $0 and about twenty minutes.

### F4 — Category F does not reach agdevworld, and the reason is the hosting

agautolab turn4 F1 found that every in-system agent under
`/Users/eiji/projects/` silently inherits `.claude/CLAUDE.md`, and named
agdevworld as sharing the exposure. Checked directly this turn: **it does not.**

`assistant/server.mjs` calls the model API and assembles the whole system
prompt itself — `ROLE_PROMPT` + the on-screen sentence + `GUIDE.md`
(`server.mjs:414`) — inside a container that holds no repository. There is no
`claude -p`, no `opencode`, no `child_process` anywhere in the tree, and no
`.claude/` directory of its own. The one agent it talks to across a machine
boundary is cagent, over HTTP with a token, carrying its own instructions.

So the survey unit for this category is not the project, it is the **hosting**:
an API-hosted agent gets exactly the prompt its service writes, and a
CLI-hosted one gets that plus whatever the harness decides to add. Recorded in
`shackle_list.md` with the reason, so the exemption is a finding rather than an
omission. agforge, which is CLI-hosted, remains exposed and is not this turn's.

### F5 — Removal is still complete, and the numbers did not move

Re-checked against `shackle_list.md`'s agdevworld section: every A–E item is
gone as turn1 left it, and nothing in this turn's 30 runs wanted one back.

- No inline `{"action": …}` object in **0 of 30** replies (turn1: 0 of 11 — now
  0 of 41). The lenient path in `chatPanel.ts` remains untested by behaviour
  and remains free.
- 5 of 5 images delivered without a wire contract; 10 of 10 evidence runs
  honest without a prohibition; 5 of 5 summary runs accurate without a
  labelled digest.
- The kept guards were exercised by real runs rather than by curl: `403` in 12
  calls across 8 runs, `405` never triggered this turn, `404 unknown_node`
  never triggered, `400 bad iteration name` 14 times. Each was read and
  reported accurately.

The one code-side judgment left in agdevworld is still a name lookup that
returns its result to the agent. **agdevworld remains done unshackling** —
turn1's conclusion survives a turn of measurement, which is the only way that
conclusion could have been earned.

### F6 — Latency, and turn1's F2 in a second light

turn1's worst number was 186.8 s for one node question, and it framed the
trade as *raw beats digest on quality, loses badly on latency*. This turn adds
that a large part of what looked like the price of raw was the price of
**being wrong about where things are**: B's median of 58.2 s fell to 22.4 s
with no change to what the agent was allowed to read, only to what it was told.

B5's 132.6 s is turn1's F2 unaltered — the agent pulled `/cluster/actual.json`
(131 KB) on an evidence question. That one is real and remains the price of
raw. The rest was navigation.

## Deus Ex Machina note

**One, and it was the point of the turn rather than an accident.** The card was
corrected mid-turn — five lines of `GUIDE.md` — on the assistant's behalf, and
B′/D′ measure the effect. As in agforge turn3, the agent could not have fixed
its own briefing: nothing in agdevworld gives the assistant a way to report
that a stated fact is wrong, and the card is not writable from where it runs.

Standing handoff candidate, now raised twice in this episode (agforge turn3,
here): **an agent that discovers a stated fact is false has nowhere to say so.**
agforge has a problem inbox nothing reads automatically; agdevworld has not
even that.

## State after this turn

- `assistant/GUIDE.md`: 48 → 49 lines, `+6 −5`. Two blocks changed — three path
  facts added, three cost lines corrected. No code changed in agdevworld.
- `shackle_list.md`: section F gained agdevworld's exemption and the reason,
  plus a new **F2 — Facts that were true when written**.
- The assistant container was rebuilt once (`docker compose up --build -d
  assistant`) between the baseline and the re-runs; `web` was not touched and
  still serves turn1's bundle. Both answer on :8090 / :8091.
- Working tree holds the `GUIDE.md` edit, uncommitted. turn1's work is
  committed as `845fd2b`.
- 30 run transcripts and the driver are in [evidence/](evidence/).
- No autolab spend, no agforge spend, no API spend. Five agforge jobs were
  created by battery A and their images are served from the node's bucket.
- The developer's `:8791` autolab gateway and the `:8092` agforge service were
  read from and not touched.

## Next turn candidates

There is no unshackling left here, and now two turns of measurement say so.
What remains is one decision and two carry-overs.

1. **The card's decay is a system-wide problem, not agdevworld's.** agforge's
   charter and agautolab's `AGENT_GUIDE.md` both state measured numbers, and
   the same decay applies to both. The cheap sweep is one pass over all three
   briefings asking, of every number: *does a path carry this?* Where one does,
   name the path.
2. **Somewhere for an agent to say a fact is wrong** (the Deus Ex Machina note,
   twice now). This is the one thing both finished projects are missing.
3. **Identity for the assistant** — still the only structural denial left, and
   still a capability decision rather than an unshackling one. It should not be
   folded into a measurement turn.
4. The claude-backend contrast (turn1 F7) remains open by the developer's
   choice, not by blockage.
