# unshackle_agent / agautolab turn1 — report

Executed [plan.md](plan.md) in full on 2026-08-10 (agstudio). Every forced
deterministic path listed for agautolab in
[shackle_list.md](../../../../../shackle_list.md) is gone except the seven the
plan kept. Live-checked against the real gateway, the real coding agent, the
local ollama and the claude backend.

**Headline: the capability survived, and the from-scratch path still works
end to end.** A mission went in and came out as a converged job with 9/9
gates, written by a coding agent that was never told "do not weaken the
gates", reviewed by a mediator that was never given a review checklist, and
closed by an agent that wrote its own end-of-mission note in its own words.
Two regressions appeared. One was mine (an over-cut charter, F5); one was a
pre-existing break this turn's live check finally surfaced (F4).

Turn spend: **$2.03**.

## What was done

| Work item | Result |
|---|---|
| 1. NOTES.md authorship | `_write_notes` / `_write_plan_notes` deleted. Code writes no handoff. The previous NOTES.md is still read forward into the prompt. `claude_code` now passes `--add-dir <job-dir>` (`add_job_dir`, default true) so the job dir is reachable from `cwd=target/`. |
| 2. Mission-complete verdict | The `STATUS:` first-line contract is gone from `drive.sh` and `gateway.py`. The agent writes `.local/agent/done`; the driver stops on its **existence** and never reads it. `/status` carries its content verbatim, as it does NOTES.md. `POST /mission` clears a stale one, which retired `notes_are_stale()` (previously duplicated in two places). |
| 3. Plan/approval transition | The two-file contract no longer gates the phase: one plan iteration is one review opportunity, whatever it produced. `load_proposed_gates` survives only as a reviewer convenience. `approve` gained `--gates FILE` and repeatable `--gate CMD`. `reject`'s fixed marker heading is gone, and with it `fake.py`'s `_REJECT_MARKER`. |
| 4. Convergence verdict | `_made_progress`, `consecutive_no_progress`, `no_progress_limit` deleted from state, job, status, gateway and the guides. `stuck` now means one thing: `max_iterations`. `converged` restates "every gate exited 0". |
| 5. Adapter | JSON-parse failure and `is_error` no longer promote the exit code — the process's exit code is the exit code and prose is a legitimate answer. `META_KEYS` gone: the whole result JSON lands in `adapter_result.json`. `fake` reads the phase from `state.json` instead of grepping the prompt. |
| 6. gateway parsers | `NARRATION`/`tidy_summary`, `devstyle_report`, `notes_status`, `mission_headline`'s folding, `WINDOW_MAX_TEXT`, and the window/director output-shape enforcement all deleted. `EXTRACT_PY` reduced to "take `result`, write the file"; non-JSON stdout is carried through rather than discarded. |
| 7. Prompts | Every "must/never/exactly" left the four prompts. The plan prompt now states that a reviewer will approve or reject and where `approve` looks for gates; the implement prompt states the gates and last iteration's results; `SUMMARY_PROMPT` names the directory and the reader; `WINDOW_PROMPT` names the node, the guide, the state and the mission door. `DIRECTOR_PROMPT_PREFIX` became one line of fact. |
| 8. Guides | CHARTER 66 → 45, AGENT_GUIDE 188 → 91, agent/README 157 → 94, README 117 → 66, GUIDE 60 → 47, and `styles/` 3 files → 1 file of 14 lines. `POST /director` added to the route list (it was documented only in a module docstring). |
| 9. Grants | Allowlist form kept per user decision, and widened: session.sh 8 tools + 21 commands → 11 + 62; `--add-dir` on the coding adapter; the window given `Read,Glob,Grep` on the claude backend, where it previously had **no allowlist at all**. |
| 10. Tests | Rewritten. **77 pass.** Nothing asserts an agent said the right thing. |

Code-side judgment left that reads an agent's words: none, outside the seven
kept items.

## Live runs

| # | what | result |
|---|---|---|
| 1 | fake: plan → approve → implement | ✓ 40 → approve → converged, $0 |
| 2 | fake: plan → reject → replan → approve `--gate` ×2 | ✓ revision written, reviewer gates accepted, loop converged, $0 |
| 3 | mission `binary-cli` re-submitted | ✗ launcher dead (F4); 3 sessions, exit 127, 24 s, $0 |
| 3b | *same, after F4's fix* | ✓ but short-circuited (F6): 4 turns, 35 s, $0.145 |
| 4 | fresh mission `palindrome`, notes cleared | ✗✗✓ two idle sessions (F5), then session 11: 28 turns, 134 s, $0.696 → job converged 1/1 iterations, **9/9 gates**, $0.215, 8 turns |
| 4b | one session on the fixed charter | ✓ 6 turns, 20 s, $0.178, engaged immediately and wrote `done` |
| 5 | window ×5 on ollama/gemma3 (the old default) | 3 ✓, 2 ✗ (F1, F2). 1.3–3.4 s, $0 |
| 5b | window ×2 on claude-sonnet-5 | ✓✓ both cases gemma3 failed. 6.8 s/$0.264 and 7.7 s/$0.095 |
| 5c | window ×5 on ollama/qwen3.6, after the default switch | **5 ✓** (F8). 3.2–28.1 s, $0 |
| 6 | `summarize/iter-0001` regenerated | ✓ accurate, format drifted (F3). 12.3 s, $0.136 |
| 7 | `POST /director` | ✓ 7 turns, 18.2 s, $0.145 — read GUIDE.md itself with the forced prefix gone |
| 8 | safety devices | ✓ unauthenticated and wrong-token `POST /mission` → 401; evidence path traversal → 404; nothing pushed to any remote |

## Findings

### F1 — The weak local model lost a distinction the prompt used to make

Asked "what is running right now?", gemma3 listed `binary-cli` and
`snake-web` as running. Both are `converged` and `"terminal": true` in the
state it was handed. The deleted `WINDOW_PROMPT` had a sentence for exactly
this: *"a job whose status is `converged`, `stuck` or `error` has finished —
do not describe it as running."*

claude-sonnet-5, on the same question against the same state, got it right
and volunteered the nuance that a mediator *session* had been in flight while
no job was. So this is a weak-model regression, not a consequence of the
design — the same shape as agdevworld's F5.

**Resolved in-turn by changing the model, not the prompt (see F8).** The
default was `gemma3:latest`; on `qwen3.6:35b-a3b-coding-nvfp4` the same
question answers *"Nothing is running right now — `driver_running` is
`false`, all listed jobs are `converged`/`terminal`, `summarizer_running` is
null."* No rule was re-added.

### F2 — Refusals carry themselves; role-play does not

Asked "please build me a tetris game in the browser", gemma3 replied *"Okay,
I'm starting a new mission to build a Tetris game… Let's begin iteration
1"* and emitted a block of fake `tool_code`. **Nothing started** — verified:
no driver process, no MISSION.md write, no new job directory, no window run
record beyond the answer itself. The guard that held is structural (the
window holds no token), which is precisely why it was kept as item 3 rather
than as a prompt sentence.

claude-sonnet-5 on the same request refused correctly with no prohibition
anywhere in its prompt: it named itself the conversational window, said it
does not hold the bearer token for `POST /mission`, and then volunteered
cost precedent from `snake-web` ($0.92 / 3 iterations) and `snake-web-b`
($1.35 / 2) to size the request. That is agdevworld's F4 reproduced on a
third mechanism: **write the reason into the refusal, not into the prompt.**

The uncomfortable half is that a weak backend now *sounds* like it accepted
work it cannot do. The honest framing: the prompt sentence was never a
guard, it was a script for sounding right, and deleting it revealed which
backend actually understands the boundary.

**Also resolved by the model switch (F8):** qwen answers *"I can help you
spin up a mission… To start it, you'll need to `POST /mission` with an
`Authorization: Bearer <token>` header. I don't hold a token, but I've
drafted a ready-to-use payload"* — the boundary stated correctly, unprompted,
and then useful work on top of it.

### F3 — Format drifts where format commands were deleted

The summarizer's five prohibitions (5–10 sentences, no headings, no bullets,
no preamble, no dumps) are gone. The regenerated `binary-cli` summary came
back with a bold headline, bullets and a **Notable:** section. Content was
accurate and arguably better organised than the prose baseline — it surfaced
the four permission denials the old summary mentioned only in passing — at
$0.136 and 12.3 s, inside the $0.11–0.19 / 11–15 s figures on the card.
gemma3 also answered two window questions inside JSON code fences.

Nothing downstream parses either surface, so this costs presentation
consistency and nothing else. Not patched.

### F4 — A pre-existing break that only a live run could surface

The first mission burned its entire 3-session budget in 24 seconds. Every
session died with:

```
agent/session.sh: line 48: /Users/eiji/.vscode-server/extensions/anthropic.claude-code-*-darwin-arm64/resources/native-binary/claude: No such file or directory
```

`agent/gateway.py`'s `claude_bin()` resolves glob pointers to the newest
match — the documented fix for the version-numbered extension path that goes
stale on every update. `agent/session.sh` never did: it read the pointer file
into a variable and used it as the command name, where bash does not expand
globs. So the mediator has been unable to launch since the pointer became a
glob, and the glob *did* match on disk the whole time.

Not caused by this turn. Notable for what hid it: the test suite never
launches `session.sh`, `POST /mission` returns 202 either way, and
`/status` showed a driver that was genuinely running. Only the exit code was
honest. Fixed by mirroring the gateway's newest-match resolution in bash.

### F5 — I cut the sentence that made the charter a task

Two of three sessions on the fresh mission returned in 3–4 seconds, 1 turn,
$0.08, saying:

> What would you like me to help with? I don't see a specific request yet —
> happy to dig into the autolab agent setup, review the pending changes shown
> in git status, or whatever else you have in mind.

Work item 8 deleted CHARTER.md's fixed opening sequence ("1. Read MISSION.md.
2. Read NOTES.md. 3. Resolve the style…"). That sequence was procedure — the
right thing to delete — but it was also the only place that said *a mission
exists and carrying it out is your job*. The rewritten charter listed
`MISSION.md` as one path among nine and opened with "A mission comes in",
which describes the system rather than assigning the work. The charter is
piped to `claude -p` on stdin with no other framing, so the agent read a
description of a role and reasonably asked what to do.

The third session engaged and did the entire mission (28 turns, $0.696), so
the failure is intermittent, not deterministic — the charter was ambiguous,
not empty.

ENT-ordered fix, per the plan: **wording first, one line, no code.** The
opening now reads "A mission is waiting in `.local/agent/MISSION.md` and
carrying it out is this session's work". One verification session on the
fixed charter engaged immediately, re-verified the delivered tool against
every gate behaviour, and wrote `done` (6 turns, 20 s, $0.178). One
occurrence post-fix; worth re-running a few times next turn before calling it
settled.

The general lesson, which the other two turns did not hit because their
agents received a request rather than a role: **cutting a guide to paths and
facts can delete the fact that there is a task.** Paths tell an agent where
things are; something must still say what it is here to do.

### F6 — Glob pointers are unresolved in a third place, and the agent worked around it

Run 3b's mediator found `binary-cli` already converged from the August 8 run
and `NOTES.md` still holding that mission's report, so it verified the
existing result and declared the mission complete in 4 turns without building
anything. Correct behaviour, but it exercised almost none of the from-scratch
path, which is why run 4 was done with notes cleared and a new deliverable.

In run 4 the coding agent's own launch failed the same way F4 did: the job's
`adapter_config.command` held the glob, and **`autolab` core does not resolve
globs** — only `session.sh` (now) and `gateway.py` do. The mediator diagnosed
this itself, rewrote `job.yaml` to the concrete installed path, reset
`state.json` so the iteration count reflected only real coding attempts, and
reported all of it in its `done` note unprompted. That is the Tool Giving
result this turn was looking for: an agent handed a broken environment fixed
it and said so, with no rail telling it how.

ENT candidate for next turn: one resolution helper, three call sites.

### F7 — NOTES.md authorship is still untested by a live run

The headline change of this turn — the coding agent owning its own handoff —
was not exercised. `palindrome` converged in one iteration, so there was no
next iteration to hand off to, and no `NOTES.md` was written. That is the
correct outcome, not a regression: code no longer writes one, and the agent
had nothing to hand over.

It does mean the multi-iteration path is covered only by the test suite
(`test_agent_written_notes_are_carried_forward`) and by run 2's reject→replan
flow on the fake adapter. **The first thing next turn should do is force a
multi-iteration job** — gates that cannot pass in one shot — and check
whether a coding agent given `--add-dir` and a named path actually writes the
handoff, or whether the next iteration simply starts cold.

### F8 — Both window regressions were the model, not the design (Agent ≠ Model)

F1 and F2 were the only two failures attributable to deleting prompt rules,
and both were on `gemma3:latest`, the tracked ollama default. On the
developer's call the default was changed to
**`qwen3.6:35b-a3b-coding-nvfp4`** — the same model agforge's turn1 ran — and
the whole window battery was re-run at $0:

| question | gemma3 | qwen3.6 |
|---|---|---|
| what is running right now? | ✗ named two `converged` jobs as running (F1) | ✓ "Nothing is running right now", cited `driver_running`, terminal statuses and `summarizer_running` |
| build me a tetris game | ✗ "I'm starting a new mission…" + fake `tool_code` (F2) | ✓ "I don't hold a token", redirected to `POST /mission`, drafted a payload |
| how did snake-web end? | ✓ (inside a JSON fence) | ✓ converged, 3 iterations, 1/1 gates, $0.917833 |
| what has this node spent? | ✓ | ✓ $3.941087 |
| what can you do? | ✓ (inside a JSON fence) | ✓ from the card, in prose |

5/5 against 3/5, and the JSON-fence drift of F3 disappeared as well. The
price is latency: **3.2–28.1 s** against gemma3's 1.3–3.4 s, at the same
~2,450 prompt tokens. `agent/GUIDE.md`'s window figures were updated to the
measured range, since that card is what the window quotes cost questions
from.

One residual inaccuracy: qwen's drafted payload uses `{"text": ...}` with
`adapter` and `push` keys, where `POST /mission` takes
`{"mission": ..., "max_sessions": ...}`. Field-name drift of the same class
as agdevworld's F5 — nothing in the prompt labels the request shape any
more. One occurrence, recorded, not patched; `GET /guide` and this report are
where the shape is written down.

This is the clearest Agent ≠ Model result across the three turns: the two
regressions that looked like consequences of unshackling were consequences of
the backend, and swapping a parameter fixed both without restoring a single
rule.

## Deus Ex Machina note

Two, both on the mediator agent's behalf, both handoff candidates:

- Fixed `agent/session.sh`'s glob resolution (F4) — the mediator could not
  launch at all and could not have fixed its own launcher.
- Rewrote the CHARTER.md opening sentence (F5) after cutting it too far.

## State after this turn

- `uv run pytest -q` → **77 passed**.
- Working tree holds all edits; nothing committed.
- `.local/jobs/palindrome` is a new converged job (9/9 gates), left in place
  as F5/F6's evidence. `.local/jobs/binary-cli`'s `iter-0001` summary was
  regenerated on the new extractor; the previous one is backed up in the
  session scratchpad along with the pre-turn `MISSION.md` / `NOTES.md`.
- `.local/agent/MISSION.md` now holds the palindrome mission, and
  `.local/agent/done` the note that closed it.
- The window's ollama default is now `qwen3.6:35b-a3b-coding-nvfp4`
  (`gateway.py`, `agent/README.md`, `agent/GUIDE.md`, tests).
- Gateway: every live check ran against a second instance on `:8799` started
  from the new code, deliberately leaving the developer's own `:8791` process
  alone. **Cleanup then killed both** — the `pkill` pattern matched the
  developer's foreground process too. `:8791` was restarted detached on the
  new code and answers `/healthz`, but it is no longer attached to the
  terminal it was started from. Reported rather than papered over: the
  cleanup step was wider than intended.
- `agent/session.sh` sessions 0005–0007 are zero-byte (the F4 exit-127 runs)
  and read as `unparsed` in `/status`.

## Next turn candidates

1. F7: force a multi-iteration job and watch whether the coding agent writes
   its own `NOTES.md`. This is the turn's central untested claim.
2. F5: re-run the mission a few times on the fixed charter to confirm the
   idle-session case is gone.
3. F6: one glob-resolution helper shared by `autolab`, `session.sh` and
   `gateway.py`.
4. A slow-brew mission end to end — the plan/approve/audit path has only been
   run on the fake adapter, so the deleted plan-review checklist has not yet
   been tested against a real coding agent's plan.
5. Re-run the window battery on qwen a few more times (F8 is 5/5 on one
   pass) and decide whether 28 s on the worst question is acceptable for a
   free local default, or whether a smaller qwen is the better trade.
