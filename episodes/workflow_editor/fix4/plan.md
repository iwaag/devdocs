# Workflow editor fix4 — an answer reaches the run that asked; the conductor may stay silent; freework work as nodes

## Evidence

Discussion `20261023-230532-z4mv12` (「切り抜きエッジの改善」, 2026-10-23),
conducted by ponderer with archsage and front as members; the work was
delegated to `gimmickimg/freework`.

### A. Front answered twice (#331, #335: "delegated to freework, no result yet")

- `front/discuss/run-010` presented gimmickimg's result (#332) and asked the
  person with `wfe run ask` on its `review` node (q1, posted as #333: a / b /
  c).
- The person answered in #334 without pressing 返信する. The post carried no
  `re`, so by the routing contract (conversations.md *Routing a person's
  message*, rule 4) it went to the conductor, not to run-010.
- `ponderer/conduct/run-026` started, read #334 as "option (a)", and asked
  front again (`ask-front-wan`, request 176). That started
  `front/discuss/run-011`, which delegated the same Wan re-run a second time
  and wrote #335.
- `front/discuss/run-010` is still `Waiting · Answer from iwaag` (q1 open)
  in a closed room; `wfe conversation state` lists it under *Obligations*.
- At #334 the conductor could see front's open question ("Waiting for the
  person" in `show`; `awaiting` in `state`). The wfrun guide says a
  person's later words for work under way are carried to that run
  (`wfe run answer … --from <person> --source-message`), but ponderer's
  `conduct.md` does not state it, and the conductor chose a new ask.

### B. ponderer repeated Front (#335 → #336)

- run-026's ask was resolved by front's #335. WFE continued the run with
  one instruction; ponderer added `ponder-after-wan` and replied #336,
  which restates #335 and adds nothing ("Front delegated… I reply and end",
  instruction ack at 14:31:32).
- The person's request 175 was addressed to ponderer, so a silent conductor
  turn leaves it `unanswered`. run-025 already left 171 and 174 unanswered
  without harm, but no guide says that this is a normal outcome when a
  participant's answer has already reached the person.

### C. gimmickimg does all its work inside one talk

- `pj-gimmickimg/devdocs/workflows/freework.yaml` has one node, `talk`.
  Every run 001–008 completed with `talk` alone; run-007 was `node.start
  talk` → `presentation.register` → `node.complete` (3 min 13 s of SAM2,
  matting and a presentation, invisible in the graph).
- This is freework's shape, not gimmickimg's: forge's freework runs are
  mostly one node too. Only `characters/freework/run-008` added
  `present`/`confirm`/`cleanup`.
- The node catalog has `present`, `cleanup`, `prestudy`, `boast` — nothing
  that names the work itself, so a run has no template for "do the thing"
  and no prompt to design one.
- The guide's "as far as possible, not an absolute rule" (fix3) has eight
  consecutive counter-examples in this project: it is not working as
  guidance here.
- Consequence: presentations `8300` (run-005), `8301` (run-007) and `8302`
  (run-008) are all still LIVE (`wfe present status`). With no cleanup node
  nobody stops them — the same failure fix3 recorded for run-003.

## Decisions to take (Developer)

1. A person's untargeted message that answers a participant run's open
   question is that run's answer. Decide where this lives:
   - (a) guide only: the conductor carries it (`wfe run answer`), or
   - (b) also in aghangout: when the room is facilitated, the person has
     exactly one open question and the composer has no `re`, the composer
     proposes replying to it (UI only; the routing contract is unchanged).
2. A conductor turn whose awaited answer already reached the person may end
   with its record alone. An `unanswered` request to the conductor is then
   a normal outcome, not a defect.
3. freework's work is done in nodes the run adds, not inside `talk`:
   - the catalog gets a `work` (do) template in the Developer's words
     ("the actual work I asked for on its own node(s), not inside the
     talk"), and `present` says that `cleanup` is its pair;
   - freework's `talk` is consultation and design only (its description
     says so);
   - whether to add a WFE warning (fix3 style: warn, `--yes` accepts) when
     a run completes with a presentation registered from a talk and no
     other work node. Recommended: not yet; see whether 3's two changes are
     enough first.

## Changes

### 1. ponderer `agent/conduct.md` (pj-ponderer)

- Fact: the person's untargeted words may answer a participant run's open
  question. `wfe conversation state` (`awaiting`) shows it; pass them on
  with `wfe run answer <run> --workspace <project> --from <person>
  --source-message <conv>:<id>` instead of asking that participant again.
  Trial reference: run-026 / front run-010 → run-011, #334.
- Fact: when the outcome of your ask already reached the person (a reply or
  result in the room), the turn ends with its ponder record; no repetition.
  The person's request to you stays `unanswered`, which is correct. Trial
  reference: #336 after #335.

### 2. aghangout composer (optional, decision 1b)

- `app/src/room.ts`: in a facilitated room, when `asking` has exactly one
  message whose receiver is the person and the composer has no `replying`,
  set `replying` to it and show the existing `#n への返信` row (removable
  with ✕). No change to WFE routing.
- Browser checks: the row appears for one open question, not for none or
  two; removing it sends to the conductor as before.

### 3. freework and the catalog

- Catalog (`wfe nodes create`, the Developer's wording):
  - `work` (do): the work the request asks for, on its own node(s), added
    by `wfe run adjust` before it is done; a talk holds the consultation,
    not the work;
  - `present` r2: a presentation is followed by `cleanup` in the same run.
- `pj-gimmickimg/devdocs/workflows/freework.yaml`: `talk` description →
  "understand the request, hold the consultation it needs, and add the work
  as nodes of its own; the work itself does not happen here". Intent
  updated the same way. The same wording goes to the skeleton that `wfe
  create` uses for new projects' freework, so other projects inherit it.
- wfrun guide *The steps the person expects to see*: add gimmickimg
  run-005…008 and the three live presentations as the trial reference
  beside the fix3 one.

### 4. Clean-up of this episode's leftovers

- `front/discuss/run-010`: record #334 as q1's answer (`wfe run answer
  --source-message 20261023-230532-z4mv12:334`), complete `review`
  with the outcome "answered by #334; carried out by run-011", so the
  obligation disappears from the closed room's state.
- Stop presentations 8300, 8301, 8302 (`wfe present stop` for
  gimmickimg freework run-005/007/008) and verify each is gone.

### 5. Verification

- Replay the routing case in a `check` room with ponderer and front: front
  asks, the person answers without `re`; expected: the answer reaches
  front's run (by the conductor or by the composer, per decision 1), no
  second front run, and the conductor turn posts nothing new.
- One gimmickimg freework request through Front: expected graph has a
  `work` node, `present` and `cleanup`; `wfe present status` shows the
  page stopped at the end.
- Existing WFE and aghangout checks pass; if 2 is taken, aghangout is
  deployed with `scripts/deploy.sh` and the check runs from agpc as well.

### 6. Report

- `report.md` here: what each symptom was, which change addressed it, the
  check room's id, and what remains (the warning of decision 3, if
  deferred). Update `devdocs/README_DEV.md` with a short fix4 section.

## Execution policy

`localrule.md`: English files, Japanese chat. No approval gates are added;
the catalog and guide changes are facts and expectations, not shackles.
This document is the plan; writing it executes nothing.
