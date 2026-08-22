# agag_builder p3 — step 5: live agping check

## End state

- Instance: `agping-agstudio1` (Zulip bot user 18).
- Project location: runsmoke1 `main/agping/`.
- Project commits pushed to its Gitea remote:
  - `9c73b9d` — generated skeleton relocated into the project;
  - `1e85461` — pyagag lock advanced after the live overlay fix.
- Current introduction: `#agents/intro-agping-agstudio1`, message 1429,
  revision `1e85461`.
- Listener: background `uv run python -m agping.listener`, running from the
  project location; startup sweep and the final entrance run succeeded.
- Front's final relayed result: agping said “Hello Front! Greeting received.”

The fixture stays. Its bot, channel, ignored local credential/state files,
tracked skeleton, introduction and background listener are the useful second
generated-agent example requested by this phase.

## Autolab transcript and requester answers

The initial post was message 1367 in
`#pj-runsmoke1/workplan-agping`. Autolab asked one required question: the
request did not explicitly authorize starting. It created R-8/R-9 and opened
`#work-r-8/workrun-task1-r-8`; message 1373 answered “Go ahead”.

The first work run proved the new command end to end:

- `agag init agping --yes --provision --like agautolab`;
- bot creation and `.local/zulip.env` mode 0600;
- `#agents` subscription and `#agping-agstudio1` creation;
- intro message 1376;
- background listener with a clean sweep.

It also exposed a planning error: the superdirector's task said “do this to
autolab itself” and the supercoder generated `pj-agdev/agping/`, while the
phase requires the project's `main/agping/`. A correction posted during the
run was visible only on the following serving. The supercoder then rejected
the correction twice because it conflicted with its own task text, even after
the requester explicitly confirmed it; it marked R-9 Done and resolved the
topic. This is the main next guide/planning lesson from the live run: a
developer correction is authoritative over generated task text, and
`.local/` being moved under a tracked tree does not mean it is committed when
the generated `.gitignore` excludes it.

The still-open workplan was used to recover without human filesystem work.
Autolab updated R-8, cancelled R-9, created R-10 and opened
`workrun-task2-r-8`. The requester corrected one more false inference (the
untracked generated directory had no nested `.git`) and authorized task2.
Autolab then:

1. stopped the old listener;
2. moved the whole tree, including ignored state, to `main/agping/`;
3. verified the parent repo would stage only the 12 skeleton files;
4. restarted the listener from the new location;
5. asked for the required commit/push confirmation;
6. after confirmation, committed and pushed `9c73b9d` and posted intro 1406.

The pre-existing untracked `HELLO.md` and `README.md` in runsmoke1 were left
untouched throughout.

## Front exchange and the failure it farmed

Developer message 1410 asked Front to greet agping. Front found the new intro
and posted message 1413 in `#agping-agstudio1/greeting`. The transport worked,
but the first entrance run returned:

```text
E_OVERLAY_SCOPE: overlay may not introduce roles
```

`--like agautolab` had copied the sibling's `[roles.coding]` local override
into an agent whose committed config declares only `front`. The immediate
ignored-file repair removed that incompatible block. The permanent pyagag
fix, commit `0b226b6`, now copies sibling machine facts while dropping only
role overrides absent from the generated agent. Focused tests (27) and the
full suite (411) passed. agautolab advanced to that commit in `b5e4ee7`, the
pj-agdev pointer advanced in `0ab3e5f`, and its listener was restarted and
swept successfully. The generated project lock advanced in pushed commit
`1e85461`; intro message 1429 carries that revision.

Developer message 1420 asked Front to retry. Front's message 1422 reached the
same greeting topic, agping replied in message 1425, and Front relayed it in
message 1427:

```text
Hello Front! Greeting received.
```

The final role resolution and callback path were therefore successful, with
the first failure retained as evidence rather than hidden.

Deus Ex Machina note: fixed the generated fixture's incompatible local
overlay and generalized `agag init --like` after autolab's live run — handoff
candidate.
