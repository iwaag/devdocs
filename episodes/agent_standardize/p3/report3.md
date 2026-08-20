# p3 step 3 — cutover

One sitting, both listeners quiesced. The realm now speaks
`assetplan-` / `assetrun-` / `workplan-` / `workrun-` and nothing else.

## 1. Listeners stopped

```
launchctl bootout gui/$(id -u)/com.agdev.agforge-zulip
launchctl bootout gui/$(id -u)/com.agdev.agautolab-zulip
```

`com.agdev.agfront-zulip` was left running, as the plan allows, and
`com.agdev.agautolab-gateway` never speaks Zulip.

## 2. Old-vocabulary state wiped

**Zulip: 31 topics deleted across 10 channels, 0 failures** — every
`create-`/`runcreate-`/`mission-`/`run-` topic, resolved (`✔ …`) ones
included, via realm-admin `POST /streams/<id>/delete_topic` with
`.local/zulip/developer.env`. `#agforge-agstudio1` 5, `#general` 9,
`#pj-foodchain` 5, `#pj-runsmoke1` 1, `#pj-runsmoke2` 1,
`#pj-simpleshooter` 1, `#work-r-1` 2, `#work-r2-1` 2, `#work-s2-1` 4,
`#zz-allpublic-20260813` 1. A re-run of the survey finds none left.

**Plane: 11 pending Works cancelled, 0 left.** Cancelled, not deleted —
Plane keeps the row, which is the ledger the earlier episode reports point
at, and a cancelled Work is as unselectable as a missing one. The target was
exactly the *selectable* set: order key embeds an old topic name **and**
state group is `unstarted`. Anything completed/started/cancelled was already
inert and was left alone.

| what | why it was pending |
|---|---|
| `F2-12`, `F2-13`, `F2-14` (`forgeauto`) | the only genuinely live ones — agforge's `assetrun` sweep would have picked these up and tried to deliver into `agforge-agstudio1/create-…`, a topic that no longer exists |
| `F2-1`…`F2-5` | unstarted but unlabelled, so never eligible; cancelled anyway so the board carries no dangling old keys |
| `P2-1`…`P2-3` | same, on the autolab side (`pj-phase2omni/mission-probe…`) |

No renames, no `✔` ceremony, one sweep each — as the plan specified.

## 3. Listeners restarted on the new code

`agforge` at `2469d69`, `agautolab` at `5675943`, both working trees clean.
The startup log lines are the evidence that the sweep really changed:

```
2026-08-20T15:11:20Z agforge zulip listener starting (pull sweep: all topics
  in 'agforge-agstudio1', prefixes ('assetrun-', 'assetplan-') elsewhere
  + DM thread)
2026-08-20T15:11:20Z agautolab zulip listener starting (pull sweep, prefixes
  ('workplan-', 'workrun-', 'assetplan-', 'bmining-'))
```

Both then reported `full sweep: 0 awaiting` — the expected consequence of
step 2: with every old-prefix topic gone and no new-prefix topic yet open,
there is nothing to serve. `bmining-` is present and untouched, as intended.

## 4. Intro re-posted

`uv run python -m agforge.intro` appended to `#agents`, topic
`intro-agforge-agstudio1` (message id 686, third post in that topic):

```
# agforge

This instance generates requested media assets such as images, video, music,
and speech. To request an asset, open an `assetplan-…` topic in this
instance's `agforge-agstudio1` channel and describe what you want.

---
Posted: 2026-08-21
Revision: `2469d69`
```

The revision stamp is the step-1 commit, so a reader can tell this post from
the p1 one without diffing the prose. This is the whole contract Front is
about to be tested against in Step 4: nothing in agfront was taught the new
prefix, so if Front opens an `assetplan-` topic, it learned it here.

## Note

One permission stop: the topic-deletion sweep was blocked by the auto-mode
classifier and re-run after the developer allowed it. No workaround was
attempted in between, per `localrule.md`.
