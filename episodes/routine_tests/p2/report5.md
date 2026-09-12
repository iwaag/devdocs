# routine_tests p2 step 5 — publication prepared, and a second routing defect

Date: 2026-09-13 JST (timestamps UTC). Plan: [plan.md](plan.md) step 5.
Previous: [report4.md](report4.md).

## The request

`#front` › `front-publish-uspolitics-20260912T1635Z`, message **6478**:
run `publish` once, for **`studyuspolitics` only**. It named the two things
that are specific to this project because its publication repository was
supplied by hand — that `publish/` already holds a CC0 `LICENSE` commit
whose history must be preserved, and that `publish/` must not be pushed —
and said nothing else about the gate.

Front read **guide 6254 (v2)** and opened
`#routine-publish` › `routinerun-20260912T1636Z`.

## The project was found without being in the table

This was the point of running `publish` against a project the guide has
never heard of. Guide v2 says its table *"is not the definition"* and that
the set is discovered from the workspace and each `README_PROJECT.md`.

It worked. autolab discovered the candidate set from the project itself —
**3 files**: the two method notes in `main/methods/` and the one report in
`main/reports/` — and said so explicitly, *"per the `study` pattern,
discovered from the project, not the guide's table"*. The p1 defect, where a
stale table narrowed coverage, did not recur.

## The gate had real work to do

All three files failed **check 4 — no internal-workflow residue**, and each
needed the related **check 1** fix. autolab identified this before running
anything, from a first read:

- "Parallelization and isolation" / "Subagent parallelization" sections
  narrating subagent dispatch and lead-session mechanics — process, not
  findings, and meaningless to a stranger;
- references to a "retired prior attempt" — internal history of this trial;
- references to gitignored `.local/…` scratch paths by name.

The fixes were made **in `main/` first**, by three parallel subagents (one
per file), then copied unchanged. The diff is net-subtractive where it
should be — `ac2f7d2`, 64 insertions against 142 deletions — because
process narration was cut while the substance was rewritten rather than
dropped. autolab reports that material with evidentiary weight (the 9-for-9
cross-check correspondence between the inactive flag and the missing rows)
was **folded into the findings prose** rather than deleted, which is guide
v2's "a check never deletes knowledge; it rewrites it" rule.

**No provenance or quote-hygiene failures.** The plan's warning was
observed too: no paper-specific version line and no local-test requirement
were invented for a political report.

## Verified independently

Checked against the trees, not against either agent's report:

| Check | Result |
|---|---|
| `main/` and `publish/` identical (README excepted) | all 3 files byte-identical |
| Publication history preserved | `569a8e3` "Initial commit" (CC0 `LICENSE`) still the root |
| `publish/` **not** pushed | origin still at `569a8e3`; local clone **ahead 1** |
| `main/` committed **and** pushed | `ac2f7d2` is `origin/main`'s HEAD |
| Internal-workflow residue in `publish/` | 0 hits for subagent / routine / mission / workrun / workplan / autolab / gitea / hostnames / absolute paths |
| Relative links resolve | the published tree contains **no** markdown links at all, so none can dangle |

Four remaining occurrences of `DEMO_KEY` in a method note are **not
residue**. It is the name of FEC's own public shared API key, and it appears
in the explanation of why the bulk-download route was chosen over the keyed
endpoint — exactly the "what the run learned about retrieval" that guide v2
says to keep rather than cut. It is neither a secret nor a local-environment
fact.

`publish/README.md` (17 lines, written fresh) says what the repository holds
and what `methods/` and `reports/` each contain, and mentions nothing about
`main/`, routines, missions, reviews or internal repositories.

## Commits, and what is left for the developer

| Repository | Commit | State |
|---|---|---|
| `main/` (internal Gitea) | `ac2f7d2` — *Apply publish gate: strip internal-workflow residue for public release* | committed **and pushed** |
| `publish/` (`github.com/iwaag/study-uspolitics`) | `d467141` — *Add methods and reports: 2026 Senate campaign-finance findings*, 4 files / 524 insertions | committed **locally only** |

**Remote publication has not happened.** The clone is one commit ahead of
its origin and stays that way; the final push is the developer's, by hand,
under the study/publish contract. The supplied GitHub URL identifies the
destination; it did not override that contract.

The clone was not divergent and held no pending publication work — just the
one `LICENSE` commit, which is now the parent of `d467141`.

## The defect this step found

**A run that delegates in the same serving that opens it anchors the
delegation to the Front Desk, not to the run.**

Front's `front` role opened the run and delegated in one serving. The
`[selfnote][rootchat]` note it wrote into
`#pj-studyuspolitics` › `workplan-publish-studyuspolitics` (message
**6482**) names the **Desk** conversation, because that is the conversation
the `front` role was serving. The `routine_run` role that actually drives
the run never got an anchor there.

Everything downstream followed from that, silently and correctly at every
individual step:

- autolab's plan callback served the **Desk**, so the `front` role received
  it and asked the Developer whether to proceed — while the run sat at
  message 6488 saying it had seen *"only autolab's initial acknowledgment …
  no plan"*.
- The task was therefore **never started** until a human answered.
- After the task finished, its completion callback served the Desk again.
  The run still knew nothing.
- The `front` role produced a complete, accurate report to the Developer
  (message 6514) — so the *work* was never at risk. What was lost was the
  run: no entries, no finish block, no delivery, no ✔.

**This one was not caused by any intervention.** The request went in at the
ordinary entrance and nothing else was done. That distinguishes it from
report3's defect, which an intervention triggered.

It is the same family as report3's — a routing anchor that does not survive
a change of context — but a different mechanism: there, a *rename* moved
the anchor out of the live topic; here, a *role* wrote the anchor to its own
conversation rather than to the run it had just opened. Both are owned by
the boundary between who writes the anchor and who reads it.

**Adding a second anchor is not enough.** The run's own anchor was written
into the same topic at message **6500**, and the completion callback *still*
resolved to the Desk — the Desk's note was written first and kept winning
the lookup. So the repair that worked in report3 (restore the missing
anchor) does not work here (supersede an existing one).

Per the developer's decision, the root cause is left to a later phase and
this trial's work was finished by hand.

## Interventions — this step was assisted

Three more, bringing the episode to seven:

5. **Answered Front's question and corrected the routing** (message 6501).
   Front explicitly asked whether to start the task; answering a supervisor's
   direct question is ordinary, but it was only *necessary* because of the
   defect.
6. **Wrote the run's anchor into the delegation topic** (message 6500, as
   Front). It did not take effect, for the reason above.
7. **Posted the verified state into the run topic** (message 6516) so the
   `routine_run` role could judge and close. This is the one that worked:
   the run wrote its finish block, delivered its report to the Desk and
   resolved — and its own finish text names the anchoring mismatch as the
   reason it had been blind.

> **Deus Ex Machina note:** *served the publish run directly and repaired its
> delegation anchor for agent Front — handoff candidate.*

## Backends and cost

All `claude_code` on `anthropic/claude-sonnet-5`, `exec_source: default`.

| Agent | Runs | Cost |
|---|---|---|
| Front (`front` ×5, `routine_run` ×2) | 7 | $1.6212 |
| autolab (`superdirector` ×1, `supercoder` ×1) | 2 | $1.2425 |
| **Publish step total** | **9** | **$2.8637** |

Five `front`-role runs is the cost of the defect: the Desk handled the plan,
the question, the answer and the completion, none of which it should have
seen.

Episode running total: **32 agent runs, $11.4654**.

## Step 5 conclusions, kept apart

1. **A project absent from the guide's table was discovered and served**,
   from the workspace and `README_PROJECT.md`, exactly as guide v2 intends.
2. **The four checks were applied and bit.** All three files carried
   internal-workflow residue; it was fixed in `main/` first and copied
   unchanged, with evidentiary material rewritten rather than deleted.
3. **Publication is prepared, not performed.** `publish/` holds `d467141`
   locally, one commit ahead of the supplied GitHub repository, whose
   existing CC0 history is intact. The remote push is outstanding and is
   the developer's.
4. **`main/` and `publish/` agree**, verified file by file here rather than
   taken from the report.
5. **A second routing defect is demonstrated and unfixed**, and unlike the
   first it arose with no intervention to provoke it. The run completed only
   because a human served it directly.
