# scheduled_routine p2 — Phase report

Basis: the `rtnotes` main-only project and two manual-fired runs of
`routine-rtnotes` on 2026-08-24 (02:52Z and 03:02Z, the latter recovered at
03:10Z after an external listener restart); the 2-hourly plist is loaded
and takes over from here. Details per step in report1–4.

## What was built

- `init_project.py <slug> --main-only`: Plane project + one Gitea repo
  (`main`), `devlog/` a plain local folder, no `direction/`. Runtime
  `init_project(slug)` infers the layout from disk, so servings never
  re-clone what the flag left out; turning a local devlog into a clone is
  refused. Full stays the default and the four existing projects are
  untouched (report1).
- `record_task_in_devlog` pushes only when `devlog/.git` exists; otherwise
  `recorded … in devlog locally (not a repository)`. `serve_bmining`
  declines politely without `direction/`. Guides read both folders
  "if exists" (report2).
- `rtnotes` (tiny check script + `NOTES.md`), `#pj-rtnotes` with autolab
  subscribed (report3); standing text in `#front › routine-rtnotes`,
  `com.agdev.routine-rtnotes.plist.in` at `StartInterval` 7200 (report4).

## Front runs per routine run; stalls and loops

| run | Front runs | wall clock | notes |
|---|---|---|---|
| 1 | 10 | 02:52 → 02:59 (7 min work + 6 min detour) | asked permission once; then the ✔-rename loop, broken by the Developer |
| (02:52 declined-fire) | 1 | 18 s | "already done today" — v3's finding, no work run |
| 2 | 5 (+2 dead servings) | 03:10 → 03:13 (2.5 min) | clean; recovered from the stale-topic trap by itself |

One loop, run 1: Front's callback read the resolved `workrun-` topic under
its old name (plain `agentchat read` does not follow Zulip's `✔ ` rename;
only `--since`/`wait` do), saw emptiness, and posted into what became a new
unbound topic. autolab answered "not bound" each time, its **entrance made
the same misread** ("never started" about a Done task), and one re-plan
happened before autolab itself refused to iterate. Timeline in report4.
The devenv.md warning about rename-blind lookups names precisely this;
p9 lost a callback to it and now p2 lost a supervisor's read.

One stall, external: both listeners restarted at 03:04:19 (another
process's interruption). Front died between ack and reply, and after
restart the topic's last poster was Front's own ack, so nothing was
awaiting — silent until a human posted. **An unanswered ack should count
as awaiting after a restart**; that is a pyagag/listener candidate.

## What the standing text had to grow

Three lines, each after exactly one occurrence (report4): approving means
acting (start + done-check, ask nothing); every trigger is a run, note
headings carry date **and time**; resolved topics are read under their
`✔ ` name. Nothing went into Front's guide or code; run 2 shows all three
applied, including self-recovery from the stale-topic trap it fell into
one run earlier.

## Was the local-only devlog read by the second plan?

**Yes, twice.** Run 1's re-plan (gen 2) wrote in `plan.md`: "A matching
work log already exists under
`devlog/r3-1-run-rtnotes-checks-fix-trivial-breakage-append-a/task-1/`
(`work.md` and `report.md`)" — and used it to refuse repeating the work.
Run 2's plan and note both read the previous `NOTES.md` entry and varied
from it (its note raises the per-day vs per-run heading question itself).
A plain folder is enough for the record to be written and read back; git
added nothing a routine project needed.

## Open findings, not fixed here

- `main` of a routine project is **ahead 2, never pushed**: the supercoder
  commits (Front's start post is the approval), the close-out pushes only
  `devlog/`, which is local. Decide whether routine `main` commits should
  be pushed by the handler like devlog used to be, or stay local until a
  human pulls; today the Gitea repo silently ages.
- autolab's entrance answers about a resolved task by reading the plain
  topic name — same rename blindness as Front's read (its board survey and
  close-out follow the rename; the entrance's ad-hoc read does not).
- The trigger posts even when nobody will serve it (03:04:19 restart);
  harmless because the next fire re-reads history, but the orphaned-ack
  fix above is the real cure.

## What a third routine / ENT episode should be (not built)

- **ENT: rename-following reads.** One episode making every read path —
  Front's callbacks, autolab's entrance, plain `agentchat read` — follow
  the `✔ ` rename, plus re-serving topics whose last post is an unanswered
  ack after a restart. Both p2 detours vanish; no standing-text line can
  do it (v2's ✔ warning helped the *next* run, not the blind read itself).
- **Third routine**: p1's phase report already names it — a periodic
  "where do my plans stand" survey through autolab's own channel. p2 adds
  a reason: the entrance's stale answer above would have been caught by a
  routine that compares the survey against Plane state every few hours.
  A routine *appending to the same `workplan-` topic* is deliberately not
  it: new-topic-per-run proved cleaner in both phases.

## State at phase end

- agautolab `3f14fc8` + `39a2be9` (pushed; pj-agdev pin `8be2691`),
  listener restarted on the new code before run 1.
- `rtnotes`: Plane R3 project, Gitea `autodev/rtnotes` (2 commits behind
  the local clone), `#pj-rtnotes`, devlog 2 task records, `NOTES.md`
  2 dated notes.
- Routine: standing text v3; run topic holds 2 runs + interventions;
  plist loaded (`launchctl list` shows it), next fire ≤ 2 h after 03:15Z.
