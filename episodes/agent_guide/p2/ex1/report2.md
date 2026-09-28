# agent_guide p2 ex1 — step 2: a trial touches nothing real

pyagag `2b18e66` (records root) and `dd8acfe` (repositories, journal);
pinned in agfront `cc108bd`, agautolab `54741e3`, archsage `b2c8bd9`,
pj-agdev `cbecc9f` (agobserver).

## Run records and workspaces: the kit's own root

`AgentSpec.topics_root` and `records_root` are now read through
`AgentSpec.runs_local`: the checkout's `.local/`, or, when
`AGAG_RECORDS_ROOT` is set, `$AGAG_RECORDS_ROOT/<agent>/.local/`. The
credentials, instance name, mirror and `agrefs` home stay the checkout's;
only where servings run and what they record moves. The `.local/` segment is
kept inside the trial root so `workspace_identity` still reads a run's
channel and topic off its path and the record carries them.

`Trial.start` sets the variable for the process, to `--records <dir>` or by
default `<out>/records`. agfront's and autolab's listeners keep both roots
as module constants taken at import, so their drivers also set those two
constants from the spec after the trial starts. This mattered: in a test
process the listener module is already imported, and without that
assignment agfront's dry run would have written into the checkout.

The relay's cost gauge (`agentroom.cost`) and in-flight view
(`agentroom.inflight`) walk `<root>/.local/agent` and `<root>/.local/topics`
for the roots in `AGENTROOM_AGENT_ROOTS`, which are the agent checkouts. A
trial root under `--out` is none of them.

**Test**: each driver's `tests/test_trial.py` runs its probes with
`--dry-run` from the main checkout and asserts the set of files under that
checkout's `.local/agent/` and `.local/topics/` is unchanged, and that the
record and chatlog are under `<out>/records` (agfront, autolab ×2,
Observer, archsage: all pass).

## Gitea: the fixture answers for its own repositories

The board carries repository rows (`Board.repositories`, stored as the
`fixture_repositories` meta): `aisvgs` at `f57eed1a27de` (after round 2,
the revision its posts report) and `growbox` at `51ab2e0c77d1` (after the
food-safety strand; the control loop is still running). `worldtrend` has
only its plan and no repository. `MirrorReads.repository(slug, org)` answers
them in `gitea_head`'s shape at `https://gitea.fixture.invalid/<org>/<slug>.git`,
a host that resolves nowhere. `agag.project.inspect_project` asks the
client when it is a fixture and the host's Gitea otherwise.

Checked with `AGAG_GITEA_URL` pointed at a closed port, so a Gitea call would
have shown up as an error:

```
#pj-aisvgs … study, state: ready
  main: https://gitea.fixture.invalid/autodev/aisvgs.git at f57eed1a27de
#pj-worldtrend … study, state: document-posted
  gitea: {"exists": false, "repository": "https://gitea.fixture.invalid/autodev/worldtrend.git"}
```

The fixture answers the repository question, as the plan preferred to
pointing Gitea at nothing: a probe about a project's state sees a
repository. Test: `test_agproject_status_reads_the_fixtures_repositories`
(the host's `gitea_head` replaced by one that fails the test if called).

## One more real thing a trial read: the listener journal

`chat_environment` gives every run `AGENTCHAT_JOURNAL`, the live listener's
`listener.sqlite`, which `agentchat receipt` reads for evidence. A trial
run's `receipt` (the `receipt-owed` probe calls it) was reading the real
journal. It opens the journal read-only, so nothing was written, but a
trial's answer could depend on the live board. `fixture_environment` now
points it beside the board (`<board dir>/listener.sqlite`, absent, so
nothing counts as evidence), and the fixture test asserts it.

Not changed: `AGREFS_HOME` stays the checkout's `.local/`. `agrefs` reads
the human references the host has, which a trial is meant to see, and may
fill its snapshot cache there. That cache is outside `agent/` and `topics/`.

## Tests

pyagag **1164 passed**; each driver's `test_trial.py` passed after the pin.
