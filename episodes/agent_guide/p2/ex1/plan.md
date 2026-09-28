# agent_guide p2 ex1 — a trial kit anyone can run again

## Goal and scope

p2 built a fixture board so a guide change can be tried against the same
board every time. Make that true for every agent and every run of the kit:
the drivers are versioned, a trial touches nothing real (no Gitea, no live
run records), the fixture can answer a delegation so the callback path is
testable, and the Comfy Notifier's `watch` command is learnable from the
board.

Read `../report.md` (§ *The fixture, and how to run it*, § *Open findings*
3–5) and `../report6.md`, `../report7.md`. README_DEV § *Trying a guide
change against the same board* is the current instruction.

Private experimental, breaking-change phase: no backward compatibility.
The implementer chooses module layout, command names and the fixture's
shape. No guide text changes in this ex unless a step below needs one.

## step1 — version every trial driver

- Front's driver is `agfront.trial` (versioned, 103 lines). The others are
  only in `pj-agdev/.local/agp2/trials/probe_others.py` (ignored, 149
  lines): archsage `archsage-loose-sage`, autolab `planner-other-project`
  and `entrance-plans`, Observer `triage-unopened`. The p1 guide tree used
  as the baseline is in `.local/agp2/trials/p1-guides/`, also ignored.
- Move each agent's driver into its own package (`archsage.trial`,
  `agautolab.trial`, `agobserver.trial`), mirroring `agfront.trial`.
  Both drivers carry the same `newest` / `tool_calls` helpers: move those
  into `agag.fixture.run` once.
- Replace the baseline copy with something reproducible from git: a
  `--guides-rev <commit>` option that materialises the guide tree at that
  commit (`git archive` or a worktree) into a temp directory is enough.
  p1's baseline is agfront `ba28e90^`; p2's is the commit before
  `agent-guide-p2` merged (report7 § Roll-out has the merge commits).
- One entry point that runs a named probe for whichever agent owns it is a
  convenience, not a requirement.

## step2 — a trial touches nothing real

- **Gitea.** `agproject status` still calls `agag.project.gitea_head`
  (`pyagag/src/agag/project.py`, `gitea_base` reads `AGAG_GITEA_URL`)
  under the fixture, so a fixture study named like a real one gets real
  repository facts. Make the fixture answer it: fixture repository rows in
  `agag.fixture.board`, served when the store is a fixture (`chat.py`
  already checks `found.fixture` around line 288), or `AGAG_GITEA_URL`
  pointed at nothing and the status saying so. Prefer the fixture
  answering: a probe about a project's state should see a repository.
- **Run records.** A trial started from a main checkout writes topic
  workspaces and `run-NNNN.json` under that checkout's `.local/`, and the
  relay's cost gauge (`agentroom/src/agentroom/cost.py`, globs
  `<instance>/.local/agent/<role>/run-*.json`) and in-flight view
  (`inflight.py`) count them as live. p2 avoided it by running from
  worktrees. Give the kit its own records root (an env var or option read
  where `agag.agent` / `next_record_path` choose the directory), defaulting
  to somewhere under the trial's `--out`.
- Add a test that runs one probe with `--dry-run` from a main checkout and
  asserts nothing new appeared under that checkout's `.local/agent/` or
  `.local/topics/`.

## step3 — the fixture answers a delegation

- The fixture refuses every write, so no probe can delegate, and the
  callback path (a serving ends; the answer arrives as a new serving) is
  covered only by p1's live mission (report7 § *Paragraphs that moved*,
  row as5-8).
- Add a scripted responder: an `agentchat send` to a fixture agent's
  entrance is recorded in the trial's own overlay (not the fixture store)
  and answered with a canned post after it, so the driver can serve the
  callback as a second serving. Scripts per probe; one canned answer and
  one "I need a decision" question are enough to start.
- Keep writes other than `send` refused, unless a probe needs one; the
  point is that the board stays the same between runs.
- New probes: Front delegates a small request and reports the canned
  answer on the callback serving; Front answers a response_request from
  the fixture agent. Pass rules judge the tool calls and the reply, as p2's
  rules do (required facts vs observed facts).

Hint: the stub profile (`[profiles.stub]`, harness `fake`, in
`agfront/agents.toml`) serves without a model; useful for testing the
responder and the driver loop before paying for a real run.

## step4 — an introduction for the Comfy Notifier

- comfynotify posts nothing in `#agents`; autolab's worker guide is the only
  place its `watch` command is described (`@**Comfy Notifier** watch
  <prompt_id>`, acknowledged with a reaction, public channels only, refused
  once per topic when it cannot read it; README_DEV § *ComfyUI notifier*).
- Give it an introduction through `agag.intro.post_intro` (pyagag
  `intro.py`): who it is, the one command, what comes back, that it is a
  tool, not an agent you converse with. Post it at start-up like the
  agents do. comfynotify still pins an older pyagag (p2 report7 § Roll-out);
  update the pin with it.
- Then shorten autolab's worker guide to a pointer at the introduction,
  keeping any sentence with a trial behind it (report1 of p2 has the
  worker's paragraph keys).
- Mind that a mention of the notifier inside a sentence is not a command,
  and one inside a code fence never fires; the introduction should quote
  the command in a fence.

## step5 — run it and report

- Re-run every p2 probe through the versioned drivers from a main checkout,
  new guides only, plus the step3 delegation probes, and one baseline run
  of the #15673 probe with `--guides-rev` to show the baseline works.
  Expect roughly p2's cost (~$6 for the full set); the stub harness is free.
- Run the affected suites (pyagag, agfront, agautolab, agobserver,
  archsage, comfynotify, agentroom). Read the summary line.
- Check the relay's cost gauge after the runs shows no trial spend.
- `report.md`: where the drivers are, how to run a probe and a baseline,
  what the responder can script, the results, and README_DEV § *Trying a
  guide change against the same board* updated to match.
