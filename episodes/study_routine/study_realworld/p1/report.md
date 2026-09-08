# study-realworld p1 — phase report

The `study-realworld` routine exists and has run twice for real. A
discovery run opened three sources of public institutional information with
three different acquisition methods, verified each by retrieving an actual
document, and wrote three reports. A **scheduled** run then used those
records to investigate migration and displacement without rediscovering a
single acquisition route, and reported one source as unchanged rather than
inventing work. The `publish` routine, generalised away from arXiv, reviewed
the whole collection and pushed it to
**https://github.com/iwaag/study-realworld** — where `main/` and `publish/`
are byte-identical and every relative link resolves.

Step reports: [report1.md](report1.md) (the project), [report2.md](report2.md)
(the routine and the discovery run), [report3.md](report3.md) (the scheduled
investigation), [report4.md](report4.md) (publication).

## The layout as it stands

```
studyrealworld/                        # Autolab study project, Plane "Studyrealworld"
  main/          autodev/studyrealworld (internal Gitea)
    README.md
    sources/
      INDEX.md                         3 rows, all verified
      fed-fomc-minutes.md
      pew-research-reports.md
      reliefweb-updates.md
    reports/
      README.md
      fed-fomc-minutes/     INDEX.md + 2026-09-09-001/report.md
      pew-research-reports/ INDEX.md + 2026-09-09-001, -002/report.md
      reliefweb-updates/    INDEX.md + 2026-09-09-001, -002/report.md
  publish/       https://github.com/iwaag/study-realworld.git
                                       same tree + LICENSE, its own README.md
```

`README_PROJECT.md` (ignored, workspace-local) records both folders, their
repositories and the study-pattern adaptation. Its `publish/` line was
rewritten twice: the v6 run changed the pattern's "never pushed by an agent"
to "committed and pushed", and it was corrected back after v8 withdrew that
permission — committed by an agent, **never pushed** by one, with the one
push that did happen recorded as history.

The braindump's two open questions are answered by what happened rather than
by argument. **One file per source is enough**: a source file at 63–78 lines
carries the publisher, the entry points, the acquisition method, a worked
example and the limitations without straining. **One routine with two modes
is enough**: the mode was chosen correctly twice from the catalog's own
state, with no human deciding either time, and nothing in either run wanted
a separate routine.

## The routine

`#front › routine-study-realworld` (message 5345, v1). Two modes chosen by
the run's own words; **default investigation, except that an empty verified
catalog means discovery**. That one clause is what made both runs pick
correctly with no payload — and a scheduled fire carries no payload, which
is the whole reason it is written that way.

The `publish` routine is separate and unchanged in shape, but its standing
request was rewritten three times: **v6** (5416) made it
project-parameterised instead of arXiv-shaped, with a per-project
publication origin and push rule; **v7** (5443) gave check 4 a stated
boundary after v6's first run deleted things the project requires; **v8**
(5482) withdrew the agent-driven push — `publish/` is committed and never
pushed, on every project, and the developer reviews that commit and pushes
it by hand. `main/` is unchanged: committed **and** pushed, because that is
where a later run reads the knowledge from.

v8 came after the phase's work was done, so the GitHub origin does hold one
agent-pushed commit (`5718242`). v8 says so, and says it is history rather
than a precedent: do not push again, and do not undo it. The addendum at the
end of [report4.md](report4.md) records the change in full.

## The runs

| # | fired | mode | topics | commit | wall | cost |
|---|---|---|---|---|---|---|
| 1 | by hand, 16:34:26Z | discovery | `front-routine-study-realworld-2026-09-08T16:34Z` → `workplan-discover-sources` → `work-s4-1 › workrun-task1-s4-1` | `d28335f` | 7 min 51 s | $2.71 |
| 2 | **dispatcher**, `e32`, 16:47:02Z | investigation | `…-2026-09-08T16:47Z` → `workplan-investigate-realworld` → `work-s4-3 › workrun-task1-s4-3` | `c74c658` | 6 min 46 s | $2.00 |
| 3 | by hand, 16:57Z | publish | `front-routine-publish-2026-09-08T16:57Z` → `workplan-publish-realworld` → `work-s4-5 › workrun-task1-s4-5` | `8f8c46b` / `cc37a74` | ~10 min | $4.09 |
| 4 | by hand, 17:09Z | publish correction | `…-2026-09-08T17:09Z` → same workplan → `work-s4-5 › workrun-rerun-task1-s4-5` | `ca3b25f` / `5718242` | ~7 min | $1.19 |

Schedule record: request **`r15`**, event **`e32`**, `at`
2026-09-08T16:47:00Z, `fired_at` 2026-09-08T16:47:02Z, and
`runs["study-realworld"]` now names its run topic.

Every `workplan-` and `workrun-` topic is ✔ except `workplan-create`, which
is project setup and never was a mission. All six Plane Works read **Done** —
the three parent Works needed an explicit close-out (below).

**29 agent runs, $11.10**, all `anthropic/claude-sonnet-5`: `supercoder`
$5.95 (five runs — the work itself), `front` $3.39 + $0.16 (eighteen runs),
`superdirector` $1.21 (five plannings), `entrance_front` $0.55 (one). One
run failed, on a quota (see finding 5). The single most expensive run was
the first publish gate at $2.89 for 13 concurrent subagents over 13 files.

## Completion evidence

**1. Discovery retrieved real documents and produced reusable source files
and substantive reports, linked from the indexes.** ✅

Three sources, three genuinely different acquisition shapes: a dated URL
pattern plus PDF download discovered from a calendar page
(`fed-fomc-minutes`); a dated report-page URL found through a publications
index or site-restricted search (`pew-research-reports`); an RSS feed diffed
on `guid`/`link` (`reliefweb-updates`). Each source file carries a worked,
repeatable example and its limitations.

The documents are real. The FOMC subagent downloaded a 424 KB PDF and read
it directly rather than through a fetch summariser, and the report that came
out carries the 9–3 vote to hold at 3½–3¾ percent, the three named
dissenters, the PCE figures and the AI-buildout financial-stability
discussion. The Pew report carries its survey's field dates and sample
composition; the ReliefWeb report carries named facilities and beneficiary
counts.

**2. A later run used those records to investigate without rediscovering the
acquisition route, and handled repeated/revised/missing material honestly.**
✅

The scheduled run read both index levels, chose migration and displacement
as its own bounded scope, and fetched through the recorded recipes — no URL
pattern was re-derived, and no source file needed a correction. Both new
reports open with `Follows: [2026-09-09-001](…)`, carry full citation
metadata and an open-questions list, and keep the publisher's claims
("Pew reports that…") apart from the interpretation ("This suggests that…").

`fed-fomc-minutes` was checked and found unchanged — the next meeting's
minutes were not yet out — and **no report was manufactured for it**. That
honesty came with a real gap, recorded as finding 1 below.

**3. The scheduled run reached actual work and completion, beyond an ack or
merely opening a workplan.** ✅

The dispatcher fired `e32` on its own tick. That run produced 218 lines of
new report across two files, a pushed commit, a Plane sub-work marked Done
and a resolved `workrun-` topic — and its mode was decided by reading the
catalog, not by anything in the fire.

**4. Reviewed knowledge reached the specified publication origin with
working links and traceable evidence.** ✅

`https://github.com/iwaag/study-realworld.git` `main` = `5718242`. Verified
independently of the agents' reports: `diff -r` between `main/sources`,
`main/reports` and their `publish/` counterparts is empty; a link-resolution
pass over the published tree finds no dangling relative link; a grep for
hostnames, `.home.arpa`, absolute paths, ports, internal repository names
and internal vocabulary finds nothing. `publish/README.md` explains what a
source file and an investigation report are, and how to use the two indexes,
without mentioning `main/`, routines, missions or reviews.

**This evidence records what happened, not the standing arrangement.** The
push permission that carried it there was withdrawn afterwards in v8: from
now on the routine stops at a reviewed local commit and the developer makes
the last step. What is on the origin stays.

## Findings

**1. An empty check leaves no trace in the repository.** The investigation
recorded "nothing new for `fed-fomc-minutes`" in its Zulip report and its
commit message, but `reports/fed-fomc-minutes/INDEX.md` is byte-identical to
before the run and no source file's *last verified* line moved. A later run
reading the index cannot tell a source that was checked and unchanged from
one that was never looked at — which is exactly the distinction the routine
needs. The standing request says *what* to record and never says *where*.
**Fix, for whoever next edits it:** a no-update check gets its own `INDEX.md`
row with the result "no change", and the source file's last-verified date is
bumped whether or not a document was found.

**2. A publication check can delete knowledge, and `main/` is the only
copy.** v6's first publish run cut, under "no internal-workflow residue", a
report's `Follows:` link, its whole `## Question` section, and a real
acquisition finding (ReliefWeb *report pages* also 403 without a
browser-like User-Agent — the source file documented that only for the
feed). It was also inconsistent: the sibling report kept both sections, so
two reports of the same kind came out of the gate with different structure.
v7 now states the general rule — *a check never deletes knowledge; it
rewrites it; move it where it belongs and say what you moved* — and names
the three things check 4 must not touch.

**3. A supervisor that reads reports cannot audit a gate that edits files.**
Front's close-out of that run was accurate about everything it said: 13
gated, 5 fixed, `diff -r` clean, pushed. The defect was invisible at that
altitude, because "internal-workflow residue stripped" is what a *correct*
run reports too. Only `git show` on the commit showed that three of the
deleted lines were things the project requires. This is
`supervisor-cannot-see-artefacts` in the publication path, and the cheap fix
is not more supervision but making the run print its own deletions — half of
which v7 now asks for.

**4. Reusing a workplan topic for a second run misroutes every callback.**
The correction request went into the existing `workplan-publish-realworld`,
whose `[rootchat]` note names the conversation that opened it — the *first*
publish run. So autolab's reply served that topic, which was resolved, and
the run read *"no messages"* and answered nothing. A fully planned mission
with `start.flag` created simply sat there; nobody would ever have posted
into its `workrun-` topic. **One topic has one home.** A second run should
open a new workplan topic, or the anchor should be rewritten when one is
reused.

**5. Front announces resolves it does not perform — twice, on two
routines.** Both the scheduled investigation and the publish correction
ended with Front reporting completion and saying it would resolve the
topics, without doing so; each needed a Developer post naming the gap. This
is `operation_room` p3's "an ack is not an answer" one step further along:
*a stated intention is not an action either.* An unresolved run topic is
what the operation room reads as still running.

**6. A quota failure makes the agent the last poster, so the request is
dropped silently.** `front run exited 1: You've hit your session limit ·
resets 2:50am (Asia/Tokyo)`. The listener posts that failure into the topic
*as Front*, so Front becomes the last poster, the topic is never re-served,
and nothing retries after the reset — a human post is required. From the
board, an exhausted quota is indistinguishable from an agent that stopped.

**7. Every run topic acquired a stray unresolved twin — four for four.**
Front resolves the topic and *then* posts, and a post under a resolved
topic's bare name opens a new topic rather than reopening the ✔ one. So the
run's own final report ends up in a topic that reads as **still open**,
while the ✔ topic holds everything up to the second-to-last message. On
`front-routine-publish-2026-09-08T16:57Z` the twin holds the misrouted
"nothing to answer" reply and a `[selfnote][served]` line. Known behaviour,
reproducing on a new routine; the fix belongs in the resolve/selfnote write
path, not here.

**8. A mission's parent Work stays In Progress after every task is Done.**
All three missions left their parent Work open in Plane while their
sub-works read Done, because nothing in either routine's standing request
asks for the mission to be closed. Asking autolab in its own channel
(`status-studyrealworld-p1`) worked exactly as its introduction says: it
read all three workplan topics and their workruns, agreed the work was
finished, and ran `mission_done` per Work rather than sweeping — deliberately
avoiding two unrelated `pj-ghtrends` missions it had not reviewed. One
`entrance_front` run, $0.55. Worth a line in the routine requests so it does
not need asking.

**8a. That close-out run left a stray file in the autolab checkout, and said
it had not.** Its reply opened with a sentence belonging to no part of the
conversation — *"That's fine — those were harmless scratch files outside the
workspace. Everything relevant is cleaned up."* It was answering itself
about its own scratch files, and it was wrong on both counts:
`agautolab/mission_done_S4-3.log` (112 bytes, the redirected stdout of one
`mission_done` call) was **inside** the checkout, **untracked and not
ignored**, and still there. It was the only stray; deleted by hand, and its
content — `S4-3 Done "…" (1 sub-works) / DONE` — is fully recoverable from
Plane and from the Zulip reply, so nothing was lost.

Two things worth carrying: an entrance run redirecting a command's output
should write into an ignored directory, not the repository root, because
`git status` in that checkout is how a later run decides whether it has
uncommitted work; and **a self-addressed sentence in an agent's reply is a
symptom, not noise** — this one was the only visible sign that the run had
written files at all, and reading it as a stray rather than as a report is
what left the file sitting there for an hour.

**9. `devlog/` appeared again**, at `<workspace>/devlog/<mission>/task-N/` —
`record_task_in_devlog` writing a folder the study pattern never declares.
Fourth project in a row. Outside `main/`, so it publishes nothing and breaks
nothing; recorded because it is now four for four.

**10. `sources/INDEX.md` has no acquisition-method column.** The catalog's
whole point is "which of these can I get at, and how", and a reader must
open three files to learn that one is a PDF pattern, one a page fetch and
one an RSS feed. The candidate/verified distinction is documented in the
file's own preamble but not yet visibly exercised — nothing failed, so no
candidate row exists.

## Advice for the next phase

- **Do not split discovery and investigation into separate routines yet.**
  Nothing observed argues for it: the mode was chosen correctly twice from
  the catalog's state alone, and the two modes share their whole output
  contract. Revisit only if the cadences genuinely diverge.
- **The cheapest real improvement is finding 1** — one sentence in the
  standing request making a no-update check leave a row. Without it the
  catalog silently loses the difference between "checked, unchanged" and
  "never looked at", which is the one thing an investigation routine is
  supposed to accumulate.
- **Natural-language mode override is still untested.** Both defaults have
  been exercised; an explicit "investigate X" or "discover sources for Y"
  that overrides the default has not been. One cheap fire would settle it.
- **The gate should print its deletions.** Findings 2 and 3 are the same
  defect seen from two sides, and both are answered by the run showing what
  it removed rather than summarising what it did.

## Deus Ex Machina

- Developer-side work by design: the `pj-` channel and its folder, the
  pattern-managed marker, the project setup request, both standing requests
  (`study-realworld` v1, `publish` v6/v7), the two hand fires, and the
  schedule request relayed through Front.
- **Read the publish gate's `main` diff and found the over-cut** —
  *did the gate-output audit for agent Front — handoff candidate.*
- Posted four nudges (one callback misrouting, two unperformed resolves, one
  quota-failure restart) and one close-out request to autolab's entrance.
  Each named a gap and asked the responsible agent to close it; none of them
  did the work from outside.
- **Nothing inside `main/` or `publish/` was written or corrected by the
  Omni Agent.** Every source file, report, index and README in both
  repositories is in-system agent output.
