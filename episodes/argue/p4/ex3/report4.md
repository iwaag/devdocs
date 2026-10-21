# argue p4 ex3 — Step 4: deployed, exercised in a bounded discussion, checked from another LAN PC

## State before deployment (Nautobot/nctl)

- I refreshed the observation (`nctl observe agstudio`). `nctl drift
  --host agstudio` reported converged, both before and after.
- No service placement changed:
  - WFE runs in `agdevworld-web`;
  - aghangout is served from its release directory (`aghangout_web`,
    whose HTTP check of `/hangout/` still returns 200).

  Nautobot needed no update.
- Active work: none. No attempt was open and no job was queued or
  claimed (576 done, 2 dropped).

## Deployed revisions

| Component | Revision | How |
| --- | --- | --- |
| WFE | agdevworld `cc156f3` (08:32Z; store **v9 → v10**), then `2eb1976` | `scripts/wfe-release.sh deploy`: the full check 497/497 in the clean release each time, then a restart that reported healthy |
| aghangout | `00f599f`, then `96d1b99` (from `c66bd02`) | `scripts/deploy.sh <commit>`: unit tests in the clean export; `current` switched |
| ponderer's guide | pj-ponderer `acb25ae` (rebased onto the archive writer's commits) | Gitea main |
| wfrun guide | agautolab `c08f117` | GitHub. The executor reads it from this checkout at every attempt |
| pj-agdev pins | `0b625d0`, then `8e4f45d` (agdevworld `2eb1976`, agautolab `c08f117`) | pushed |
| docs | WFE `docs/discussions.md` *Workflows in a discussion*; README_DEV *Workflows beside the speech*; aghangout README | in those commits and this one |

**The second deploy, found on the deployed data.** A derived record's
`at` is when it was recorded. The first backfill therefore dated every
historical record "1分前", and that would also have placed silent-turn
rows by the backfill time. Since `2eb1976`/`96d1b99`, each version
carries its own `at`: the adjustment, the run's creation, or the moment
WFE read the bytes. aghangout dates and orders records by it.

The 36 backfilled rows were enriched in place by
`step4/enrich-at.ts`:

- only `at` was added;
- each version was first verified against its run's records (36/36).

Records 1–3 had already been exported to the archive. Their transcript
lines lack `at`, as said in README_DEV.

## Backup and recovery

- **Backup**: `.local/agdev/.local/backups/argue-p4-ex3-predeploy-20261021-172752/`.
  It holds:
  - the conversation store at v9 (249 messages);
  - `exec.sqlite`, `conversations.json`, `registry.json`;
  - the archive writer's clone;
  - a `pj-ponderer` bundle;
  - aghangout's head and its served release.
- **Rollback**: written in the ignored `pj-agdev/.local/argue-p4-ex3/notes.md`.
  Code and data must go back together, because `5d6f511` refuses store
  v10. Restoring the v9 copy loses every workflow record and #250 onward
  of the check.
- aghangout rolls back with `deploy.sh activate c66bd02…`.
- The archive's workflow records and files are additive.

## Backfill

`wfe discussion workflow repair --all` ran once after the migration.

- **36 records** came from proven sources only:
  - conductor revisions, with their ponder activity;
  - delegated children of discussion runs;
  - one hand-off link (`p16-s3-room` → `aghangout/change/run-002`, with
    its message).
- Every record is `evidence: backfill`, with its source in the note.
- All 38 versions were captured.
- Messages, invitations, activities, jobs and every discussion's
  `updated` were identical before and after (`step4/work-*-backfill.txt`).
- Old speech was untouched, and nothing was placed by time.

## The bounded check (`argue-p4-ex3-check`, category `check`)

The brief (#250, by "Omni Agent (for Developer)", given once) asked forge
for four things:

- to point to its `provide` workflow and the node that decides what to
  make;
- to make a draft copy `provide-review-check` with an added review step,
  and save it;
- to run one small, safe piece of work through its own project (no
  images, nothing published) and present the result.

It asked ponderer to conduct briefly. It did **not** name the attach tool.

### What happened (08:40–08:47Z), and what WFE recorded

| # | Turn | Operation | Record |
| --- | --- | --- | --- |
| 37 | ponderer `conduct/run-010`, activity 48 | `ponder --next` → r2 | **edited**, progression; r1 → r2; nodes `ask-forge`, `ponder-after-forge` |
| 38 | forge `discuss/run-010`, attempt `amvhv4apj`, invitation 74 | `wfe workflow new provide-review-check` | **created**, a draft (the empty template) |
| 39 | the same attempt | `wfe workflow save` | **edited** draft → draft: 5 nodes added (the copy of `provide` with `review` "Review before delivery"), plus intent, documentation, execution and repositories |
| 40 | the same attempt | `wfe run adjust` of its own run | **edited**, forge `discuss/run-010` r1 → r2 (adds `small-work`) |
| 41 | the same attempt | `wfe delegate request` | **executed** `forge/freework/run-004` at r1. The child wrote one line to `scratch.txt` |

- **Operations before speech.** The attempt that did all four said
  nothing. forge's result, #251, came from a **later attempt**
  (`amvhv8jzl`) once the child had finished. The room therefore shows
  them as a 「発言のない操作」 row with the run, attempt and invitation.
  This is the operation with no spoken message the plan asks for, and
  it occurred naturally.
- **Records it noticed.** forge's #251 says "The workflow and its saved
  revision are attached to this turn". It read the save's own output
  ("Recorded in discussion …").
- **The conclusion was not announced.** ponderer concluded with a ponder
  decision (activity 50: answered, "review step never run end to end").
  It wrote a Japanese, source-linked overview **without being asked**,
  as in ex2. It did not announce the conclusion, so its own invitation
  is `unanswered`. This is the same pattern as ex2, recorded as
  annotation 5 and not corrected.
- **Assistance after the brief.** None to the agents. Annotation 4
  records my later edit to the draft (below).

### Gaps the trial showed

- **No reference record for `provide`.** forge pointed to
  `devdocs/workflows/provide.yaml` and its `make` node **in prose only**,
  without `wfe discussion workflow attach`. By design WFE does not infer
  a reference from a name in text. The guide fact was not enough here.
  This is a missing reference capture.
- **No message link.** forge's #251 linked none of its records
  (`contribute --workflow` was not used). The cards are therefore shown
  by turn, which is the honest placement.
- **No copy provenance.** "Copy of `provide`" exists only in forge's
  text. The records show creation from the template, then the save.
- **Record 40 shows as subject.** It is forge's revision of its own
  `discuss` run, which is not the conductor's progression. Whether a
  participant's own conversational graph should also fold away is an
  open design question.

### A later edit does not move the turn

- **The edit.** After the turn I saved a node `later-check` into
  `provide-review-check` from outside any attempt. No record was written,
  correctly, since it was not a discussion turn. Annotation 4 says so.
- **Pinned.** Record 39 still reads 5 nodes (`cleanup, confirm, make,
  present, review`).
- **Current.** `…/workflows/39/current` reads 6 nodes with
  `same: false`. The room's 現在 shows `later-check`, labelled 「この議論で記録された版ではありません」.

### Closing

- **The agpc message.** The check discussion received one message from
  agpc (#252, below). ponderer's next turn (`run-011`, activity 51) read
  it as needing no reply.
- **Closed.** I then closed the discussion.
- **Archive.** It is current at pj-ponderer `ef15ccd`, with 5 `workflow`
  records and 7 version files. Across every archived discussion, 54
  version files each match their sha256 name.

## LAN evidence (agpc)

`step4/lan-check.mjs` ran on agpc through Ansible as
`~/p15-check/lan-check-ex3.mjs`, with Chrome. The final run passed
**24/24** (`step4/agpc-lan-check.txt`).

| Area | Result |
| --- | --- |
| Contract | progression and subject records; created, edited and executed; the edit's added nodes identified; before and after both readable |
| Cards | every subject record has a card; 2 silent-turn rows; progression folded |
| Edit | the panel opens at このターン時点・変更後. WFE's graph renders over the LAN with ＋ 追加 marks. The change list names the additions. 変更前 lacks them. "No message recorded" is said. 現在 is labelled as not the recorded version. The editor link is the secondary action |
| Execution | the run's state at the recorded point (ランの記録 1 番目) |
| Reload | every record names the same version |
| History | `argue-p4-trial` shows backfilled progression and subject records |
| Ordinary reading | a discussion without workflows reads as before; the list opens; no page errors |
| Narrow (390 px) | no sideways scroll; the panel is a full-screen sheet |
| No dispatch | reading, opening graphs and switching versions posted nothing and invited nobody |

**Posting** was confirmed from agpc separately (`step4/post-debug.mjs`):
the room's composer posted #252 and WFE answered 200. The check run that
first tried to post (run 3, kept as `agpc-lan-check-run3.txt`) had
pressed Enter while the room was still opening, and nothing was posted.
I could not reproduce that on the fixture: an Enter before the room's
first read posts there. It is recorded as unexplained, not as fixed.

## Review links (from any LAN machine)

- **The check, with every relation and a silent turn**:
  <http://agstudio.local:8093/hangout/?discussion=argue-p4-ex3-check>.
  Open 「編集 Provide an asset (review check)」 for before/after and
  現在, and 「実行 Free work」 for the run.
- **A historical discussion, backfilled**:
  <http://agstudio.local:8093/hangout/?discussion=argue-p4-trial>
  (jevlab's revision and its executed experiment; the conductor's
  progression).
- **The full editor (secondary)**:
  - the draft: <http://agstudio.local:8093/wfe/#/ws/forge/wf/provide-review-check.yaml>;
  - the executed run: <http://agstudio.local:8093/wfe/#/ws/forge/run/freework/run-004>.
- **The records as JSON**:
  <http://agstudio.local:8093/wfe/api/discussions/argue-p4-ex3-check/workflows>.
- **Screenshots**:
  - agpc: [cards and overview](shots/agpc-lan-cards.png),
    [edit, after](shots/agpc-lan-edit-after.png),
    [edit, before](shots/agpc-lan-edit-before.png),
    [現在](shots/agpc-lan-edit-current.png),
    [execution](shots/agpc-lan-execution.png),
    [history](shots/agpc-lan-history.png),
    [phone](shots/agpc-lan-narrow-panel.png);
  - deployed, from agstudio: [argue-p4-trial](shots/deployed-argue-p4-trial.png).
