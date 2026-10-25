# p18 step 4 — one progression in aghangout

Plan: [plan.md](plan.md), step 4. Representation: [report1](report1.md).

## What was built

### WFE (agdevworld `ad227b6`, `ade9d77`)

- **`progression` in the conversation view** (`server/progression.ts`,
  contract in `workflow_editor/docs/conversations.md` *Progression*). It is
  built from records only and holds:
  - the conductor: its run, run view, status and standing;
  - the stretches: the opening one, then one per wake, each with its topic
    and whether it has open work;
  - each node: its messages, outcome, wait and human review round;
  - each node's recipients, as links with their own state and correction
    count;
  - the room's earlier conductor runs and the cancelled one a replacement
    names.

  The recipients' own nodes are not copied in. They are shown by their own
  runs.
- **Run views.** The focus card of a standing conductor at rest, suspended
  or closed says so and links to its room. The project run badge reads
  "Idle", not "In progress".
- **Work counted honestly** (`ade9d77`). A standing conductor that only
  waits for its delegations no longer appears among the obligations: its
  delegations stand for it. Before this change, a conductor with two open
  commissions counted as three things under way. A resting conductor was
  already left out (step 2).

### aghangout (`98d483d`, `6af3a45`)

- The reading view has a **「進行のグラフ」** section, and the same tree
  heads the 進行 panel. It shows:
  - the conductor's state in Japanese: 待機中（次の発言で続きます）,
    依頼先の作業待ち, 進行は休止中, 終了（会話を再開すると同じ進行が続きます）,
    Human review待ち, 中断, 失敗 and so on;
  - the sentence "ひとつの run（…）がこの会話の進行を最後まで受け持ちます",
    with the number of wakes;
  - the stretches. Stretches with open work and the latest one are unfolded;
    completed stretches fold to one line. Topics name stretches, never
    runs;
  - each step with its state, kind, topic and message links;
  - each commission as an expandable line: the recipient, its workflow,
    its state, "同じ依頼先への修正 N回" and a link to its own run.
- **Navigation.** Each message has 「進行 ↑」, which unfolds and focuses
  its step. Each step links back to its messages.
- A conductor at rest has its own status line,
  "Pondererは待機中です。次の発言で同じ進行が続きます。". It is not
  counted in the tab badge or as work under way. Work awaiting human review
  is not counted either.
- A cancelled conductor gets an explicit 「新しい進行役に交代」 action,
  which asks for a recorded reason. Nothing is replaced automatically.
- Earlier conductor runs are listed under 「以前の進行 run」, with their
  relation (earlier, or replaced).

## What the browser check found and fixed

The fixture rooms were screenshotted at desktop and phone widths, in
Chromium and WebKit. Two problems were found:

1. The fixture contradicted WFE. `fx-progression` showed a conductor
   "Idle" while two delegate nodes were waiting, and 「次に必要なこと」
   said nothing was owed. WFE would read that conductor as waiting for its
   delegations and list them. The fixture now matches WFE's reading, and
   the WFE count was corrected as described above. A second room,
   `fx-progression-idle`, shows a conductor that truly rests:
   - 待機中;
   - its first stretch folded;
   - no 作業中 badge.
2. A review step read "Human review Human review", because the kind and
   the name were the same word. A step no longer repeats its kind as its
   name.

## Checks

- New `progression` section in `app/checks/browser.ts` (15 checks):
  - one run, the number of wakes and the waiting state;
  - work counted as the two commissions;
  - which stretches are unfolded, and that they are named by topic;
  - a review step named once;
  - a commission that opens to its own run, with its correction shown as
    a correction and not as a new job;
  - a separate commission as a separate run;
  - message → step and step → message navigation;
  - the 進行 panel;
  - a resting room: 待機中, folding, nothing counted;
  - no sideways scrolling at phone width;
  - reading writes nothing.
- aghangout `npm run check` 33/33. `browser-check` all passed in
  Chromium (340 + 15) and WebKit (355). Screenshots are in the ignored
  `pj-aghangout/.local/shots/*progression*.png`.
- WFE `npm run check`: 619/619, `tsc` and both builds pass.

## Not yet

- The WFE run view for a standing conductor is checked against live data
  in step 6, after deployment.
- aghangout's commits are local to its workspace and are pushed and
  deployed in step 6.

## Commits

- agdevworld `ad227b6` (progression view, run views, with step 5's
  transition) and `ade9d77` (obligations).
- pj-aghangout `98d483d` and `6af3a45`.
