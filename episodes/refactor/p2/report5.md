# p2 step 5 — deployed, demonstrated, and one defect found

The phase's own conclusions are in [`report.md`](report.md). This is the step
record: what was deployed, what the live run did, and what it found.

## Service state before deploying

Read, not assumed:

```
$ launchctl list | grep agdev
com.agdev.agautolab-zulip  com.agdev.agautolab-gateway  com.agdev.agentroom
com.agdev.agfront-zulip    com.agdev.agforge-zulip      com.agdev.agforge
com.agdev.arxivsage-zulip  com.agdev.comfy-notifier          — all running

$ uv run --project nctl nctl agents observe
agforge:            zulip=yes channels=[general, ops] plane=yes
autolab-agstudio:   zulip=yes channels=[general, ops] plane=yes
… ingested through the Nautobot Job
```

`plane=yes` there is a **Nautobot desired-state identity**, not a thing forge
uses — and the plan reserves those identities for a later phase. It is
recorded here because it is exactly the sort of row that would otherwise read
as a contradiction of this phase's claim.

## What was deployed

| what | how |
|---|---|
| agforge (`ea51b58`) | `uv sync` (pyagag `0a33830` → `218cb31`, for `agag.document`) + `launchctl kickstart -k gui/$(id -u)/com.agdev.agforge-zulip` — came up clean, `0 awaiting, 6 calls spent` |
| agentroom relay | plist regenerated without `AGENTROOM_PLANE_ENV`, installed, `bootout` → wait → `bootstrap` (`kickstart -k` does not re-read `EnvironmentVariables`); `/healthz` `{"ok": true}`, banner reads `completion on` |
| agdevworld frontend | `npm run build && docker compose build web && docker compose up -d web` — `:8090` answers 200 |
| forge's introduction | `uv run python -m agforge.intro` → `#agents` › `intro-agforge-agstudio1` |

`devenv/launchd/README.md` and the ignored `pj-agdev/.local/devenv.md` were
updated in the same pass, because a note describing a credential that no
longer exists is worse than no note.

## Plane isolation on the exercised path

Not a rejecting client — the credential was **absent for the whole run** and
the code cannot load the module:

```
$ ls agforge/.local/plane-credentials.env
No such file or directory

$ cd agforge && uv run python -c "import agforge.zulip_listener,
  agforge.assetplan_topic, agforge.assetrun_topic, agforge.record,
  agforge.cli, agforge.retire, sys;
  print([m for m in sys.modules if m.startswith('agag.plane')])"
[]

$ cd agdevworld/agentroom && uv run python -c "import agentroom.main,
  agentroom.close, agentroom.closing, agentroom.ops, sys;
  print([m for m in sys.modules if m.startswith('agag.plane')])"
[]

$ launchctl print gui/$(id -u)/com.agdev.agentroom   | grep -c PLANE   → 0
$ launchctl print gui/$(id -u)/com.agdev.agforge-zulip | grep -ci plane → 0
```

Both on the **deployed** venvs, and both processes were serving the live run
in that state. The import-graph property is supporting evidence for "no
network call"; the missing credential file is what makes it a live isolation.

## The demonstration

`#pj-refactorp2`, a channel opened for this run, one request through
autolab's ordinary entrance. Every message below is a post a person would
have made.

1. **Plan.** `workplan-splash` → mission **m5790**, two tasks: *get a
   512x512 PNG logo from forge and commit it*, then *add a splash page*.
   `work-m5790` opened with a topic per task.
2. **Start.** "The plan looks right. Please start the mission." → `started`.
3. **Task 1 — the delegation.** autolab opened
   `#agforge-agstudio1 › assetplan-refactorp2-logo` and asked. forge planned
   it and answered:

   ```
   recorded a5814 "Plan: refactorp2 logo asset" (toolsets: toolset-image)
   posting in assetrun-refactorp2-logo-a5814 starts it
   ```

   — the new record and the new run-topic name, live. autolab posted there;
   forge generated through SwarmUI and delivered into the `assetplan-` topic
   with the presigned URL and `[S3KEY] files/2026-09-10/51afa927…zip`.
   **That delivery post is the ordinary callback**: it named autolab, which
   resumed `workrun-task1-m5790`, downloaded the zip, checked the PNG
   (`512 x 512, 8-bit/color RGB`), committed `main/assets/logo.png`
   (`3afe543`), pushed to Gitea and filed the task report in `devlog`.
4. **Task 2.** `main/index.html`, 11 lines, no dependencies; the developer
   approved the content in the topic and it was committed (`645ea17`),
   pushed and filed.
5. **Inspect.** The artifact is real and is what was asked for: a flat
   geometric mark, one orange accent on black, 512x512. It is in the repo at
   `main/assets/logo.png` and the page renders it.
6. **Accept**, through the operation room's completion door:

   ```
   ready  work         work:m5790                      every one of its 2 tasks is finished; accepting them
                                                       and marking the mission done
   ready  work         work:a5814                      its asset was delivered; accepting it
   done   topic        …/workrun-task1-m5790           already ✔
   done   topic        …/workrun-task2-m5790           already ✔
   ready  topic        …/assetplan-refactorp2-logo     will be marked ✔
   ready  topic        …/assetrun-refactorp2-logo-a5814 will be marked ✔
   ready  channel      channel:work-m5790              will be archived — every one of its 2 topics …
   ready  conversation …/workplan-splash               will be marked ✔ once everything above is done

   counts {'ready': 6, 'blocked': 0, 'done': 2, 'kept': 0}
   status {'zulip_read': True, 'zulip_write': True, 'reason': ''}
   gaps   {'truncated': False, 'unread': [], 'bounded': [], 'errors': []}
   excluded []
   ```

   **Two work records, both read from the conversations, no Plane row, no
   credential warning and no gap** — which is the picture this phase set out
   to produce. The result:

   ```
   applied  work         m5790                    m5790 is done; accepted m5790#1, m5790#2
   applied  work         a5814 refactorp2-logo    a5814 is accepted; its asset is files/2026-09-10/51afa927…zip
   applied  topic        assetplan-refactorp2-logo is ✔
   applied  topic        assetrun-refactorp2-logo-a5814 is ✔
   applied  channel      #work-m5790 is archived
   applied  conversation workplan-splash is ✔
   partial: False
   ```

   `#agforge-agstudio1` — a shared agent channel — was not archived and was
   never a candidate. Only `work-m5790` was.

### Read back through the anchors alone, after everything closed

```
a5814: state='accepted' at agforge-agstudio1/'assetplan-refactorp2-logo'
       stem='refactorp2-logo' title='Plan: refactorp2 logo asset'
       tools=['toolset-image'] replaces=None
       results=['files/2026-09-10/51afa9273ce043769fbbce1519f8c011.zip']
       run topic='assetrun-refactorp2-logo-a5814'
run r5820: state='delivered' request=5814
       results=['files/2026-09-10/51afa9273ce043769fbbce1519f8c011.zip']
```

The plan, the toolset selection, the execution relationship, the outcome and
the durable result reference — all of it out of Zulip, through one message
id, after both conversations were resolved.

Git holds the durable half: `main` at `645ea17`, pushed, with
`assets/logo.png` and `index.html`, and two task reports in `devlog`.

**Cost**: 10 agent runs, ~335 s of run time, **$1.29**.

## The defect it found

The read-back above first said `state='delivered'`, not `accepted`.

The operation room accepts with the **Developer's** credential — that is what
acceptance means — and forge's own reader took a conversation's state notes
only from the record's own author. So the room's `accepted` note landed in
the request's topic and was invisible to forge: asked "where do my plans
stand", forge would have gone on calling an accepted request merely
delivered, and offered to work on it again.

This is `refactor` p1 step 4's defect, on forge's half, and it has the same
answer: `agforge.anchor.EXTERNAL_STATES`. `accepted` is read from any sender;
every other state stays forge reporting its own work, so nobody can claim
`delivered` for a generation they did not run; and newest still wins, so
running a request again after an acceptance is the newest word. Three tests,
one of which fails without the fix. Re-verified live after redeploying — the
read-back above is the *second* one.

It is worth recording that this is the only defect the live run found, that
it is the same seam as last time, and that the agentroom side of it was
already right — `agentroom/forge.py` was written with `EXTERNAL_STATES` from
the start, precisely because p1 had recorded it. What was missed was that the
rule has **two** readers, and only one of them was carrying the lesson.

## Tests and build

```
pyagag      543 passed
agforge     221 passed   (+3: acceptance written by another sender)
agautolab   237 passed   (untouched by this phase)
agentroom   267 passed
agdevworld  tsc clean; vite build ok; web container rebuilt on :8090
```

## What was done through the route rather than the canvas

The preview and the acceptance were made through `GET /complete/plan` and
`POST /complete` with the fingerprint the preview handed back — which is
exactly and only what the browser's button does, including the stale-approval
refusal. The board itself was inspected in the browser: the screenshot in
`agdevworld/.local/shots/p2/` (ignored) shows the operation room listing
`assetplan-refactorp2-…` and `assetrun-refactorp2-l…` as **DONE · resolved**
under the new run-topic name. Driving the Phaser canvas to press the button
was not attempted.

## What was left alone

- **Front supervision** was not part of the scenario, as the plan allowed.
- **The asynchronous ComfyUI path was not exercised live.** The demonstration
  used image generation, which is synchronous through SwarmUI and returns in
  seconds. Pending jobs, restarts, repeated callbacks and a late result after
  a replacement are covered by controlled fixtures (step 3) and are reported
  as fixtures, not as live evidence.
- **Retirement/replacement was not exercised live** either, for the same
  reason: its four properties are fixture-tested, and the demonstration had
  no wrong request to retire.
- **agautolab1** runs older code and no listener; it was not updated.
