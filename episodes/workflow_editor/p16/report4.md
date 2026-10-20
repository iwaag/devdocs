# workflow_editor p16 — Step 4: archive, synthesize and hand off

The archive, synthesis, knowledge, hand-off and reuse ran on the deployed
system through the agents' own workflows. The Omni Agent posted the
person's words (as `Omni Agent (for Developer)`, marked as trial) and
otherwise only observed, except where noted.

## Archive: incremental, serialized, visible

The archive writer (report2) runs in every executor reconcile. On pj-argue
main it published each discussion's records as they came (`e2d4132` #124,
`c31db8a` #128, … `e1bf479` #136, `094b477` #139, `b8c08ab` #146),
interleaved with WFE's integrations of argue's runs (`a5316d2`, `3fadb6c`,
`0724871`, `7480a1b`) — the writer's files and the runs' files never
conflicted. Status is visible in `wfe discussion archive --status`, in the
discussion view (`archive`), in aghangout's Records panel and on WFE Home.

| Proof | Evidence |
| --- | --- |
| Two discussions publish without losing either | Closing p16-s3-room and posting p16-s4-provenance's brief within one minute: one writer pass made `7217f17` (p16-s3-room, closed state) and `2c6ad2d` (p16-s4-provenance, #140) in the same push |
| Interrupted after the push, before success was recorded | Live: the bookkeeping was set back to an old export (#124, `e2d4132`, state `publishing`) while the repository held #136. `wfe discussion archive p16-s3-room` appended **3 records** (#137 and its events) from the repository's horizon — not 13 from the bookkeeping's — and recorded `e754310` (`.local/p16/step4/recovery-*.txt`). The same in tests, plus a failed push that keeps live messages and a rejected push rebuilt on a moved main |
| A fresh checkout reads it | `git clone` of developer/pj-argue into a scratch folder: `tools/argue validate` ok; p16-s3-room has messages #124–#137, each once, horizon = newest message (`fresh-validate.txt`). After the later exports, both discussions validate on main |

## Synthesis and knowledge (`argue/synthesize`)

The person asked in the discussion to keep it and hand one outcome over
(#137). The facilitator (`facilitate/run-003`) requested
`argue/synthesize/run-001` on the person's behalf and said so (#138).
synthesize/run-001 (default lane, ~45 s of work):

- `discussions/p16-s3-room/summary.md` (`e717460`): rests on the archive
  commit and horizon; the person's intent (#124, #130, #135, #137) apart from
  each agent's view; alternatives and reasoning; conclusions; **both
  disagreements kept**; open questions; the facilitator's own inference
  marked as such. Every claim cites `#<message>` and whose words it was.
- `knowledge/room-image-provenance.md` and `knowledge/index.md`: the finding
  (no prompt/seed for any room; C2PA `gpt-image` on one), the working policy
  "agreed in part", sources `argue@e4937184:…transcript.jsonl#<msg>`,
  *Revisions*.
- A synthesis message in the discussion (#139) naming the commit.
- Integrated by WFE (`0724871`).

Flaw seen: the knowledge file cites `summary.md` at the archive commit
`e4937184`, where summary.md does not exist yet (it is at `e717460`). Left as
found; the transcript citations are correct.

## Hand-off with pinned provenance

synthesize/run-001 requested `aghangout/change/run-002` as an ordinary
request on the person's behalf (`request.md`: goal, the facts to record, the
constraint "do not invent a prompt, seed or model", **Source (pinned):
`argue@e7174609…:discussions/p16-s3-room/summary.md` and
`knowledge/room-image-provenance.md`; messages #136, #131, #127, #137;
archive transcript `argue@e4937184de19`**) and linked it back
(`wfe discussion link … --as handoff`, shown under #137 and in the run
list). aghangout-agent changed `app/public/ASSETS.md` (`148a8aa`, citing
`argue@e7174609f1c0:knowledge/room-image-provenance.md`), deployed it
(`scripts/deploy.sh`, served release `148a8aa`, which keeps the meeting
rooms `8c8b13d`) and completed. The project plan stayed in aghangout; the
cross-project decision stayed in argue.

## Reuse and a changed conclusion

A later discussion, **p16-s4-provenance** ("Where should the provenance of
forge-made assets live?"), asked to start from what earlier discussions
found.

- The facilitator found and cited the knowledge on its own
  (`argue@0724871:knowledge/room-image-provenance.md`, #141) and asked each
  project the part left open there.
- forge (#142) and archsage (#143, answering #142) each **withdrew the broad
  form of its earlier position** and converged on a new rule neither had
  proposed: canonical **per fact**, with the sha256 as the test (making
  facts: forge's record while the hash matches; shipping facts: the shipping
  repository; a different hash names its parent; "not recorded" is never
  filled). Single-agent additions stayed attributed as proposals (#144).
- The person asked to keep it with the earlier conclusion visible (#145);
  `argue/synthesize/run-002` rewrote the topic: **revision 2 is current;
  revision 1's statement, policy, open question and commit (`e717460`) are
  kept under *Revisions* with why it changed**; `index.md` updated;
  integrated (`7480a1b`).

## A failure seen and the guidance it produced

synthesize/run-002 never told the discussion that the knowledge landed, and
the facilitator (#146) had promised to point to it — but a completed
facilitator turn is told nothing. Evidence-driven guidance in pj-argue
`5d74f4d`: the synthesize guide says the discussion learns of its result
only from its own message (citing this run); the facilitate guide says the
requested run reports itself and a turn must not promise a follow-up. The
gap in p16-s4-provenance was left as found.

## Deus Ex Machina notes

- Simulated the lost bookkeeping on the live store to prove recovery (one
  `UPDATE archives`; a test of the writer, not work of an in-system agent).
- Corrected argue's guides twice (`a362eac`: an agent's own request is
  `--request … --on-behalf-of`; `5d74f4d`: above) — handoff candidate for
  argue-agent (a `freework` run on its own guides).
