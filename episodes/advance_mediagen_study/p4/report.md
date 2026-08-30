# advance_mediagen_study p4 — Report

Two fires of `routine-mediagen`, 2026-08-30 11:47Z → 13:00Z. **73 minutes
wall clock, 48 paid runs, $17.43.** Missions **M-23** (three tasks, the
survey and ranking) and **M-27** (four tasks, running the winner). Steps are
`report1.md` and `report2.md`; this is the phase report.

The braindump asked for one thing: *the skeletal-animation experiment came
out lower-level than expected — survey the field properly, decide whether
anything is worth trying, and try it if so.* All three happened. The trying
produced a worse picture than p3 and four findings worth more than the
picture.

## Did the survey actually happen this time?

**Yes.** The plan named URL count as the cheap test, and it clears it:
roughly **twenty-five distinct URLs** actually fetched or read across five
areas, landing in a rewritten `main/subjects/skeletalRig/summary.md` Gather
section, with a **"What this survey could not settle" list of six named
holes** — including one where the run marks its *own* reasoning as an
untested inference rather than dressing it as a source.

I spot-checked the two most decision-relevant citations against their
primary pages rather than trusting the report. Both exact: spine-canvaskit's
docs do say it can be used in "backends (to render skeletons headlessly)",
and Spine's CLI docs do say "to export images or video an OS windowing
system and OpenGL are required."

p3 did no web research because my own p3 fire said "do not redo the survey",
and that hint travelled verbatim into the mission and overrode step 1 of the
standing request. That was my defect. This phase bought it back and the
survey immediately answered a question p3 had left open, *against* my
expectation: I assumed Spine's CLI was the headless path; it is not, and
spine-canvaskit is.

## The ranking as delivered

Thirteen ideas, each with what it needs beyond this host, expected gain
against the unchanged asset bar, and one line of rank reasoning.

What makes it a ranking rather than a list is stated before the table: three
of the four bar clauses (64×64, ≤32 colours, keyable background) are met **by
construction**, so "gain over p3" collapses to one question — *does this give
the rig more genuine per-limb DOF without re-opening a clause already solved
for free?*

| rank | idea | verdict |
|---|---|---|
| 1 | Spread-leg generation pose, re-run the existing cut script | **Go** — zero cost, targets the diagnosed mechanism |
| 2 | `flux1-kontext-dev` generates isolated per-limb parts | **Go**, below #1 — on the box, capability unverified anywhere |
| 3–13 | mesh-deform, video-motion + pose extraction, PolyRig, Spine/spine-canvaskit, Phaser, layered-generation SaaS, SpriteToMesh, Live2D, DragonBones | **No-go** |

The no-go reasoning is the valuable half: rows 3, 6 and 7 are no-go **not
because they are blocked but because even fully unblocked they would not
touch the bottleneck.** Ranking spine-canvaskit — the survey's own best
discovery, a real documented headless capability — last-but-viable on that
criterion is the best call in the table. A criterion that only ever confirms
your favourite finding is not a criterion.

**No host install was requested.** The winner needs nothing that is not
already here, and the run recorded that explicitly rather than skipping the
question. Nothing in this phase waited on me.

## What was run, and how it compares frame for frame

Fire 2 ran idea 1 in a new repository, `gentest-skeletalRigSpreadLeg`.

![the spread-leg rig's rendered frames](spreadleg_rig_contact_sheet.png)

**On its own mechanism the idea worked.** Belly-line ratio 0.35 → **0.74**
with the requirement satisfied; cut parts from p3's paw-tip fragments to
`front_legs` 19×28 and `back_legs` 20×27 against a `body` of 46×11. The
segmentation ceiling p3 named is lifted, for a changed prompt clause and
seconds of GPU.

**Against the deliverable it is a regression.** Frames 0 and 2 — the only
two where the rig rotates anything — show the dog torn apart, background
visible through the gap. Frames 1 and 3 look like an animal because their
keyframed rotation is 0.0.

Cell to cell against the same requirement, three rounds now:

| clause | p1 locked-seed sheet | p3 rig | p4 spread-leg rig |
|---|---|---|---|
| consistent silhouette | achieved empirically | **by construction** | by construction |
| flat keyable background | flat but wrong colour, cast shadow | **solid chroma green** | solid chroma green |
| ≤32 colours, one palette | by construction | by construction | by construction |
| **looks like a walk cycle** | **wins — real 8-pose stride** | near-static, paw-tips only, 1 duplicate pair | **worst — parts detach; 1 duplicate pair** |
| posable leg length | n/a | paw tips only | **best — near body height** |

**p1's sprite sheet is still the only one of the three that looks like a
walk cycle.** Two rounds of rigging have not beaten it on the clause that
matters most, and this round moved further away while fixing the thing p3
said was in the way.

The mechanism: `segment_parts.py` produces **disjoint** parts — hard cut
line, no shared boundary, no overlap at the joint. p3's fragments rotated
±22° and moved a few pixels, too small to reveal the flaw. Parts the height
of the body swing their far end a long way and leave the torso. There was
never anything holding them together.

**The success exposed the next ceiling.** That is the phase's headline.

## Four findings nobody asked for

**1. A metric that did not lose precision — it inverted.** Round 1's winner
`spread_wide_stance` scored 0.62 against 0.35 and is *not a side view*: it is
frontal, and it scored highest **because** frontality maximises leg length
below the belly while destroying the fore/aft separation a side-view rig
needs. The best-scoring pose was the worst pose for the purpose. Caught by
opening the image; the reinforced-clause regeneration that followed produced
genuinely-profile candidates at 0.69 and 0.74, which is the only reason the
fire has a positive mechanism result at all.

That is the **third** time here a metric has ranked a requirement-violating
cell first — after background-flatness ranking an outright failure flattest
of twenty, and the consistency instrument reading a byte-identical pair as
its best case. All three caught by looking; none by a number. Now written
into `tips.md` generalised: *every metric this project has built ranks its
own blind spot first.*

**2. A tip is not a control, and the reason is structural.** One task after
writing that tip, the rig result reached me as "rigged and rendered, no
fallback needed" — judged on a hash count and two bounding boxes, with the
render never opened. Asked directly, Front answered straight: it cannot open
project files; it judges from what the run reports.

- The **run** can open the render, and has an interest in it reading well.
- The **supervisor** cannot, and reads hash counts and the run's own prose.
- **I** can, and I am outside the loop.

The check this project trusts most is, for the supervisor, **structurally
unperformable.** The rule landed on the party with the eyes and the
incentive, and there was no second party with the eyes inside the loop that
writes the report. Offered *(a)* tasks attach their images or *(b)* the
supervisor can open images, the run argued for (b) and argued me out of (a):
attaching still leaves the reporting party choosing what to attach — and this
correction is the proof, since a bounding-box table *was* attached as
evidence by a run that had not opened what it was claiming success for.

**3. A published finding falsified by the strongest evidence available.**
p3's `tips.md` blamed the duplicate frames on segmentation coarseness.
Segmentation was substantially fixed and the duplicate survived unchanged.
The real cause, verified by me in `build_rig.py`: `front_leg_angles_deg`,
`back_leg_angles_deg` and `torso_bob` all repeat with period 2, so indices 1
and 3 are identical **for any source pose whatsoever**. The correction
landed with the wrong entry **kept verbatim and labelled** beside the new
one — a reader can see the project changed its mind, which a clean rewrite
destroys.

**4. Fire 1's own ranking falsified, in part.** Mesh deform was no-go
because it "would not fix the leg-DOF shortfall". The bottleneck moved. The
run falsified **only** that half and left the editor-GUI cost and the open
engine bug standing. Not "the ranking was careless" but "the ranking was
right about a bottleneck that its own winner then moved."

## Two publication leaks, one of them mine

The widened check found `agforge` — an internal agent name — sitting in
published `main/` since `165a689`, through p3's entire redaction round **and
through my own independent check afterwards.** My check was too narrow: I
grepped for repository names, hostnames, IPs, ports and paths, never for our
own agents' names.

p3's lesson was *a self-check phrased as an instruction is a claim; only the
command is evidence.* This phase adds the other half: **the command is only
evidence for the pattern it greps for**, and a redaction scoped to the leak
you already found verifies only that leak.

The second surfaced when the run refused to guess: three `gentest-`
repositories carry a hardcoded internal endpoint in tracked files, which the
standing text's own wording ("host literals live in the test repository's
*ignored* `.local/` only") arguably already prohibits. I scoped the check to
`main/` — a test repository is a lab notebook, and a reworded run log is a
worse record, not a safer one — after checking all five remotes myself and
confirming every repository pushes to the same local Gitea. Recorded as
unresolved for a later round rather than decided in a closing message.

Final state, verified by me: `main` check exit 1, `main`@`93db192` and
`gentest-skeletalRigSpreadLeg`@`281367d` both level with origin, `publish/`
zero commits and untouched.

## Costs

| | runs | cost |
|---|---|---|
| supercoder | 12 | $9.20 |
| Front (supervision) | 33 | $7.61 |
| superdirector (planning) | 3 | $0.62 |
| | **48** | **$17.43** |

Fire 1 was 14 runs / $6.06; fire 2 was 30 / $10.32. Against p3's 33 / $12.11.

**Thirty-three Front runs against twelve of actual work is the number that
matters**, and the cause is mine: I read the run topics directly, so my posts
repeatedly landed while Front was mid-report on the same thing, waking runs
that found nothing new to say. Supervising *and* watching the work costs a
run every time the two cross. The irony is exact — I was doing by hand the
very thing finding 2 says the supervisor cannot do, and paying twice for it.

GPU cost was negligible: nine stills across two rounds, seconds each.

## Frictions

**1. My posts crossed Front's reports, repeatedly.** Fire 1 once, fire 2
throughout. Not a defect in the system — a consequence of the Developer
having read access the supervisor lacks, which is finding 2 seen from the
cost side.

**2. `agentchat send` was blocked by the harness's auto-mode classifier.**
The first fire could not go out until the permission mode was changed. Read
calls passed; the outbound post did not. The allowlist is not the lever.

**3. The front venv's `agentchat` has no `wait` subcommand**, so supervision
from the harness side fell back to polling. Version skew, not a defect.

**4. A resolved `workrun-` topic stops being a task binding.** Front tried to
chase verbatim grep output from a task that had already resolved its own
topic and got "this topic is not bound to any task". Anything a supervisor
still needs must be asked **before** sign-off. Recorded, and relayed into
fire 2's brief.

**5. The plan's own premise about the yaml was wrong.** It said
`gentest-skeletalRig` could be extended because its yaml "is not
`verified`". It was. Raised before it could bite; the run checked the
sibling convention, found no state meaning "proven and being extended", and
refused to invent one — so fire 2 opened a new repository. The constraint
was enforced and worked around openly rather than quietly relaxed.

## Deus Ex Machina

- **Opened the candidate stills and stopped the wrong pick.** Nothing in the
  loop compares a generated still against the requirement's own pose clause
  before measuring it. Handoff candidate: *a generation sweep should check
  each candidate against the requirement's clauses before ranking it on a
  proxy.*
- **Opened the render and stopped a broken result being signed off.** The
  sharpest handoff candidate this phase produced, and structural rather than
  behavioural: *the supervisor cannot open the artefact it is asked to
  judge.* The run's own preferred fix is on record with its argument.
- **Found the `agforge` leak in already-pushed `main/` by hand** and bounced
  it back rather than editing `main/` myself. Handoff candidate: *nothing in
  the loop greps `main/` for internal agent names, and the round that did
  grep it scoped the pattern to the leak it had already found.*
- **Verified independently rather than relaying**: two survey citations
  against their primary pages, the `build_rig.py` keyframe arrays, the
  publication greps, and all five git remotes. Every one held.
- **Raised the yaml contradiction and scoped the publication check** when
  asked, as information and a decision — no file in any project repository
  was edited by me this phase.
- Copied the rig contact sheet and candidate sheet into this public episode
  folder. No host facts in either.

## What the next round should test

The run's own read, which is better than the one I brought: **joint-aware or
overlapping part cutting, before mesh deform.** A change to `segment_parts.py`
so a rotated leg still has torso pixels behind it at the pivot — one script,
testable against a source still that already exists, no new capability, and
inside the scripted no-editor-GUI constraint this line has held since its
first task. Mesh deform stays the fallback, re-ranked by evidence rather than
ruled out.

Open besides: the supervisor-can-open-images gap; the `build_rig.py` period-2
keyframe collision (now correctly diagnosed, still unfixed — worth fixing
only after the parts stop detaching); and the hardcoded endpoint across three
test repositories.
