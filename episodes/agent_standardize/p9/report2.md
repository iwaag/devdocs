# p9 step 2 — agforge: name the trigger once

AI-generated (Omni Agent, 2026-08-21). Commits `0cfe8a8` in `pyagag` and
`2dcf0e1` in `agforge`, both pushed; listener kickstarted, introduction
re-posted.

## The problem, from p8's proof

One forge delivery posts twice — the record into the `assetrun-` topic it was
triggered from, and the delivery into the `assetplan-` topic where the
requester was talking. Both named the trigger poster, and being named is how
a participant's next run happens at all, so p8 watched Front be served twice
and tell the developer "done" twice for one image (messages 891, 893).

Harmless for Front. In p9 the requester is a supercoder with a checkout of a
live repository, and the second turn is the integration done twice.

## Which post names them

The **delivery**, in the `assetplan-` topic. p8 called this a genuine design
question: the delivery is what the requester waited for, the run topic is
where they last spoke. The delivery wins because it is the answer and the
other is a record — the requester asked for an asset, not for a run report,
and the post that carries the download URL is the one whose arrival means
their next turn is real work rather than an acknowledgement.

## The change

`pyagag`: `serve_topic(..., handoff=False)` posts the reply without the
`@**<last other speaker>**` prefix. The default is unchanged and stays the
turn-taking rule — the skeleton names the next speaker so no guide has to.
The flag is for the serving that is a record rather than an answer, and the
caller says which of its posts is the one that counts. The `handoff_mention`
lookup is not made at all when it is off, so it is one API call cheaper too.

`agforge`: `handle_assetrun` passes `handoff=False`. `deliver_to_origin` is
untouched — it already called `handoff_mention` itself, which is why the
delivery keeps naming the trigger.

## The introduction

Re-posted, because the wording changed and the introduction is the contract
other agents plan around:

> I name you **once** per result, in the `assetplan-…` topic — that post is
> your turn. The copy in the `assetrun-…` topic is the record of the run and
> names nobody, so one result never brings you back twice.

Posted 2026-08-21, revision `2dcf0e1`, under `intro-agforge-agstudio1`.

## Tests

pyagag 362 green (was 361); agforge 191 green.

- `test_handoff_false_posts_the_reply_without_naming_anybody` (pyagag) — the
  reply is the body alone, and the lookup is skipped.
- `test_the_result_reaches_both_topics_and_only_one_names_the_trigger`
  (agforge) — renamed from `test_the_result_reaches_both_topics`, which
  asserted the old two-naming behaviour. Still three posts in the same order;
  the delivery starts with `@**Developer**`, the record contains no mention
  at all.

## Kickstart

```
06:48:22Z agforge zulip listener starting (pull sweep: ... + DM thread)
06:48:22Z full sweep: 0 awaiting, 0 mentioning, 18 calls spent, 981 left in the window
```

Zero runs. forge owns every topic it sweeps and has no callback route, so the
served marker of step 1 does not apply to it — there is nothing for forge to
mark, because it is never the participant.

## Note on criterion 2

This makes *forge's delivery* start one run. The other half of "exactly one
supercoder run per callback" — the startup sweep starting none — is step 1's
marker, which agautolab adopts in step 3.
