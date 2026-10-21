# argue p4 ex3 — workflows alongside the discussion: final report

The Developer asked to see the workflows a discussion referenced,
created or edited in the same screen as the conversation, tied to the
turn in which each happened. ex3 delivers that:

- WFE records each such association as evidence, at the exact version
  the turn had;
- aghangout shows it as a card at that turn, which opens WFE's own
  read-only graph beside the speech.

**Delivered and deployed:**

- WFE agdevworld `2eb1976` (store **v10**);
- aghangout `96d1b99`;
- ponderer's guide `acb25ae` and the wfrun guide (agautolab `c08f117`);
- the pj-agdev pins;
- 36 historical records backfilled from proven sources.

Everything was checked from agpc.

Step reports:

- [report1](report1.md): what was provable, the recording matrix, the
  contract, the renderer boundary, the screens;
- [report2](report2.md): records, journaling and repair, reads,
  comparison, archive, tests (497/497);
- [report3](report3.md): cards, the panel, the embedded graph, browser
  evidence (181/181 in Chromium and WebKit);
- [report4](report4.md): deployment, the bounded check, the later-edit
  check, LAN 24/24, recovery.

## Outcomes

| Completion criterion | Outcome |
| --- | --- |
| A referenced, created or edited workflow can be inspected in the same screen and its turn identified, without the editor | A card at its message, or at its turn (「このターンの操作」 / 「発言のない操作」 with run, attempt, invitation), opens the ワークフロー panel beside the conversation (a sheet on a phone). The panel has the graph (WFE's renderer), the node list and details, and 「#n へ移動」 back to the speech. The editor is a secondary link |
| The initial graph is the evidenced historical version; current definition and state are explicit alternatives | The panel opens on このターン時点: the captured bytes, sha256-checked, and a run's states replayed to the recorded `seq`. 現在 is a separate read, labelled 「この議論で記録された版ではありません」, with "same/changed". Verified live: a later edit left record 39 at 5 nodes while 現在 showed 6. A version not captured is said to be unavailable and never substituted |
| Creation, reference, editing and execution have distinct meanings backed by evidence; missing associations are visible limits | Each is recorded by its own operation (`workflow new/save`, `run adjust`/`ponder`, `request`, `delegate request`) or by an explicit `attach` (retrieval, not comprehension), with evidence `operation`/`claim`/`backfill`. A name in prose, a shell read or the time of a message is never taken as a reference. The trial's missing reference to `provide` is reported, not inferred |
| Node, connection and property changes can be inspected, removals included, not by colour alone | Comparison is by node id and edge. Properties are shown before → after; layout-only and generated steps are told apart. The graph marks ＋追加 / ～変更 / －削除 as text badges and dashed, dotted or struck patterns; the change list and node list say it in words |
| Subject workflows and conductor progression stay distinct; routine changes do not overwhelm ex2's reading path | Progression (the facilitator's own graph) folds into 「進行の更新 N件」 and is never a large card. The overview-first room is unchanged, and the cards come after the speech they belong to |
| Associations survive retries and archives with no duplicate operation and no change to speech; reads start nothing | Tokens make each record once (hooks and repair agree). Interrupted saves and requests are recovered from their own evidence or marked `unconfirmed`. Tests and live counters show that no message, invitation, job or `updated` moved for any record, backfill or read. The archive carries the records and the version files (54, all matching their sha256) |
| Tests pass; deployed and verified from another LAN PC; commits, pins, help, docs, review links, Omni notes | WFE 497/497. aghangout 25 unit tests, 181 browser checks in each engine and 177 without the renderer. agpc 24/24, posting confirmed. Docs: WFE `docs/discussions.md`, `wfe discussion help`, the aghangout README, README_DEV. Links in [report4](report4.md#review-links-from-any-lan-machine). Omni note below |

## The bounded check, in short

**What was recorded.** forge's first attempt did everything the brief
asked: it created the draft, saved the review step, revised its own run
and delegated a `freework` run that wrote one line. It said nothing in
that attempt. WFE recorded all four operations at that turn by their own
commands, and the room shows them as a row of operations with no speech.
forge's result came from a later attempt. ponderer's revision was
recorded as progression, and ponderer again wrote a Japanese overview
without being asked.

**What was missing.**

- forge pointed to `provide` only in prose. It never attached it, so no
  reference record exists for it.
- forge linked none of its records to its message.
- ponderer concluded without announcing it (as in ex2).

## Remaining limitations

- **Reference capture depends on the agent.** Consulting a workflow is
  recorded only by `attach`. In its first trial forge did not use it, and
  the one guide fact was not enough. Evidence-driven guidance needs more
  trials before more is said. Reads by shell are never recorded.
- **No message links in practice.** Agents have `contribute --workflow`
  and `workflow link`, but did not use them. Cards fall back to their
  turn, which is honest but coarser.
- **The participant's own run revision is a subject card.** forge's
  revision of its `discuss` run shows as a subject card. Only the
  conductor's graph is progression; whether a participant's own
  conversational graph should fold away too is open.
- **Editor edits are not recorded.** An edit made in the editor during a
  discussion has no attempt, and so no discussion context.
- **Earlier drafts are gone.** Definitions saved before this episode
  have no captured history. Backfill covers run revisions, delegated
  children and hand-off links only.
- **Three archive lines lack `at`.** The transcript lines of records 1–3
  were exported before versions carried `at`. The manifests are
  regenerated on their next export.
- **English inside the graph.** The graph's type labels (DO, TALK,
  STUDY), "Fit", and WFE's raw reasons for an unreplayable state are
  English.
- **Not covered by tests.** `wfe delegate request` was proven live, not
  by a unit test. A project actually being unregistered was not
  simulated; snapshot resolution covers it.
- **An unexplained posting failure.** One LAN run's post did not go out
  when Enter was pressed while the room was still opening. It was not
  reproduced; posting works ([report4](report4.md#lan-evidence-agpc)).
- **Not measured.** Whether the Developer finds the turn connection
  intuitive was not studied, and this report does not claim it.

## Omni handoff note

> Deus Ex Machina note: backfilled 36 workflow records from proven run
> histories, delegations and hand-off links into the discussion store for
> agent ponderer, and edited forge's check draft `provide-review-check`
> after its turn (annotation 4) — handoff candidate.

## Review links

- The check, with every relation and a silent turn:
  <http://agstudio.local:8093/hangout/?discussion=argue-p4-ex3-check>
- A historical discussion with backfilled records:
  <http://agstudio.local:8093/hangout/?discussion=argue-p4-trial>
- The full editor (secondary):
  <http://agstudio.local:8093/wfe/#/ws/forge/wf/provide-review-check.yaml>,
  <http://agstudio.local:8093/wfe/#/ws/forge/run/freework/run-004>
- Screenshots: [shots/](shots/), with fixture, deployed and agpc
  captures.
