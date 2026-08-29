# scheduled_routine p8 — phase report

## Both braindump features, answered

**1. Is the level scale usable and honest?** Yes. The four-level contract
(`agautolab/agent/project_pattern.md`, `c6477ef`) was applied by an
autolab mission with no steering: it graded both existing "verified" tests
as **L1** in its plan before writing a line, and each `test.md` says
explicitly why it is not L2 and what would make it L2. The scale did what a
scale should — it turned "verified" (a state) into a claim about *how much*
was verified (a one-shot smoke token), and the INDEX column now says `L1`
where it said `verified`. Nothing was inflated; no prompt fight was needed.

**2. Did the independent publish gate catch what it should?** Yes, within
what these two papers offered. Fired as its own mission (`S3-12`) through
the production dispatcher → Front → autolab route, it:

- rewrote the one remaining local fact in each `test.md` (the internal
  Gitea repository name) into a generic sentence — condition 1;
- found that the Prime Agent summary's "First posted: 2026-08-05" was
  **wrong** (arXiv has a single v1 submitted 2026-08-24; the Developer
  re-checked this independently against arxiv.org) and that the Apodex
  summary had **no version line at all** (arXiv: v1 2026-08-24, v2
  2026-08-25) — condition 2, both fixed in the copy;
- judged the blockquotes as short attributed README excerpts — condition 3;
- **rejected all seven summary-only papers** under one stated rule
  (publication requires a completed local test) and wrote that rule into
  `publish/README.md`;
- committed `publish/` locally (`df91fd3`) and did not push; `main/` is
  byte-identical to before.

The main→publish diff is four one-to-two-line hunks. That readability is
itself the evidence the gate is doing surgical edits rather than rewriting.

## What the Developer's review caught that the routine didn't — and vice versa

Developer review of `publish/` at `df91fd3` against the three conditions,
independent of the mission's report:

- Condition 1 grep (`agstudio`, `.local`, `home.arpa`, `/Users`, `/tmp`,
  `localhost`, `host.docker`, `11434`, `gitea`, `autodev`, credential
  words): **no hits**. The only environment facts are "Apple-silicon Mac,
  local Ollama, model `qwen3.8:27b-mlx-bf16`" and upstream-documented
  placeholders (`OPENAI_API_KEY=your-key`, `sk-ant-...`) quoted from the
  projects' own READMEs. Nothing the routine missed.
- Condition 2: both version lines verified against arxiv.org submission
  histories; the routine's correction of the 08-05 date was right.
- Condition 3: the two `manual.md` files carry a handful of short quoted
  phrases (≤ ~40 words) from README/config docs, attributed, and none from
  the paper text. Acceptable under "avoid direct transcription in general";
  a stricter reading would paraphrase the longest Apodex GPU-compat
  excerpt — noted, not blocking.
- **What the routine missed: nothing that fails a condition.** Two soft
  observations for the standing text: (a) it did not say whether it
  checked the seven rejected papers' files at all — a rejected paper
  should still get its condition-1 grep so the rejection is "not yet
  publishable", not "unread"; (b) the harness result JSON came back null
  again, so "quote your own result JSON" should leave the standing text (see
  costs).
- **Over-rejection?** Rejecting all summary-only papers is a defensible
  rule and was stated plainly, but it is stricter than the braindump
  implies (the conditions are about content hygiene, not about having a
  local test). Whether summaries of runnable-but-untested papers should
  publish is a Developer policy choice; the routine made the conservative
  call and documented it, which is the right failure mode. Recommend the
  Developer decide and, if summaries should publish, say so in v2 of the
  standing text.

**Hand push:** pending — the Developer reviews `publish/` at `df91fd3` and
pushes by hand; autolab and the Omni Agent did not and must not.

## Costs and timings (from the run records; missions cannot quote their own)

| step | runs | cost_usd | model time |
|---|---|---|---|
| 2 backfill mission (plan + approve + task) | 3 autolab | 0.71 | ≈ 7.3 min |
| 3 publish fire (4 Front legs + plan + task) | 4 front, 2 autolab | 1.49 | ≈ 8.0 min |
| closeout requests via autolab's own channel | 1+ front-role | 0.39 + pending | — |

Wall clock: Step 2 mission 07:52 → 08:00Z; Step 3 dispatch → Front final
report 08:12:20 → 08:20:03Z. The `e42` fire was moved forward from 08:16 to
08:12 and the launchd dispatcher kickstarted at the user's request; it was
still the production dispatcher that marked and fired it.

Both missions answered the "quote your result JSON" instruction with
nulls and an accurate explanation: the harness writes those fields after
the run. The fix is listener-side (append `run-NNNN.json`'s
`cost_usd`/`num_turns`/`duration_ms` to the Zulip report) — a small
evidence-driven change for a later episode; until then the numbers are
read from `agautolab/.local/agent/<role>/run-*.json` and
`agfront/.local/agent/front/run-*.json`.

## Should `routine-publish` recur on a schedule? (recommend only)

**Stay manual-fire for now.** Reasons: (1) the gate's output is a local
commit that a human must push — a recurring fire that nobody reviews just
stacks unpushed commits, or worse, tempts automating the push; (2) new
material arrives only when a `localtest` run writes a `test.md`, which is
itself manual-fire (p6 verdict); (3) one fire is ~$1.5 and eight minutes,
cheap enough to run on demand after each local test. Revisit when
`localtest` recurs on a schedule; the natural shape then is a conditional
`decide` after each localtest fire ("if a new `test.md` landed, fire
`publish`"), which the schedule format already supports.

## Deus Ex Machina interventions

- Moved `e42` forward and kickstarted the dispatcher (`rtschedule move`,
  `launchctl kickstart`) for agent Front — handoff candidate: a Developer
  could have asked Front in `front-schedule` to move it.
- Started `workrun-task1-s3-10` and approved the Step 2 plan by hand as the
  Developer — expected per plan (p7 precedent), not a substitution.
- Asked autolab's own channel to close out S3-10/S3-12 instead of running
  `mission_done` as Omni; the first attempt ended mid-retry on a Plane rate
  limit (see below).
- No permission-classifier stops this phase.

## Findings for the standing texts and the harness

1. `routine-publish` v2 should: grep the rejected papers too; drop the
   result-JSON line; state the Developer's decision on summary-only papers.
2. Entrance runs that hit a transient error (Plane 429) end by promising
   to retry — but a finished run cannot. Either the entrance guide says
   "report what is not done, do not promise", or `mission_done` retries
   inside the call.
3. autolab's task resolved its own topic without the requester's sign-off
   (Front noticed); autolab's introduction says the opposite. One of them
   should change.
4. Mission texts that name `agent/project_pattern.md` should say it lives
   in the autolab checkout, not the project workspace (Step 2 spent ~8
   tool calls finding it).

## Close-out status (addendum)

Both missions are closed by autolab itself through its own channel
(`#autolab-agstudio1 › closeout-s3-10`): `S3-10` and `S3-12` **Work Done**
(message 2791, 08:24:19Z), both workplan topics resolved. It took four
prompts (2750, 2782, a Developer post 2785 at 08:21:30Z not sent by the
Omni Agent, 2789): the first two entrance runs ended promising a retry
that a finished run cannot make. Diagnosis: Plane CE limits each API key
to 60 requests/minute (`X-Ratelimit-Remaining` header, verified with a
read-only call); an entrance run's board sweep plus `mission_done`'s own
state/project list calls exhaust it, so the *verification* sweep after a
successful mark is what 429s. Told to call once and report headers
instead of looping, the run succeeded on the first try. Findings 2 above
stands; add: `mission_done` (or the entrance guide) should budget Plane
calls, and a 429 after the mark is not a failure of the mark.

Costs of the four closeout runs are entrance-front runs
(`agautolab/.local/agent/entrance_front/run-0016` = $0.39 for the first;
the later three not itemised here).

**Remaining for the Developer:** review `publish/` at `df91fd3` and push it
by hand (`git -C agautolab/.local/projects/studyarxiv/publish push origin
main`). Nothing else in this phase is open.
