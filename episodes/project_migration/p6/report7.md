# project_migration p6 — Step 7 report: end-to-end trials and review preparation

Status: **done, with a reduced scope set by the Developer** (2026-10-17 ~21:35 JST). The snake model turned
out too distorted, so construction from it stopped. The Developer judged that the migration of playprey itself
was sufficiently shown, and T3 stopped after its plan was agreed. This report supersedes
[mid-report7.md](mid-report7.md) for everything after the pause. Raw evidence is in the ignored
`pj-agdev/.local/project-migration-p6/step7/`.

## What stalled the work (asked for by the Developer)

The Developer asked for a record of what made this step stall, so that meaningless rules and guardrails can be
removed. Each item names the rule, what it cost, and whether it protects anything.

| # | Rule or guardrail | Where | What it cost | Does it protect anything? | Disposition |
| --- | --- | --- | --- | --- | --- |
| S1 | "You do not commit, push or post" in the shared agent guide | agautolab `agent/guides/wfrun/guide.md` | run-003's `present` failed: `delivery.py` needs a commit and the agent believed it must not make one. This was the open defect at the pause (one failed node, one extra attempt, the hold over the pause). The engine had published the agent's own commits since p9, so the rule was false | No. Pushing and posting stay with the host; committing never needed forbidding | **Removed** (agautolab `500be25`). The next attempt committed by itself (`8189f134`) |
| S2 | Shared areas carry no dot entries at all | WFE `server/shares.ts` `listFiles`, `docs/shares.md` | run-003's first delivery `o1` was published without `game/.gitignore` while its own manifest listed it, so it could never verify on a consumer. One review round (about $1) | Partly. It hides the share marker, `.partial-*`/`.staging-*` folders and `.DS_Store`. Hiding *every* dot entry is broader than needed | playprey adapted (`delivery.py` drops dot entries). **Candidate to narrow** to those named entries |
| S3 | `final: true` puts a generated `_final-report` **after** the `confirms: result` talk, and integration refuses any change after acceptance | playprey workflows (Step 6) + WFE integration | run-003's integration stopped `needs-agreement` on `report.md`, so the result had to be re-read and accepted again. The same trap stopped forge's first three runs in p3 (p3/report4) | The binding (nothing changes after acceptance) is meaningful. Allowing a definition that is certain to violate it is not | playprey fixed (`bdd3eb7a`: `final: false`, `outcome.md` before the confirmation). **Engine candidate**: refuse or reorder this combination at validation, so a third project does not hit it |
| S4 | Claude Code auto-mode classifier (the Omni Agent's harness, not the system) | this session | Refused, among other things: resuming run-002 as the Developer's proxy, `wfe run integrate` (twice), a read-only `git status`, a history rewrite of a run branch and a plain `grep`. The executor sat idle from 19:24 to 20:42 JST (about 80 min) waiting for manual approval, and again about 20 min before 21:17. The refused integration had in fact already been done by the executor | Not for this work: every refused action was within the Developer's standing delegation, and some were read-only | Not a project rule; it cannot be deleted from the repositories. It can be avoided by running these sessions outside auto mode with an allow-list, or by approving per call |
| S5 | The >50 MiB guard covers only unpublished commits | WFE `e19fa04` | run-002's branch already carried a 63 MB `work/Cavity.npz` (committed and pushed by a host checkpoint before the guard existed). Accepting it would have fast-forwarded the blob into main. Needed a history rewrite of the run branch (Developer-approved) | The guard is meaningful (run-001's 2 GB). The gap only concerned branches older than the guard, and run-002 was the only one | No new guard added (it would protect nothing from now on). Recorded here |
| S6 | Every edit of a cagent `plan.md` needs a fresh agreement | pj-cagent change workflow | cagent run-007 asked 22 questions, most of them re-agreements of small plan edits (mid-report7) | Weakly. The first agreement is the one that authorizes effects | **Candidate to remove or relax** (for example: re-agreement only when the set of effects changes) |

Not stalls, for completeness:

- `delivery.py` refusing a dirty tree is right, because a delivery names its code revision. It stalled only because of S1.
- The pause itself was the Developer's request.
- My own verification of `o2` ran Python directly against the share and left `__pycache__` in it. The
  agent noticed and removed it. Checks against a share now run with `python3 -B` and read-only commands.

## Trials

| Trial | Run(s) | Result | Evidence |
| --- | --- | --- | --- |
| T1 existing case | `onboard/run-001` (rejected), successor `onboard/run-003` | **Accepted and integrated** (playprey main `0a53a487`). Clean rebuild of `eagle_beakfit` from the shared inputs, byte-identical asset `632c3b971e13ef7d`, all eagle case checks pass, materials stay small, delivery `o2` (1533 files, 55 MiB) verified on the share | run-003 records; `step7/t4-*.txt` |
| T2 different anatomy | `onboard/run-002` (snake → profile `anaconda`) | **Stopped by the Developer**: the model is too distorted. Up to then it exposed and fixed a real hidden assumption (a tongue was required everywhere; a declared tongue-absent capability, 5 code sites, no regression in 9 profiles + eagle P2 suite). Its closed pose was not a real closure (24° of a 49° gape) and showed stretched strips at 24°; Omni did not accept it (q3). Output `o1` stays unaccepted | run-002 records, `knowledge/cases/anaconda.md` on its branch |
| T3 coexisting approaches | `compare/run-001` (eagle_beakfit vs eagle_beak2) | **Stopped after the plan was agreed**, at the Developer's choice. The plan keeps the two candidates as separate profiles and outputs, and adds a visible `prestudy-b` go/no-go node, so a negative result for B could end `build-b` early. That is a change of plan made visible as a revision, not a hidden branch. No candidate was built | `devdocs/runs/compare/run-001/plan.md` (run branch) |
| T4 cross-host delivery | run-003 `o2` | **Done.** On the Linux VM, through its own mount, `delivery.py verify` passes (1533 files, commit `8189f134`), `model_profiles.py check eagle_beakfit` passes in a copy of `game/`, and no producer checkout exists there. `delivery.json` fetched over the LAN URL matches byte for byte. The three eagle input GLBs read identically from both hosts | `step7/t4-inputs-agautolab1.txt`, `t4-verify-agautolab1.txt`, `t4-http-agautolab1.txt` |
| T5 resume and publication | fixture | **Done** (mid-report7): a killed publish leaves nothing visible, the retry publishes once, and a changed input is recorded as `changed` on resume. The leftover partial folder is fixed (WFE `a1b217f`) | `step7/t5-kill.*` |
| T6 knowledge reuse | run-003, run-002, compare/run-001 | **Shown in plans and records, not in a finished later case.** run-003 reused run-001's findings by name. run-002 cited the eagle's lipless/toothless lessons from `knowledge/onboarding.md`. compare/run-001's plan cites `knowledge/cases/eagle_beakfit.md` (written by run-003) and P1 Ex2's closure measurement as the *method* for B, "not copied as values, since B is a different source" | plans of those runs |

### Defects found and fixed after the pause

| # | Defect | Fix |
| --- | --- | --- |
| 4 | `delivery.py build` needs a commit, but the guide forbade the agent to commit (S1) | agautolab `500be25`, pj-agdev `fc70763` |
| 6 | Published deliveries listed a dot entry the share does not carry (S2) | playprey `8189f134` (by the agent, through run-003) |
| 7 | Generated final report after the result confirmation blocked integration (S3) | playprey `bdd3eb7a` |
| 8 | Pre-guard 63 MB blob in run-002's pushed branch (S5) | run branch rewritten by Omni with the Developer's approval: tree unchanged (`96fa9100`), head `16ac9e6b` → `9ac1de14`. The run's earlier checkpoint records still name the old commit ids |

## Human review preparation (plan items 7.3–7.5)

- **LAN review page:** the WFE run view of `onboard/run-003` on the editor service (`:8093`, reachable from the
  LAN). It shows the explanation, the same-camera stills (baseline vs rebuild), the output identity
  (`o2`, sha256 `7827b5a7…` manifest) and its download link.
- **Native launch:** one action on the review Mac,
  `pj-agdev/.local/project-migration-p6/step7/review/launch-eagle_beakfit.command` (ignored). It:
  1. resolves `shared://agshare/runs/playprey/onboard/run-003/o2/` with `wfe share resolve`;
  2. copies it locally, because the Godot import writes into `game/`;
  3. verifies every byte, imports twice (the first pass extracts textures) and launches Godot.

  Tested end to end: verified 1533 files, Godot ran with no errors.
- **Human judgement recorded:**
  - The Developer delegated all answers and acceptance for this step to the Omni Agent. Every answer is
    recorded under that name, with the delegation cited.
  - The Developer's own judgement in this step was on T2: the snake model is too distorted to continue. It is
    recorded in run-002's stop reason.
- **Delivery contract:** demonstrated as an isolated consumer check on another PC (T4) and the local launch.
  No protoprey integration is claimed. protoprey uses gl_compatibility, so embedding needs a separate
  integration (knowledge/delivery.md).

## Costs and time (executor attempts)

| Run | Attempts | Cost | Attempt time |
| --- | ---: | ---: | ---: |
| playprey onboard/run-001 (T1, rejected) | 7 | $10.24 | 102 min |
| playprey onboard/run-002 (T2 snake, stopped) | 7 | $24.49 | 100 min |
| playprey onboard/run-003 (T1 successor, accepted) | 5 | $5.42 | 26 min |
| playprey compare/run-001 (T3, stopped after plan) | 2 | $0.61 | 2 min |
| **playprey total** | 21 | **$40.76** | 230 min |
| cagent change/run-007 + develop/run-004 | 27 | not reported (Codex gpt-6-luna) | 59 min |

## Omni interventions after the pause

- Settled defect 4 by correcting the shared guide (S1), not by telling the agent a workaround.
- Verified both deliveries independently: on the share, on the Linux VM and over the LAN. This found S2.
- Re-accepted run-003 after reading its post-acceptance report, correcting two statements in the decision
  note: the checks were the Omni Agent's, not the Developer's; and the baseline leaves the lower beak ahead
  of the hook.
- Fixed S3 in playprey's workflows and rewrote run-002's branch (S5).
- Built the review launcher.

## Leftovers that need a decision (not removed)

| Item | Size | Note |
| --- | --- | --- |
| `shared://agshare/runs/playprey/onboard/run-001/o1/` | 2.06 GiB | rejected T1 delivery |
| `shared://agshare/runs/playprey/onboard/run-003/o1/` | 55 MiB | superseded, does not verify (S2) |
| `shared://agshare/runs/playprey/onboard/run-002/o1/` | 37 MiB | unaccepted snake delivery |
| Gitea branch `wfe/playprey/onboard/run-001` | ~2 GB blobs | record of a rejected run |
| `refs/omni/pre-rewrite/run-002` in run-002's local copy | 63 MB blob | pre-rewrite backup, local only |
| runs `onboard/run-002`, `compare/run-001` | — | held; cancel them if they will never resume |

## Deployed state

| Component | Revision |
| --- | --- |
| WFE (live release) | agdevworld `e19fa04` |
| agautolab wfrun guide (live from checkout) | `500be25` (pj-agdev `fc70763`) |
| playprey main | `bdd3eb7a` (includes run-003 at `0a53a487`) |
| pj-clusterintent (live) | `fff9a4c` |
