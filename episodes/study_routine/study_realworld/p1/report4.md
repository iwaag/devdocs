# Step 4 — publication

`https://github.com/iwaag/study-realworld.git` now holds the reviewed
collection: three source recipes, five investigation reports, both index
levels and a stranger-facing `README.md`. The `publish` routine's standing
request was rewritten twice in the process — once to stop being about arXiv,
and once because its first run under the new text cut real knowledge out of
`main/`.

## The standing request, v6

`#front › routine-publish`, message **5416**, 7 187 bytes. v5 was entirely
arXiv-shaped — `#pj-studyarxiv`, `main/papers/`, "the paper's version is
clearly stated", `publish/` never pushed. v6 keeps every v5 rule that was
about *how* to publish and generalises every one that was about *papers*:

- **Projects are a table, not a hard-coded channel.** `studyarxiv` and
  `studyrealworld`, each with its candidates, its publication origin, and
  its push rule; the run's own topic may name one, and covers both if it
  names none.
- **The push rule is per project.** `studyarxiv` is unchanged — commit
  `publish/` locally, never push. `studyrealworld` commits *and pushes* to
  its GitHub origin, and the request says plainly that this is a deliberate
  departure from the pattern document's older "never pushed by an agent"
  convention, made by the developer for this project, and that
  `README_PROJECT.md` still carries the old line and must be corrected in
  the same run.
- **Check 2 became "provenance is stated"** rather than "the arXiv version
  line is present": for a `studyrealworld` report it is the per-document
  metadata the project README requires; for a source file it is the entry
  points, the worked example and the last-verified date.
- **Check 3 says "third-party document"** rather than "the paper", so
  minutes and press releases are covered by the same quote hygiene.
- Two additions of its own: **every relative link must resolve inside the
  published tree**, and **`README.md` is the one file allowed to differ**
  between `main/` and `publish/`, because the published one is written for a
  stranger.

## The first run

Fired at **16:57Z** into `front-routine-publish-2026-09-08T16:57Z`. This one
was posted by hand rather than through `trigger.sh`, because the fire needed
to carry one thing the trigger script cannot: *"This run covers
`studyrealworld` only. Leave `studyarxiv` alone entirely."* Without that,
v6's default would have re-gated ten arXiv papers for no reason.

Front relayed v6 into `pj-studyrealworld › workplan-publish-realworld`
essentially in full — including the push rule and the `README_PROJECT.md`
correction — and, unlike the two study runs, **asked for the plan first**
because v6 says the run reports its plan before starting.

autolab planned 13 candidates, 13 concurrent subagents, and it checked its
own concurrency claim with `ListAgents` rather than asserting it. Front
approved; the task ran 17:00:39 → 17:07.

**What the gate actually found — 5 of 13 files needed work:**

- `fed-fomc-minutes/2026-09-09-001` and `pew-research-reports/2026-09-09-001`
  had **no citation-metadata block at all**. The discovery run had written
  the metadata into the source files and the report bodies but never as the
  structured block the project README requires; the two later
  (investigation) reports had it. So check 2 caught a real, systematic gap
  in the older half of the collection — which is exactly what a gate is for.
- `pew-research-reports/2026-09-09-002` and both `reliefweb-updates`
  reports had text cut under check 4 (see below).
- autolab also caught and fixed an internal file path that **its own**
  reliefweb-001 fix had introduced, before copying.

**Results:** `main` `8f8c46b` (pushed to Gitea), `publish` `cc37a74` pushed
to `https://github.com/iwaag/study-realworld.git` — 14 files, 713
insertions. `diff -r` clean. Verified independently afterwards: the GitHub
remote's `main` was `cc37a74`, `main/sources` and `main/reports` were
byte-identical to their `publish/` counterparts, no relative link in the
published tree dangled, and a grep for hostnames, local paths, ports,
internal repository names and internal vocabulary found nothing.

`publish/README.md` explains the collection to someone who has never heard
of any of this — what a source file is, what an investigation report is, and
how to use the two indexes — with no mention of `main/`, routines, missions
or reviews.

## The defect: check 4 cut knowledge

Reading the `main` diff rather than the report is what caught it. Under
"no internal-workflow residue" the run had deleted, from
`pew-research-reports/2026-09-09-002`:

- `Follows: [2026-09-09-001](../2026-09-09-001/report.md)` — the link to the
  investigation it follows, which the routine's own standing request
  *requires*;
- the entire `## Question` section, which the project README requires.

And from `reliefweb-updates/2026-09-09-002`, a `## Status` section whose
substance was a genuine acquisition finding: *the individual ReliefWeb
report page also returns HTTP 403 to a bare fetch and needs the same
browser-like User-Agent as the feed.* The source file documented that
requirement only for the RSS URL, so after the cut it existed nowhere.

Worse, it was **inconsistent**: `reliefweb-updates/2026-09-09-002` kept its
`Follows:` link and its `## Question`, and the Pew sibling lost both, so two
reports of the same kind came out of the gate with different structure.

The root cause is in the request, not only in the run. Check 4's examples
were all about *a private workspace leaking into a published file*, and its
one worked example was a sentence referring to task instructions — which a
`## Question` section, phrased as an instruction, superficially resembles.
Nothing in v6 said that a check may not delete something a later run needs,
and `main/` is the only copy.

## v7 and the correction run

**v7** (message 5443, 8 871 bytes) adds two things to v6:

- a general rule at the top — ***a check never deletes knowledge; it
  rewrites it***. If a sentence fails a check but carries something a later
  run would want, move it where it belongs (usually the source file) instead
  of deleting it, and say what was moved;
- a **"What check 4 is not"** block naming the three things that stay: the
  `Follows:` link, the report's own `## Question`/scope section (reword an
  instruction-shaped question into a question; do not delete it), and what
  the run learned about retrieval. Plus: two reports of the same kind must
  come out of the gate with the same structure.

The correction run (`front-routine-publish-2026-09-08T17:09Z`) was scoped to
the damage only — explicitly *not* re-gating the eight files that passed.
autolab recovered the wording from `git show 8f8c46b` rather than rewriting,
and checked the other three touched files, reporting that all three diffs
were additions-only and needed nothing. Results: `main` `ca3b25f`,
`publish` `5718242`, both pushed; `diff -r` clean again.

Independently verified afterwards: `main/sources` and `main/reports` are
byte-identical to `publish/`, both remotes hold the new commits, and
`sources/reliefweb-updates.md` step 3 now carries the report-page
User-Agent note.

## Findings

**1. The gate's own output must be diffed, not just believed.** Front's
close-out of the first run was accurate about everything it mentioned — 13
gated, 5 fixed, `diff -r` clean, pushed — and said nothing wrong. The defect
was invisible at that altitude because "internal-workflow residue stripped"
is exactly what a correct run also reports. Only `git show 8f8c46b` showed
that three of the "-" lines were things the project requires. **A supervisor
that reads reports cannot audit a gate that edits files**; this is the
`supervisor-cannot-see-artefacts` shape, in the publication path.

**2. A reused workplan topic misroutes every callback.** The correction went
into the existing `workplan-publish-realworld` topic, whose `[rootchat]`
selfnote names the conversation that opened it — the **16:57Z** run. So
autolab's reply served that topic instead of the 17:09Z one, and because
16:57Z was resolved, the run read *"no messages"* and answered nothing:

```
17:14:18Z mention in 'pj-studyrealworld'/'workplan-publish-realworld' serves front/front-routine-publish-2026-09-08T16:57Z
17:14:18Z nothing to answer in 'front'/'front-routine-publish-2026-09-08T16:57Z': no messages
```

The mission was fully planned with `start.flag` created and simply sat
there; nobody would ever have posted into `workrun-rerun-task1-s4-5`. One
Developer post naming the situation got Front to start it from the right
topic — and Front's own post into the *new* run topic anchored that one
correctly, so the execution callbacks came back to the right place. **One
topic has one home, and a second routine run through the same workplan topic
inherits the first run's home.** A new run should open a new workplan topic,
or the anchor should be rewritten when a topic is reused.

**3. The close-out defect from step 3 reproduced, twice.** Front again
reported completion and announced resolves it had not performed — once after
the first publish run's correction request, once after the correction pass.
Two Developer nudges were needed. Two occurrences on two different routines
make this a behaviour rather than an accident: *an agent that says it will
resolve a topic does not verify that it did before ending its run.*

**4. Front hit an account session limit mid-close-out.**
`front run exited 1: You've hit your session limit · resets 2:50am
(Asia/Tokyo)` at 17:17:47Z, on the nudge asking for the two resolves. The
work was already complete and pushed; only the bookkeeping was blocked. Two
things are worth recording: the listener posts the failure into the topic as
Front's own message, which makes **Front the last poster**, so the topic is
never re-served and the request is dropped silently until a human posts
again; and a quota exhaustion is indistinguishable, from the board, from an
agent that simply stopped.

After the reset, one Developer post (5474) saying the failure was a quota
and nothing needed re-doing brought Front back, and it resolved both topics
within 25 s — `front-routine-publish-2026-09-08T17:09Z` and
`pj-studyrealworld › workplan-publish-realworld` are ✔. Every topic this
phase opened is now resolved except `workplan-create`, which is the project
setup conversation and belongs to no run.

## Cost

Twelve runs across both publish runs, **$5.28** total:

| run | what | cost |
|---|---|---|
| `supercoder/run-0229` | the first gate, 13 subagents | **$2.8879** |
| `supercoder/run-0230` | the correction pass | $0.3202 |
| `superdirector/run-0164`, `run-0165` | two plannings | $0.5879 |
| `front/run-0570`…`0576` | seven Front runs | $1.4109 |
| `front/run-0577` | the session-limit failure | $0.0779 |

The first gate is 55 % of it, and 13 concurrent subagents over 13 files is
where that went. The correction pass cost $0.32 — about a ninth — because it
was scoped to two files and recovered its text from a diff.

## Deus Ex Machina

- Wrote v6 and v7 and fired both publish runs as the Developer: the standing
  request is the developer's instruction by definition.
- **Read the `main` diff and found the over-cut.** This is the one
  intervention that mattered, and it is a handoff candidate: *did the
  gate-output audit for agent Front — handoff candidate.* Front cannot
  currently tell a correct check-4 strip from a destructive one, because it
  reads a report of the run rather than the diff the run produced. The
  smallest fix is not more supervision but a line in the standing request
  making the run print its own deletions for review — which is half of what
  v7's "say what you moved and where" now asks for.
- Posted the three nudges (callback misrouting, two close-out resolves).
  Each named a gap and asked the agent to close it; none of them did the
  work from outside.
- Nothing in `main/` or `publish/` was written or corrected by the Omni
  Agent.

## Addendum — the push permission is withdrawn (v8)

After the phase closed, the developer withdrew the agent-driven push:
**`publish/` is committed and never pushed, on every project, with no
exception.** The developer reviews that commit and pushes it by hand.

`#front › routine-publish` message **5482**, **v8**. What it changes against
v7:

- the opening states the withdrawal explicitly, so a run that remembers v6
  or v7 cannot act on the older permission;
- a new paragraph, *"Never push `publish/`"* — commit locally and stop
  there; `main/` is the opposite and unchanged (commit **and** push, because
  that is where a later run reads the knowledge from); and if a `publish/`
  clone is found ahead of or behind its origin, leave it and say so, because
  reconciling it is the developer's call and not a step of this gate;
- the projects table's origin column is relabelled `publish/` **origin** and
  both rows now read **no**, with a sentence saying the column records where
  the developer's own push will go and is not permission for the run to push
  there;
- the report line no longer asks where `publish/` was pushed; it asks the
  run to say plainly that `publish/` was committed and **not** pushed and is
  waiting for review.

v8 also says that `studyrealworld`'s `publish/` already holds one
agent-pushed commit from the v6/v7 runs — *"that is history, not a
precedent, and it stays where it is. Do not push again, and do not try to
undo it."* Nothing was reverted on the GitHub origin; `main` there is still
`5718242`, and it is the same content the gate reviewed.

`README_PROJECT.md` in the workspace, which the v6 run had rewritten to say
`publish/` is "Committed and pushed by an agent", now says it is committed
by an agent and **never pushed** by one, and records that one push happened
under v6/v7 before the permission was withdrawn.

**What this costs the phase's evidence.** Completion condition 4 asked that
reviewed knowledge reach the specified publication origin, and it did —
`5718242` is on GitHub and was verified there. That evidence stands as a
record of what happened; it is simply no longer the standing arrangement.
From v8 onward the routine stops one step earlier, at a reviewed local
commit, and the last step is the developer's.

**Deus Ex Machina.** Wrote v8 and corrected `README_PROJECT.md` as the
Developer. `README_PROJECT.md` is an ignored, workspace-local file stating a
developer-owned rule, so no in-system run was bought for a one-line
correction — but it is worth noting that the *previous* value of that line
was written by an autolab run under v6, so the file now mixes agent-written
description with a hand-corrected rule.
