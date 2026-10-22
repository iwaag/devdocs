# Report: workflow_editor fix3

## Delivered

- **agdevworld `f858dcf`, `3d7b6fb`** (deployed: the service and `bin/wfe`
  run `3d7b6fb`).
  - `server/discussions.ts` `postQuestions`:
    - Every open WFE question of a run that speaks for an open discussion
      becomes a `question` message, once per question
      (key `question:<run>:<q>`). Speaking runs are the discussion's
      invitations' runs and its conductors.
    - Each message carries the asking agent, the run's participant, the
      question id and the node.
    - Its `re` is the message the run was invited for.
    - On the conductor's talk node it is also recorded as the talk
      activity.
  - Posting happens in two places:
    - right after `wfe run ask` (`cli/run.ts`);
    - in `wfe exec reconcile`.

    A run that cannot be read (a retired project's run) is skipped
    silently, like the rest of the reconcile.
  - `status.awaiting`: a non-talk question goes to its `to`. That is a
    person when `to` is the configured person or a person member;
    otherwise it is the agent.
  - `wfe discussion talk` and `contribute --kind question|--question` are
    refused, with a message pointing at `wfe run ask`. `DiscussionStore.talk`
    is removed.
  - The catalog meaning in the brief, in `wfe run nodes` help and in
    catalog.md now reads: "perform it as a node of its own as far as
    possible … not inside another node such as a talk. It is not an
    absolute rule …".
  - Docs: discussions.md *Questions* (new), *Activities*, *Contributions*
    and *Reading* `awaiting`; contract.md *Discussion activities*.
- **agautolab `40f32e7`**: in the wfrun guide, the catalog paragraph says
  "as far as possible", citing gimmickimg run-001…004. A participant's
  question is `wfe run ask` alone.
- **pj-ponderer `ec06b3b`**: in guide.md and conduct.md, a question is the
  run's question on whichever node waits. The `talk` command is gone from
  the tool list.

## Checks

- `npm run check`: tsc, 516/516 tests and both builds. The talk tests now
  ask through `question.ask` + `postQuestions`.
  - New test: a conductor's question on a non-talk node is posted by the
    reconcile and awaits the person. An untargeted reply is its answer, with
    no second run (the run-020 case).
  - The participant test asserts one message per question and
    `awaiting` = Pat/you.
- Before deployment, a read-only preview over the live area found exactly
  one open, unposted question in open discussions: run-020 q1.

## Rescue

`wfe exec reconcile` after deployment posted run-020 q1 as **#301** in
`20261022-201638-8q25lf` (`[question q1 node delegate-try] re #298`).

- `wfe discussion show` now says "Waiting for the person: #301".
- The API's `awaiting` is `[{message: 301, receiver: iwaag, you: true}]`.
- The review page `http://agstudio.local:8300/` answers 200.

The person's reply to #301 (or an untargeted reply) is delivered to run-020
as q1's answer.

## Not done / limits

- A question's presentation link is not added to the posted message. Its
  text carries whatever the asker wrote; the run topic still gets the
  notifier's link.
- gimmickimg's behaviour is untested after the wording change. The next
  gimmickimg run shows whether "as far as possible" is enough.

## Addendum: the conclusion run-020 never spoke (2026-10-22)

The person answered #301 with #302: 「期待していた以上の良好な結果です。今回のargueともども、完了としてください。」 (better than expected; mark it complete, together with this argue).

- run-020 took the answer up and relayed "accepted" to the child.
- When the child completed, a later attempt disposed d1. It completed its
  last ponder node `ponder-after-try` with plain `wfe run complete`.
  - The run completed with no decision, no announcement and no close.
  - The discussion stayed open with nothing said.
- fix3 contributed. Before it, an untargeted #302 would have reached
  run-020 as an instruction from the discussion. Now it arrives as the
  answer to q1 on the delegate node, and was read as the child relay only.

Rescue: Omni invited ponderer (invitation 92). `conduct/run-021`
concluded with #303 and closed the discussion.

Changes (Developer: warnings, not hard barriers):

- **agdevworld `25c74c4`** (deployed). Two completions now record
  nothing; they print the risk and the recommended command, and the same
  command with `--yes` records them:
  - `wfe run complete` or `run adjust --completing` on an announce, ask or
    ponder node;
  - a `run complete` that would end a conductor run whose open discussion
    it never concluded.

  Also: `--yes` is a global flag, documented in the help and in
  discussions.md *Activities*. One new test (517/517 pass).
- **pj-ponderer `b76fcad`**: a person's answer is their speech in the
  discussion and may ask more than the question did. The guide cites
  run-020.
