# scheduled_routine p1 — Phase report

Basis: three runs of `routine-imgprompt` on 2026-08-23 (15:46Z manual,
16:00Z scheduled, 16:41Z and 16:46Z manual), not the planned day of hourly
fires — the Developer switched to manual firing after run 1 because hourly
observation was too slow for this phase. Details per step in report1–4.

## What was built

- Definition: `#front` › `routine-imgprompt`, one post, the Developer edits
  it in place. Front never serves it (no `front-` prefix).
- Trigger: `pj-agdev/devenv/routine/trigger.sh <name>` posts one line as the
  Developer into `#front` › `front-routine-<name>`. Front is served by the
  existing rule; zero code change in Front, forge, pyagag, autolab.
- Schedule: `com.agdev.routine-imgprompt.plist.in`, hourly, log
  `agfront/.local/out/routine-imgprompt.log`. Verified by one fire
  (16:00:03Z, 3 s to Front). Currently **booted out**; re-enable with the
  install ritual in report3.

## Did Front need anything it was not given?

One thing, once: how to phrase a request to forge so that forge's
generator role does not choke on it. Front's first `assetplan-` ended with
"Please register this as a Work; once you open the run topic I'll trigger
it", forge's front role copied that into `required_items.md`, and forge's
generator declined ("no tool can register a Work"). Front asked the
Developer; the answer was "describe only the asset". Runs 2 and 3 show Front
applying that from topic history without it being in the standing text —
so the guidance is **not** needed in the standing text, and the real defect
is forge's (`assetplan_generator/guide_plan.md` never says writing
`plan.md` is the registration; `assetplan_front` copies requester prose
verbatim). That belongs to an agforge episode.

Candidate Evidence-Driven Guidance for the **standing text** (not Front):

- Say that the variation axis is the *prompt technique*, not the subject.
  All three runs used one four-slot template (painterly wide / photoreal
  macro / minimalist silhouette / flat-vector aerial) with the theme swapped
  in. Front even named the template in run 3. The text asks for "clearly
  different prompts" per run and "vary the theme" between runs; it got
  exactly that.
- Optionally: ask for one URL per image if that matters. The shape that
  emerged is one `assetplan-` → one Work → one zip of 4; Front reports one
  URL and one S3KEY for the set, file names distinguish images.

Not needed, kept as-is: the pre-stated forge spec (forge asked nothing in
three runs) and "don't wait for me" (Front never waited after run 1's
question was answered, and retried/stopped sensibly on its own).

## Front runs per routine run; loops and stalls

| run | Front runs | wall clock | notes |
|---|---|---|---|
| 1 | 7 | 15:46 → 16:04 (18 min) | forge misread (1 question), backend down (2 failed attempts), scheduled fire used as retry |
| 2 | 4 | 16:41 → 16:45 (4 min) | clean |
| 3 | 3 | 16:46 → 16:49 (3.5 min) | clean; plan-registration and button press in one callback |
| — | 1 | 16:31 | the Developer's judgement post is itself served by Front (reply: "noted, no action") |

No loop. One stall, external: SwarmUI on agpc was down (ComfyUI had been
launched in its place on 08-22); Front retried once, stopped after the
second identical failure and said so; the Developer started SwarmUI by
hand (~16:00Z) and the next fire cleared it. nctl reported `swarmui
converged` during the outage — reported to cagent by DM (message 1486).

Overlap (one data point): the 16:00Z fire landed while run 1 was stuck.
Front read the topic and retried the stuck Work instead of starting a new
theme. No lock needed so far; a fire landing mid-generation is untested.

Scheduler cost: a sonnet Front run per routine run plus one per callback
plus one per Developer comment — 3–7 per hour at hourly cadence, which is
fine by `localrule.md`.

## Second routine autolab-side?

**Yes**, a periodic "where do my plans stand" survey — but issued exactly
like this one: Front posts into each `pj-` channel's autolab entrance
(or autolab's own channel), autolab answers, Front reports home. Reasons:

- It exercises the `workplan-`/own-channel question path that p1 never
  touched, and autolab's whole-board survey (~220 s, ~$0.48) is the kind
  of work nobody remembers to ask for by hand.
- It needs no new entrance: a survey is a question, and autolab's intro
  says questions go to its own channel. Routines *issued by* autolab stay
  out of scope — the braindump's reason (a no-questions-asked path) still
  holds.
- It tests the one thing imgprompt cannot: a routine whose answer is
  text the Developer must read, not files to score. Whether the Developer
  actually reads an hourly survey is the finding to farm.

## Easier Next Time (not built in p1)

- `trigger.sh` could take the standing text's location from the plist so a
  second routine is one plist + one topic, no script edit (already true —
  only `<name>` is parameterised — but the `#front` home is hard-wired).
- A Developer comment in the run topic costs a Front run that says
  "noted". A convention such as a `[judge]` prefix that Front's sweep
  treats like a selfnote would remove it — but that is a Front/pyagag
  change, and only worth it if judgements become frequent.
- The `agentchat` in `agfront/.venv` lacks `wait` though devenv.md
  documents it; the venv is behind the pin. Sync before anything relies on it.
- forge: the two guide gaps above; and `unknown toolset 'toolset'` — the
  CSV header is read as a toolset name (log noise only).
- nctl/cagent: the swarmui `observe_only` http check did not surface a
  dead port as drift. A dump that says converged while the service refuses
  connections is the wrong answer for a GPU backend other agents depend on.

## State at phase end

- Routine definition live in `#front` › `routine-imgprompt`; run topic
  `front-routine-imgprompt` holds 3 runs and 1 judgement.
- Scheduler: plist in repo, job **not loaded** (manual `trigger.sh` for now).
- Commits: devdocs (reports), pj-agdev (`devenv/routine/trigger.sh`,
  `devenv/launchd/com.agdev.routine-imgprompt.plist.in`), both pushed.
