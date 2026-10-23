# Workflow editor fix4 — report

Plan: [plan.md](plan.md). Decision 1 was taken as (a): guide only, no
aghangout change.

## What the three symptoms were

| Symptom (discussion `20261023-230532-z4mv12`) | Cause | Change |
| --- | --- | --- |
| Front said the same thing twice (#331, #335) | #334 answered front run-010's q1 without naming it. The routing contract sends such words to the conductor. ponderer asked front again, and front's new run-011 delegated the same re-run. run-010 stayed waiting in the closed room. | ponderer `conduct.md`: the person's untargeted words that answer a participant run's open question are that run's answer (`wfe run answer … --from … --source-message …`). |
| ponderer repeated Front (#336 after #335) | Once its ask was resolved, the conductor turn spoke so that the person's request to it would not stay `unanswered`. | `conduct.md`: when what it awaited has already reached the person, the turn records its ponder and ends without speaking. An `unanswered` request to the conductor is then correct. |
| gimmickimg did every run inside one talk; review pages stayed live | freework is one talk node. The catalog had no template for the work itself. "As far as possible" (fix3) had eight counter-examples. Pages 8300–8302 had no cleanup node. | The catalog has `work` r1 and `present` r2 (cleanup is its pair). freework's talk says that the work is added as nodes of its own: WFE's skeleton and the existing freework of 12 projects. The wfrun guide adds run-005…008 as a trial reference. |

## Releases

| Component | Revision |
| --- | --- |
| WFE (agdevworld) | `36588fd` (previous `61e547a`; skeleton text only, store unchanged) |
| agautolab (wfrun guide) | `881ca9f` (previous `1b65813`) |
| pj-ponderer | `bd88bd4`, `565c497` (conduct.md), `3320d2d` (freework) |
| freework.yaml | gimmickimg `ff5b55a`, aghangout `9315210`, agops `a8a8eb1`, archsage `50490db`, cagent `a923416`, characters `b0675fa`, forge `fca6201`, front `1a1f98b`, glossary `0ac5159`, lowerpoly `8424a95`, jevlab `90be945`, ponderer `3320d2d` |
| Node catalog (user data) | `work` r1 (new), `present` r2 |
| agpc worker | WFE `36588fd`, executor `881ca9f` (nctl batch, playbook ok=22 changed=4; the unit is active) |

Each project's `converse.yaml` keeps its talk: a converse run is a conversation
turn, and its work is delegated.

## Cleanup

- `front/discuss/run-010`: q1 is answered by #334 (from iwaag, with the
  source message), taken up, and `review` is completed with the reason.
  No attempt ran in between (seq 28–30). The closed room no longer lists it
  as an obligation.
- gimmickimg freework run-005, run-007 and run-008 p1 (ports 8300, 8301
  and 8302) are stopped and verified gone.

## Verification

**Routing** (check room `fix4-route-check`, closed):

- #340: the brief asked ponderer to have Front ask the person "red or
  blue?" as a run question. front/discuss/run-012 asked it in #341.
- #342 「青でお願いします。」 was posted **without `re`**, as #334 was. It
  reached the conductor's open run-028 (request 180).
- run-028 acknowledged it as "Passed #342 to front/discuss/run-012 q1".
  front run-012 records `question.answer` by ponderer-agent, from the
  person. Front took it up and replied #343. **There is one Front run**.
- After the ask resolved, run-028 recorded "front's reply #343 already
  reached the person; no further speech". **ponderer did not speak again.**

**freework nodes** (`gimmickimg/freework/run-009`, a small check request
that did not mention nodes):

- The graph was `talk` → `sheet` (the work) → `present` → `confirm` (q1) →
  `cleanup`.
- The page served two images (both 200). After the answer, `cleanup`
  stopped p1 and verified it. Port 8300 no longer answers.

**WFE**: `npm run check` passed 541/541. The release is healthy, and the
service and `bin/wfe` name `36588fd`.

## Remaining

- A person's answer that names no question still goes to the conductor.
  The fix is the conductor's reading (decision 1a), so it is probabilistic.
  It held in one trial. If it fails again, option (b), aghangout's composer
  defaulting `re` to the one open question, is the next step.
- gimmickimg/freework/run-007, run-008 and run-009 end "Done · Every step
  completed" while `_final-report` is still pending. This existed before
  fix4 and was not addressed here.
- The WFE warning for a run whose presentation was registered from a talk
  (decision 3) is deferred. One trial passed without it.

Machine-specific revisions, recovery commands and the agpc batch are in the
ignored `pj-agdev/.local/fix4/notes.md`.
