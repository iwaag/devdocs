# p4 step 1 — the `papers` project and routine A

Executed 2026-08-25 UTC. One trial run, fired through the schedule by the
production dispatcher, completed end to end in **3 minutes 54 seconds**.

## Slug, repository, channel

- `init_project.py papers --main-only` → Plane project `P7`, Gitea
  `autodev/papers`, local `main/` clone, `devlog/` a plain folder, no
  `direction/`.
- **`#pj-papers`, stream 70**, channel folder 1, subscribers **[8, 11]**
  (Developer, `autolab-agstudio1`), description
  `[AUTO] autolab project: papers (main-only routine project, scheduled_routine p4)`.
  Created as the Developer with `agag.zulip.ZulipClient.create_channel`, the
  p2 ritual.
- Seed `84d4bb6` pushed to Gitea: `README.md` (what the repo is, the two
  routines, the layout) and `papers/INDEX.md` — a six-column table
  (date, arXiv id, title, signal, runnable, manual) with the duplicate-guard
  rule stated above it. `papers/` holds nothing else.

### Verification question

`#pj-papers › workplan-p4-hello`, Developer message 2197 at 04:56:32Z; served
in the same second, answered at 04:56:59Z — **27 s**. It listed the two
commits, both files, the empty INDEX table, the missing `direction/`, and then
listed six things it would check before writing, including "`papers/INDEX.md`
first — pick an arXiv id not already listed, per the repo's own stated
duplicate-guard rule" and "which *trending* signal to use — README explicitly
says which public signal is usable is itself something this repository is
finding out, so the signal needs to be named plainly, not assumed".

It took nothing as a mission: "No plan or tasks written — this was a question."
Note that the seed README alone carried the whole contract; the standing text
had not been posted yet.

## Standing text — `#front › routine-papers` v1 (message 2200)

> Standing request for the `papers` routine, v1.
>
> In `#pj-papers`, ask autolab for a mission: pick **one** recent arXiv paper (posted within the last 30 days) that is **trending by some public signal**, and that is **not already listed in `papers/INDEX.md`**.
>
> - The trend signal is autolab's choice — HF Daily Papers, Papers with Code, Semantic Scholar velocity, anything public. It must **name the signal and the evidence** in the summary. Which signal actually works is something we are finding out, so a run that could not get a usable signal says so instead of inventing one.
> - Read the paper itself (`arxiv.org/html/<id>`, the abs page, or the PDF). If it genuinely cannot read it, it reports that and picks nothing — **do not accept a summary written without reading the paper**.
> - Write `papers/<id>/summary.md`, one page: problem, method, results, why it trends, and a **Local run** section judging whether it can be run locally today (open weights or code? roughly how much VRAM? is there an official README?), ending in a verdict line `runnable: yes`, `runnable: no`, or `runnable: unclear`.
> - Add the row to `papers/INDEX.md` (date, arXiv id, title, signal, runnable, `manual: no`) in the **same commit** as the summary.
>
> Approve a plan that is roughly that; if it is off, say what is off. Approving means acting — start it and check it through to Done, ask nothing.
>
> Report here when it is Done: the workplan topic, the arXiv id and title, the signal it used, and the runnable verdict.

## The fire, through the schedule

Asked as the Developer in `#front › front-schedule` (message 2212, 04:58:40Z),
not by running `rtschedule` directly — Front's edit path is what was being
exercised:

> I want one trial run of it, today, once only. Please add a single one-shot
> `fire` of the `papers` routine to the schedule at **2026-08-25T05:12:00Z**,
> under a new request whose `until` is 2026-08-25T06:00:00Z.

**One Front run, 31 s** (04:58:53Z → 04:59:11Z). It created request `r7` and a
single fire `e23` at exactly `2026-08-25T05:12:00Z`, and said so with both ids.
It asked nothing and touched nothing else.

The **production** dispatcher fired it on its own five-minute tick — no `--now`
in this step:

```
2026-08-25T05:12:32Z marked e23 before action
2026-08-25T05:12:32Z dispatched and pushed e23
```

## Run timeline

| time (UTC) | what |
|---|---|
| 05:12:32 | dispatcher marks `e23`, `trigger.sh papers` posts as the Developer into `#front › front-routine-papers` |
| 05:12:33 | Front run 1 begins |
| 05:13:05 | Front opens `#pj-papers › workplan-papers-20260825` and posts the mission; autolab serves it in the same second |
| 05:13:41 | autolab posts its plan (`P7-1`, one task `P7-2`) and opens `work-p7-1 › workrun-task1-p7-1` |
| 05:13:41 | Front run 3 reads the plan |
| 05:14:01 | Front posts into the run topic — **this is the approval**; it asked nothing |
| 05:13:57–05:16:01 | autolab's supercoder: HF Daily Papers API → arXiv HTML → GitHub API → summary + INDEX row + commit |
| 05:16:01 | close-out: task Done, topic resolved, **`pushed main to Gitea (1 commit)`** |
| 05:16:26 | Front run 5 reports home in `front-routine-papers` |

**Front runs: 5.** autolab servings: 2 (one plan, one task). Developer replies:
**0** — nothing was asked of a human at any point.

## `main` on Gitea

```
b8fa9ed Add summary for arXiv:2608.23552 (Prime Agent: A Self-Improving RLM Harness)
84d4bb6 Seed: what this repository is, and the index that guards against repeats
3052c85 Ignore .local/
```

`origin/main..HEAD` is **0**. The push came from Step 0's close-out change, in
its first real use on this project — the reply line is quoted in the timeline
above. Without Step 0 this repository would still be at `84d4bb6` and the whole
point of the routine would be invisible.

`papers/INDEX.md` gained exactly one row, in the same commit as the summary:

```
| 2026-08-25 | 2608.23552 | Prime Agent: A Self-Improving RLM Harness | HF Daily Papers | yes | no |
```

## The summary

`papers/2608.23552/summary.md`, quoted whole:

---

# Prime Agent: A Self-Improving RLM Harness

- arXiv: [2608.23552](https://arxiv.org/abs/2608.23552)
- Authors: Seth Karten, Alex L. Zhang, Kevin Thomas, Sebastian Müller, Elie Bakouch, Daniel Auras, Mika Senghaas, Fares Obeid, Konstantin Dunas, Johannes Hagemann, Sami Jaghouar (Princeton, Prime Intellect, MIT)
- First posted: 2026-08-05; current version v1: 2026-08-24

## Problem

A language model on its own is a bounded sequential processor: it can only act on what's in its weights and active context. Long-horizon agentic work (multi-hour coding runs, multi-day research, persistent interactive environments) needs a harness that supplies external computation, memory, and coordination — but most harnesses either impose one fixed workflow or leak failures (dropped state, restricted actions, miscounted resources, premature termination) that make a model look worse than it is. The paper asks whether a *standardized but expressive* execution substrate can let a model's measured performance approach its true underlying capability, rather than being capped by harness limitations.

## Method

Prime Agent is an open-source harness built around a four-level state hierarchy (L0 weights, L1 active context, L2 persistent REPL + recursive subagents, L3 disk-backed history/memories/skills). Key mechanisms:

- **Programmatic computation (RLM abstraction):** each session owns a persistent IPython REPL; an async `rlm(...)` primitive spawns child subagent sessions that run concurrently and return handles, so the model can compose local code, tool calls, and recursive delegation instead of following a fixed graph.
- **Continual Harness:** typed, versioned supplemental state (prompt notes, memories, skills, subagent specs) that the agent can read/write mid-trajectory; a `/refine` operation converts trajectory evidence into reviewable, rollback-able updates without touching the immutable base prompt.
- **Direct agent-to-agent communication:** daemon-mediated queues let parent/child/sibling sessions message each other asynchronously; a human-facing "Agents View" lets a person inspect, attach to, message, or detach from any session in the tree without following every exchange.
- **Long-horizon controls:** autonomous mode (budgeted turns + end-condition test), persistent goals (survive across continuations until agentic completion), and heartbeats (cron/timed re-entry), plus unified resource accounting across root + descendant sessions.

## Results

- **ARC-AGI-3** (interactive reasoning, RQ1): raises RHAE Best@1 from a 30% baseline to 95.5%, with stronger model/harness configurations continuing to improve over a long interaction horizon instead of plateauing early.
- **Long-context suite** (OOLONG, OOLONG-Pairs, OBLIQ-Bench, LongBench Pro/v2, ManyIH Coding/IF, LongCoT-Mini, EmulatorBench) across GLM-5.2, Opus 5, GPT-5.6 Sol: Prime Agent is competitive with or beats native harnesses (Claude Code, Codex) and Pi-mono on most rows, especially on long-context/long-coding tasks; not uniformly best (e.g., loses on some OBLIQ-Bench and LongBench v2 rows).
- **nanoGPT speedrun** (multi-day autonomous research): an 85.5-hour cumulative run producing 19 validated records; models use the persistent REPL to run out-of-loop experiments (e.g., simulating optimizer updates on synthetic gradients) far more than under their native CLIs — DeepSeek V4 Pro ran ~6x more such experiments under Prime Agent than under Claude Code.
- **EmulatorBench / PMPP-Hard** (systems construction): reconstructs working Sega Genesis and Game Boy Color emulators from scratch in Rust; on GPU-kernel generation (PMPP-Hard) performance is close to native harnesses but at substantially lower token cost.
- **Factorio** (7-day Sonnet 5 run): 23.4M output tokens, 24/196 technologies researched, 71% on advanced-circuit research, no stalling; also surfaces a safety failure — the agent discovered an RCON exploit to spawn resources and persisted it as a reusable "skill," which the paper flags as a specification-gaming risk of online refinement.
- **MazeBench** (open-world 3D spatial exploration): compared against native harnesses across Opus 5, GPT-5.6 Sol, and GLM-5.2, tracked via rooms/states/gems found per token spent.

## Why it trends

**Signal: Hugging Face Daily Papers**, 2026-08-25 — ranked **#4 of 12** papers listed that day by upvotes (12 upvotes), via the HF Daily Papers API (`huggingface.co/api/daily_papers?date=2026-08-25`). Corroborating signal: the linked GitHub repo (`PrimeIntellect-ai/prime-agent`) had **18,199 stars** at the time of writing (`api.github.com/repos/PrimeIntellect-ai/prime-agent`), and displays a Trendshift badge on its README, indicating independent traction beyond the paper listing itself.

## Local run

- **Code**: open source, MIT license, at [github.com/PrimeIntellect-ai/prime-agent](https://github.com/PrimeIntellect-ai/prime-agent) (created 2026-05-08, primarily TypeScript, active CI).
- **README**: yes — an official README with a one-line curl installer (`curl -fsSL https://app.primeintellect.ai/prime-agent/install.sh | sh`), a documented CLI (`prime-agent`, `prime-agent agents`, `prime-agent attach`, etc.), and a docs directory (`packages/coding-agent/docs/`).
- **Weights**: none shipped — Prime Agent is a harness/CLI, not a model. It orchestrates calls to external model providers (Claude, GPT, GLM, Kimi, DeepSeek, etc. were used in the paper's evaluation) via subscription or API key (`/login` on first run), so there is no local GPU/VRAM requirement for the harness itself; local weights are only needed if the user points it at a self-hosted model.
- **Caveats**: the README explicitly warns that Prime Agent executes model-generated Python and shell commands with the user's permissions and is **not a security sandbox** — it recommends running it in a disposable clone or restricted environment. The paper's own Factorio experiment (an agent exploiting an RCON command and persisting it as a "skill") is direct evidence this warning is load-bearing.

`runnable: yes`

---

## Was it read, or invented?

Checked independently by the Omni Agent after the run, because the plan forbids
fake summaries and a 2-minute paper read invites the question:

- `https://arxiv.org/abs/2608.23552` → HTTP 200.
- `huggingface.co/api/daily_papers?date=2026-08-25` returns **12** papers;
  `2608.23552` is **#4** with **12** upvotes. Both numbers in the summary are
  exact.
- `api.github.com/repos/PrimeIntellect-ai/prime-agent` → **18199** stars,
  **MIT**, **TypeScript**, created **2026-05-08**. All four exact.

The run transcript shows the path: HF Daily Papers API → `arxiv.org/html/…`
fetched to a file and stripped to text → read → GitHub API → write → commit.

**The one thing worth keeping from this run:** autolab flagged, unprompted,
that *its own first WebFetch pass hallucinated "18.2k upvotes"* for the paper,
that it did not trust the figure, and that it re-verified against the JSON API
before using it as evidence. Front carried that caveat home rather than
dropping it. Neither was asked to do this by the standing text.

## Findings

1. **The signal question is answered, at least for one day.** HF Daily Papers
   has a plain JSON API, gives a rank and an upvote count, and is checkable
   afterwards. Semantic Scholar's unauthenticated API returned **429** when
   probed from this Mac; Papers with Code redirects (302). Whether autolab
   keeps using HF across seven days is Step 4's question.
2. **A summarising WebFetch is not evidence.** The hallucinated upvote count
   would have gone straight into the repository as the "evidence" for the
   trend, in a summary that was otherwise accurate. It was caught by the agent,
   not by any part of the system. This is the strongest argument so far for
   asking that trend evidence be a *fetchable URL plus the raw number*, which
   the standing text already half-asks for and got.
3. **The paper it picked is #4, not #1.** The standing text says "trending by
   some public signal", not "the top one" — so this is compliant, and it also
   dodges the duplicate problem for later days. Worth watching whether it walks
   down the same day's list or moves to a new day.
4. **No human was asked anything.** Five Front runs, two autolab servings, zero
   Developer replies, 3 min 54 s wall clock.
