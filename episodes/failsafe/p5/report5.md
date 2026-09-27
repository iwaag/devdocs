# failsafe p5 — step 5: sage refresh bound to its run and revision

Date: 2026-09-27 (UTC). Deployed 13:00Z: every listener, the gateway,
forge's service, cagent and the relay were restarted on pyagag `a44850a`,
with nothing in flight.

## The return relation

**One topic per run.** The study routine guides now ask for the refresh
in a topic of the run's own, e.g. `refresh-growbox-<run id>`, naming the
integrated `main` commit. The run is complete once that refresh is
recorded for it. New versions were posted with `agroutine update` under
archsage's credential:

- `study-growbox` v2, #13876;
- `study-worldtrend` v2, #13877;
- `study-aisvgs` v2, #13878 (the same fixed-topic line).

Deus ex machina note: the Omni Agent wrote these guide versions for
archsage, which is a handoff candidate (archsage owns the guides). The
wording archsage uses for future guides is in its own guide
(`agent/guides/archsage/guide.md`), so the next study it establishes
starts per-run.

**The tool refuses the misroute.** `agentchat send` (and `use`, anything
that anchors a topic) now refuses to post into a topic where this agent's
effective root note belongs to **another request**. It compares the
origins reached by following the agent's own root notes up from both
conversations, not the names. The refusal says where answers would go and
what to do: open a topic of your own, or `agentchat anchor` if the
conversation really moved. Posting within one request stays allowed
(e.g. the desk into a workplan its run opened). The refusal is left as
`[opfail]` in the serving's home. The existing root note carries its
anchor, so an answer still reaches the requesting run after a rename or
restart (`locate`).

## The record

`archsage sage sync <name> --require <commit>` records:

```
[selfnote][sagesync] <sage> <revision> project=<slug> findings=<n> for=<channel>/<topic>#<anchor> includes=<commit>|missing=<commit>
```

- `for=` is read from the asker's root note in the topic archsage is
  serving, so it is the conversation whose request this refresh answers
  (the run, or the desk that asked). Nothing is taken from names.
- `includes=`/`missing=` is `git merge-base --is-ancestor <commit> HEAD`
  in the refreshed tree. A newer revision that contains the required
  result counts, and one that lacks it (not pushed yet, another
  repository) is said to lack it. archsage's guide tells it to use
  `--require` with the commit the request names and to say when the
  refreshed tree misses it.

## The readers

`agag.progress` `knowledge_refreshed` no longer matches project and
time. The required result is the `main` commit(s) the run's plans
integrated, from each close-out's `[change] accepted main=<commit>`. A
`sagesync` after the research's acceptance counts when:

| record | counts when |
|---|---|
| `missing=` the required commit | never |
| `for=` one of this request's conversations | it `includes=` (or is) the required commit; or no commit is known |
| another request's (`for=` elsewhere, or `includes=` only) | its `includes=` or its revision *is* the required commit |
| an older record without either | its revision is the required commit, or, with no commit known, it is in this request's own conversations |

The stage says which commit it waits for (`required`). The relay reads
the same notes as before (`_syncs`), so the panel follows with the pin.
Observer does not read refreshes.

## Verification

pyagag 1045, `test_failsafe_p5.py`:

- a topic returning to another request is refused, nothing is posted
  there, and `[opfail]` is left in the run; a per-run topic is accepted
  and anchored to the run by anchor;
- the same request may post from its desk into work its run opened;
- step 1's baseline flipped: another run's refresh of another revision no
  longer completes this run, while a refresh whose recorded revision
  holds this run's commit does;
- two runs of one study, refreshes arriving in either order: each run is
  completed only by its own (`for=` plus `includes=`), never by the
  other's; `missing=` for this run is not a refresh.

`test_progress.py`: the progress_panel fixtures still complete. Their
refreshes' revisions are exactly the integrated commits.

archsage 39 (the record's `for=`, `includes=` on a real repository,
`missing=` for a commit not in it). Consumers on the pin: agfront 193,
agautolab 331, agobserver 175, relay 363, agforge 265, cagent 204.

## Left for step 6

- A live refresh through a per-run topic in both concurrent studies.
- One study repeated, to check its callback does not return to the
  previous run.
