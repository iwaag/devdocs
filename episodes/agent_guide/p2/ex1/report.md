# agent_guide p2 ex1 — report

## Outcome

The trial kit p2 left on this host is now versioned and can be run again
by anyone, from a main checkout:

- **Every driver is in its agent's package**: `agfront.trial`,
  `archsage.trial`, `agautolab.trial`, `agobserver.trial`. All four share
  pyagag's kit (`agag.fixture.run`).
- **A baseline is a commit**: `--guides-rev 447bb03 --no-shared` rebuilt
  p1's composition to the character (19 534), where p2 had kept a copy
  outside git.
- **A trial touches nothing real**:
  - servings and run records go under the trial's own root, not the
    checkout's `.local/`;
  - the fixture answers `agproject status`'s repository question itself;
  - a run's `agentchat receipt` reads a journal beside the board, not the
    live listener's.
  - Checked after about $6 of trial runs: the relay's cost gauge unchanged
    and no new files in any checkout.
- **The fixture answers a delegation**: on a trial's own copy of the
  board, a `send` is answered by the probe's script after the serving ends,
  and the conversation is served again, as the listener serves a callback.
  `delegate-decision` passed the whole loop 3 of 3 times: ask, receive a
  decision question, answer it in autolab's topic, report the result.
- **The Comfy Notifier introduces itself** on `#agents`, and autolab's
  worker guide points at it instead of describing the command.

Step reports: `report1.md` (drivers, kit, baselines), `report2.md`
(records root, Gitea, journal), `report3.md` (responder), `report4.md`
(notifier introduction), `report5.md` (runs, checks, suites).

## Where the drivers are, and how to run a probe

```
python -m agag.fixture probes                      # every probe, its rule, and the line that runs it
cd <agent checkout> && .venv/bin/python -m <agent>.trial <probe> --out <dir>
    [--store <mirror.sqlite>]                      # default: a fresh board under <out>/board
    [--guides <tree> | --guides-rev <commit>]      # another guide tree; from git at that commit
    [--no-shared]                                  # without pyagag's shared sections
    [--dry-run]                                    # write <out>/prompt.md, run no model
    [--records <dir>]                              # default: <out>/records
```

| agent | driver | probes |
|---|---|---|
| Front | `agfront.trial` | `aisvgs-sufficient`, `growbox-thing`, `forge-protoprey`, `receipt-owed`, `hold-release`, `delegate-answer`, `delegate-decision` |
| archsage | `archsage.trial` | `archsage-loose-sage` |
| autolab | `agautolab.trial` | `planner-other-project`, `entrance-plans` |
| Observer | `agobserver.trial` | `triage-unopened` |

**A baseline**:

- p1's composition: `cd agfront && .venv/bin/python -m agfront.trial
  aisvgs-sufficient --out <dir> --guides-rev 447bb03 --no-shared`;
- before p1: `--guides-rev 'ba28e90^' --no-shared`;
- other agents before p2: archsage `df6e0af`, autolab `aab6810`.

The extracted tree and a `REVISION` file stay in `<out>/guides@<rev>/`.

`<out>/outcome.json` holds:

- the verdict, and the required facts met or missing;
- facts observed outside the rule;
- cost, turns, duration and model;
- the tool calls;
- which guides, board, overlay and records root were used;
- for a conversation, every serving's reply and calls, and the posts sent.

## What the responder can script

A probe's `script` is a tuple of `Canned(agent, text, topics=())`.

**Where it applies.** It works only on an overlay, the trial's own copy of
the board. On an overlay:

- `agentchat send` is recorded (the root note, then the message);
- `send --to <name>` resolves against the board's users;
- every other write stays refused.

**What takes a line.** A post the served agent sends that a listener would
be served by: it mentions the agent, or is in its channel, or is under one
of its topic prefixes. That post takes the agent's next unused line. A
`to=` alone reaches nobody (report5). `{ask}` in the line is the id of the
post it answers, and `{asker}` the asker's mention. A line can be:

- an answer (with `ag-post intent=report re={ask}`);
- a question back (`intent=response_request to=<asker> ask=decision`),
  whose answer then takes the next line.

**When it arrives.** Lines are posted after the asking serving ends. The
kit then serves the home conversation again with the calling topic as
`extra_threads`, up to four servings.

**How it is judged.**

- `must` / `must_not`: the last serving's reply;
- `tools_must` / `tools_must_not`: every serving's tool calls;
- `sends_must`: the posts sent;
- `servings_min`: the answer came back on a serving of its own.

## Results

Full tables in `report5.md`. New guides, main checkouts, pyagag `3a16479`.

| probe | result |
|---|---|
| `aisvgs-sufficient`, `growbox-thing`, `forge-protoprey`, `receipt-owed`, `hold-release` | all pass (8–15 turns, $0.15–$0.28) |
| `aisvgs-sufficient` at `447bb03 --no-shared` (baseline) | pass, 7 turns, $0.18 |
| `archsage-loose-sage`, `planner-other-project`, `entrance-plans`, `triage-unopened` | all pass |
| `delegate-decision` | 3 of 3 (step 3 once, step 5 twice), three servings each |
| `delegate-answer` | Front delegated in 3 of 5 valid step-5 runs (3 of 6 with step 3's); one run void (below) |

The ex spent about **$6.9**: step 3 $1.30, step 5 $5.57. The triage ran
on the local model; the stub loop is free.

- **The #15673 probe observed the decision**, once. With the new guides it
  read `archsage-agstudio1 › study-aisvgs-round3` and carried the
  Developer's decision to do round 3 by hand. That had never happened on
  the fixture (p2: 0 of 2); the baseline run beside it missed it. It is
  observed, not required, and one run is not a rate.

## What this ex learned

- **Front often does not ask when it can read.** Asked to "ask autolab"
  about finished work, Front read the ✔ mission topic and answered "no
  stall on record" instead, in 3 of 6 runs. Asked about running work, it
  cited `board.md`'s as9 sentence and asked nobody (step 3's first
  probe). Every such reply was honest about what the record says. None did
  what the Developer asked. Two paragraphs with trials behind them (as9: a
  "how is it going?" restarts their job; fd-wr: ✔ is finished) are read
  as reasons not to post at all, even when the Developer names who to ask.
  No guide was changed here. `delegate-answer` now measures it.
- **A fixture that answers must answer only what the realm would.** The
  responder was wrong twice, and both times the run looked like a pass:
  - an answer posted inside `send` was read in the same serving, so the
    callback carried nothing new;
  - a `to=` with no mention was answered, though no listener would have
    been served.

  Each was found by reading the tool calls behind a passing verdict,
  never by the verdict itself.
- **An introduction without a roster is a visible fault.** The
  operation room lists every `intro-` topic as an instance, and one
  without a roster block is an unknown row. The notifier's roster declares
  no channel and no prefixes, so it shows as `ok`, with nothing to serve.
- **The agents do not post their introductions at start-up**, though the
  plan assumed so. Each has an `intro` command run by hand. The notifier's
  daemon posts at start-up when the board's copy differs, so a restart
  stacks nothing.

## Open findings

1. **Delegation is skipped when the board looks sufficient** (above): 3 of
   6 `delegate-answer` runs answered from the record instead of asking the
   agent the Developer named. This is a guide question (as9 and fd-wr read
   as "do not post"), for the phase that owns the text. The probe is ready
   to measure a change.
2. **p1's open finding 1** (a study decision recorded only in archsage's
   channel) was found once with the new guides and never with the baseline.
   Repeat the pair several times before reading anything into it.
3. p2's open finding 1 (a reply claiming records it never made) is
   unchanged. The fixture's overlay could stage it, but no probe here does.
4. comfynotify's roster declares no channel; the cost gauge still names it
   `missing`, as it does agobserver, archsage and cagent, because no
   `AGENTROOM_AGENT_ROOTS` entry exists for them on this host. That is the
   gauge's existing rule, not new.

## Documents updated

- `devdocs/README_DEV.md`:
  - *Trying a guide change against the same board* is rewritten: drivers,
    baselines as commits, what a trial does not touch, and delegation
    probes with the responder;
  - *ComfyUI notifier* gains the introduction.
- Host notes (ignored `pj-agdev/.local/devenv.md`): *agent_guide p2 ex1 on
  this host*, covering the version set, driver lines, outcome folders and
  the notifier introduction.

## Deus Ex Machina notes

- The Omni Agent restarted the Comfy Notifier twice so that its daemon
  would post its own introduction. The post and its content are the
  notifier's, made by its own code on its own credential. Nothing was
  posted for an in-system agent.
- No trial post reached the realm: every run read the fixture or its
  overlay.
