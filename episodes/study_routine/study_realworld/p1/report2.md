# Step 2 — the routine, and the first discovery run

The `study-realworld` routine has a standing request, and its first fire ran
discovery end to end: three sources with three different acquisition
methods, each verified by a document actually retrieved, three substantive
reports, both index levels, one commit pushed to `main`, and both topics
resolved by Front without a human in the loop after the fire.

## The standing request

`#front › routine-study-realworld`, message **5345**, posted as the
Developer — v1, 5 979 bytes (well under this realm's ~10 000-character
silent-truncation cap; read back afterwards to confirm it landed intact).
No `front-` prefix, so Front never sweeps it; it is read only when a run
topic points at it.

Its shape, and why each part is there:

- **Two modes, chosen by the run's own topic.** Plain words — "discover
  sources for X", "investigate X" — are enough. It deliberately does not
  invent a mode token, because a scheduled fire cannot carry one.
- **The default is investigation, with one exception**: if
  `main/sources/INDEX.md` lists no source whose retrieval has been verified,
  do discovery. That single sentence is what let the first fire pick
  discovery with no payload at all, and what will make the next one pick
  investigation.
- **Discovery** — roughly three sources with *differing* acquisition
  methods, not three feeds of the same shape; a source is one acquisition
  unit with a stable id independent of its URL; **at least one document body
  actually retrieved per source**, with failure recorded as a candidate
  rather than hidden; source file + index row + one substantive report built
  from what was retrieved. "A report that only describes the source and
  cites no document body is not a report."
- **Investigation** — read both index levels first, use the recorded recipe
  rather than rediscovering it, fix a stale recipe and say so, keep the
  per-document metadata, compare document ids and versions against previous
  reports, link a follow-up to what it follows, and **record an empty check
  honestly**: "Never manufacture a full report for a check that found
  nothing."
- **Both modes** — `main/` is publish-ready; automation only where repeated
  work or an observed failure justifies it; push `main`, never touch
  `publish/`; **one task with parallel subagents**, not one task per source,
  because autolab's listener sweep is single-threaded and a task per source
  would guarantee serial execution and one Front round trip each.

The last point is lifted from the `publish` routine's v4/v5 experience
rather than rediscovered: it is the one piece of this request that exists
because another routine already paid for the lesson.

## The fire

`devenv/routine/trigger.sh study-realworld`, by hand, at
**2026-09-08T16:34:26Z** → `#front › front-routine-study-realworld-2026-09-08T16:34Z`,
message 5346. No mode, no topic, no region, no budget in the fire line — the
trigger script has no way to carry one, which is exactly the condition the
standing default was written for.

## What happened, in order

| time (UTC) | what |
|---|---|
| 16:34:26 | fire posted; Front acks |
| 16:35:07 | Front opens `pj-studyrealworld › workplan-discover-sources` and states the mode choice and its reason |
| 16:36:00 | autolab plans: one task, three subagents, `start.flag` created, `S4-1` + sub-work `S4-2`, `work-s4-1 › workrun-task1-s4-1` opened |
| 16:36:27 | Front posts into the workrun topic — that post is what starts the task |
| 16:40:02 | the single commit `d28335f` lands in `main` and is pushed |
| 16:41:27 | Front reads the transcript, checks publish hygiene, tells autolab it looks good |
| 16:42:12 | `workrun-task1-s4-1` ✔; workplan topic ✔; run topic ✔ |

Seven minutes fifty-one seconds from fire to both topics resolved.

**Front chose the mode, out loud.** Its first post says
*"`main/sources/INDEX.md` lists no source with verified retrieval yet, so per
the routine spec this run is discovery, not investigation"* — and autolab
independently re-checked the same file before planning
(*"Checked `main/sources/INDEX.md` first — it's genuinely empty, so this is
discovery, not a re-run of existing work"*). The default was exercised
rather than assumed.

**autolab started without a second round trip.** Its plan reply says so
explicitly: *"Since the request already gave standing approval for a plan
matching this shape and nothing here departs from it, I started the mission
(`start.flag` created) rather than waiting for another round trip."* The
standing request's "Approving means acting" line, relayed by Front, is what
bought that.

**The subagents really were concurrent.** The streamed transcript in the
workrun topic shows the three `Agent` launches in one batched message and
then interleaved `WebFetch`/`Bash` calls against federalreserve.gov,
pewresearch.org and reliefweb.int — not three sequential blocks. The parent
then wrote both index levels and made the one commit.

## What landed

`main` commit **`d28335f`** — *"Add first three verified sources: FOMC
minutes, Pew Research, ReliefWeb"* — 10 files, 376 insertions, pushed;
working tree level with `origin/main`.

| source id | publisher | acquisition method | status |
|---|---|---|---|
| `fed-fomc-minutes` | Federal Reserve (FOMC) | dated URL pattern + PDF download, discovered from the meeting calendar page | verified |
| `pew-research-reports` | Pew Research Center | dated report-page URL, found via the publications/topic index or a site-restricted search, HTML body fetched | verified |
| `reliefweb-updates` | ReliefWeb (UN OCHA) | RSS feed parse, `<item>` diffing on `guid`/`link` | verified |

Three genuinely different retrieval shapes, which is what the request asked
for and the part that would have been easiest to fake with three RSS feeds.

**The retrievals are real.** The FOMC subagent fetched
`fomcminutes20260729.pdf` — a 424 KB binary, read directly rather than
through a fetch summariser, on the run's own account ("a real PDF was
downloaded … let me read it directly to verify actual content") — and the
report that came out of it carries the 9–3 vote to hold at 3½–3¾ percent,
the three named dissenters, the PCE figures, the AI-buildout
financial-stability discussion and the deferred six-meetings-a-year
question. That is a document that was read, not a page that was described.
The Pew report carries the survey's field dates and sample composition; the
ReliefWeb report carries beneficiary counts and the named facilities.

**The honest-failure path was exercised without being needed.** No source
failed, so no candidate row exists yet — but `reliefweb-updates` records
that `api.reliefweb.int/v2/reports` answers **HTTP 403** without a
pre-approved `appname` and that `v1` is **HTTP 410**, and files that as a
limitation of the API path rather than of the source, since RSS already
works. It also records that a browser-like User-Agent is needed. That is the
kind of thing a second run would otherwise pay for again.

**Publish hygiene held.** A grep of the whole `main` tree for hostnames,
`.home.arpa`, absolute local paths, `localhost`, ports, `autodev/`, Gitea,
and the internal vocabulary (`workplan`, `workrun`, `supercoder`,
`subagent`) returns nothing. The reports are original-wording summaries; the
source files say in as many words not to reproduce publisher text verbatim.

## Cost and timing

| run | role | duration | cost |
|---|---|---|---|
| `front/run-0561` | front | 48.4 s | $0.2102 |
| `superdirector/run-0162` | planning | 50.4 s | $0.1433 |
| `front/run-0562` | front | 74.1 s | $0.2548 |
| `supercoder/run-0226` | the task itself | 88.2 s | **$1.5013** |
| `front/run-0563` | front | 50.1 s | $0.2384 |
| `supercoder/run-0227` | close-out report | 28.5 s | $0.1478 |
| `front/run-0564` | front | 31.6 s | $0.2144 |

**$2.71 for the whole run**, all `anthropic/claude-sonnet-5`, every one
`"outcome": "done"`. The task itself is 55 % of it and the three subagents
are inside that single number; Front's four runs are 34 %, which is what
supervision costs when a mission is four round trips long.

## Findings worth carrying forward

1. **`sources/INDEX.md` has no acquisition-method column.** The whole point
   of the catalog is "which of these can I get at, and how" — and today a
   reader has to open three files to learn that one is a PDF pattern, one a
   page fetch and one an RSS feed. The status column also reads `VERIFIED`
   in caps for all three rows, so the candidate/verified distinction is
   documented but not yet visibly exercised. Worth a column, not worth a
   correction run of its own.
2. **`devlog/` appeared again**, at
   `<workspace>/devlog/s4-1-…/task-1/{work.md,report.md}` —
   `record_task_in_devlog` writes a folder the study pattern never declares.
   This is the known friction from `scheduled_routine` p5, reproducing
   unchanged on a fourth project. It is outside `main/`, so it publishes
   nothing and breaks nothing; it is recorded here because it is now four
   for four and belongs in whatever episode finally fixes the pattern's
   folder list.
3. **Natural-language mode selection is not yet proven.** Both modes have
   now been *specified*, and the discovery default has been *exercised*; an
   explicit "investigate X" request that overrides the default has not been.
   Step 3's scheduled fire deliberately tests the other default rather than
   the override, so the override stays untested at the end of p1 unless a
   spare request is spent on it.

## Deus Ex Machina

- Wrote the standing request and fired the trigger as the Developer. Both
  are Developer work by design — the routine's request is the developer's
  standing instruction, and the routine is fired by the dispatcher or by
  hand.
- Nothing inside `main/` was written or corrected by the Omni Agent. The
  three source files, three reports and both index levels are the autolab
  run's own output, unedited.
