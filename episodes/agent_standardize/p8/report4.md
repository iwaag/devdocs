# p8 step 4 — the proof run

AI-generated (Omni Agent, 2026-08-21). **All four success criteria met on the
first attempt.** One image, seven agent runs, 1.10 USD, from the developer's
go to silence in 96 seconds.

| criterion | verdict |
|---|---|
| 1. developer → Front → forge completes, zero human/Omni posts after the go | **met** |
| 2. it stops — five minutes of silence in all three listener logs | **met** (11 minutes) |
| 3. no `[selfnote]` in any chatlog, `threads/` file or default `read` | **met** (0 of 79 files) |
| 4. `participation.py` and `AGENTCHAT_LEDGER` are gone | **met** ([report1](report1.md), [report3](report3.md)) |

## The trail

Two developer posts, both before the go. Everything after 05:31:05 is agents.
`[SELFNOTE]` marks a post hidden from every chatlog, `threads/` file and
default `agentchat read`.

| # | at (UTC) | where | who | what |
|---|---|---|---|---|
| 869 | 05:31:05 | `front/front-20260821-p8-icon` | **Developer** | one paper-airplane icon, flat, transparent, throwaway |
| 870 | 05:31:05 | " | Front | ack |
| 871 | 05:31:16 | " | Front | the proposal — names agforge, the `assetplan-` topic, **and that forge will open `assetrun-` itself**; asks permission |
| 872 | 05:31:28 | " | **Developer** | "Yes, go ahead. Please see it through." — **the go** |
| 874 | 05:31:42 | `agforge-agstudio1/assetplan-paper-airplane-icon` | Front | `[SELFNOTE]` `[rootchat] front/front-20260821-p8-icon` |
| 875 | 05:31:42 | " | Front | the request, in Front's own words |
| 876 | 05:31:42 | " | forge | ack |
| 878 | 05:31:57 | " | forge | the assetplan front's answer (`required_items.md`, `toolsets.csv`) |
| 879 | 05:32:13 | `agforge-agstudio1/assetrun-paper-airplane-icon` | forge | `[SELFNOTE]` `[rootchat] agforge-agstudio1/assetplan-paper-airplane-icon` |
| 880 | 05:32:13 | " | forge | `[SELFNOTE]` `[work] 5b4728ec…/11155d05…` |
| 881 | 05:32:13 | " | forge | `This topic runs F2-18 "Plan: Paper Airplane Icon". Post here to start it…` |
| 882 | 05:32:13 | `…/assetplan-…` | forge | `@**Front**` created F2-18; **`posting in assetrun-paper-airplane-icon starts it`** |
| — | 05:32:13 | | | mention route: `serves front/front-20260821-p8-icon` |
| 884 | 05:32:32 | `…/assetrun-…` | Front | `[SELFNOTE]` `[rootchat] front/front-20260821-p8-icon` |
| 885 | 05:32:32 | " | Front | "Go ahead and generate it. No changes from the plan…" — **the go, into the run topic** |
| 886 | 05:32:32 | " | forge | ack |
| 887 | 05:32:36 | `front/front-…` | Front | reports to the developer where it posted and what forge said |
| 888 | 05:33:41 | `…/assetplan-…` | forge | `@**Front**` the download URL and `[S3KEY]` |
| 890 | 05:33:41 | `…/assetrun-…` | forge | `@**Front**` running / uploaded / delivered to `…/assetplan-…` / **work F2-18: Done yes** |
| 891 | 05:33:54 | `front/front-…` | Front | "Done." — the URL, the durable key, how to re-sign it |
| 893 | 05:34:09 | `front/front-…` | Front | "This one's done." — the second callback, same answer |
| | 05:34:09 → 05:45 | | | **silence** |

**Zero human or Omni posts after 872.** The Developer's two messages were
posted by the Omni Agent from `.local/zulip/developer.env`, as in p7.

### The three hops the phase exists for

1. **Front is called back and knows where it came from.** 882 named Front;
   the listener read the topic, found Front's own root note (874) and served
   `front/front-20260821-p8-icon` — no ledger file involved. The log line is
   the mechanism in one sentence:
   `mention in 'agforge-agstudio1'/'assetplan-paper-airplane-icon' serves front/front-20260821-p8-icon`.
2. **The called-back run answers at home, and talks outward on purpose.**
   That run's *reply* was 887, to the developer. What it said to forge — 885,
   the go — was a deliberate `agentchat send` into a **different** topic, and
   `agentchat` anchored that topic first (884). This is the exact move p7
   said Front never took: "Starting task 1 means posting into a different
   topic — a deliberate `agentchat` act, not a reply. Front never took it."
3. **The run topic is a conversation, not a button.** The generator's
   `chatlog.md` in the Work's workspace, in full:

   ```
   [agforge-agstudio1 (you)] This topic runs F2-18 "Plan: Paper Airplane Icon". Post here to start it, saying anything you want done differently; the result is posted back here and in assetplan-paper-airplane-icon.
   [Front] Go ahead and generate it. No changes from the plan — default sizing/format is fine for this throwaway test asset.
   ```

   Two selfnotes were posted into that topic and neither appears.

## Criterion 2 — it stops

Last run: Front's, ending 05:34:09Z. Checked at 05:44:58Z, **10 min 49 s**
later. The last line of each listener log:

```
agfront    2026-08-21T05:33:54Z mention in 'agforge-agstudio1'/'assetrun-paper-airplane-icon' serves front/front-20260821-p8-icon
agforge    2026-08-21T05:32:32Z assetrun topic 'agforge-agstudio1'/'assetrun-paper-airplane-icon'
agautolab  2026-08-21T05:29:30Z mention in 'pj-assetpipe1'/'create-asset_…' matches no participation; ignoring
```

Nothing after. Not one run was started by any listener.

**Why it stops, precisely.** Front's last two replies went to
`front/front-20260821-p8-icon`, prefixed `@**Developer**`. The Developer is a
human who does not answer, and Front's own sweep skips a topic whose last
real speaker is Front. Nothing was posted outward, so nothing came back.
That is the whole terminator: *a called-back run's reply goes home, and home
is a dead end.* p7's loop needed Front to post into autolab's topic by
reflex; the reflex no longer exists.

The guide's last line — "If you think task is already done, just reply so" —
did its half too: Front's second callback (893) recognised the work was
finished and answered in one short reply instead of re-doing anything.

## Criterion 3 — selfnotes stay invisible

`grep` over every `chatlog.md` and `threads/` file under all three
workspaces:

```
agfront/.local/topics    52 files scanned, 0 selfnote lines
agforge/.local/topics    26 files scanned, 0 selfnote lines
agforge/.local/agentws    1 file  scanned, 0 selfnote lines
```

`grep -r selfnote` over the two workspaces of this proof —
`agfront/.local/topics/front/front-20260821-p8-icon` and
`agforge/.local/agentws/11155d05…` — matches **0 files**. Four selfnotes were
posted (874, 879, 880, 884) and no agent ever saw one. No listener run was
triggered by a selfnote post: every serving in the three logs names a topic
whose trigger was a real message, and the two topics whose newest post was a
selfnote (`assetrun-…` after 880, `assetplan-…` after 874) fired nothing.

## Runs, durations and cost

Nothing came near a ceiling; the whole delegation cost about a dollar.

| run | ended | duration | turns | outcome |
|---|---|---|---|---|
| Front `run-0032` (the request) | 05:31:16 | 10.0 s | 5 | done |
| Front `run-0033` (the go) | 05:31:45 | 15.2 s | 6 | done |
| forge assetplan front `run-0026` | 05:31:57 | 14.0 s | 6 | done |
| forge assetplan generator `run-0024` | 05:32:13 | 14.7 s | 5 | done |
| Front `run-0034` (callback → posts the go) | 05:32:36 | 21.8 s | 8 | done |
| forge assetrun generator `run-0003` | 05:33:41 | **68.3 s** | 11 | done |
| Front `run-0035` (callback, delivery) | 05:33:54 | 11.6 s | 6 | done |
| Front `run-0036` (callback, "already done") | 05:34:09 | 14.4 s | 7 | done |

Front's ceiling is 360 s, the assetrun generator's 1200 s. The longest run
used 5.7 % of its budget. Total 1.10 USD, all `anthropic/claude-sonnet-5`.

## What was produced

`files/2026-08-21/fcd032c079a64345bf9fbd7cbea6b9d5.zip`, one file:
`paper-airplane-icon.png`, 512×512 8-bit RGBA. It is a paper airplane. It
also has a red rounded-square border nobody asked for — a generator quality
matter, outside this phase, and worth noting only because "the asset is
right" was never one of p8's criteria and should not be read into this one.
Plane work F2-18 is `Done`, external id
`agforge-agstudio1/assetplan-paper-airplane-icon`.

## Two findings

### 1. Two callbacks, two identical answers

forge names the trigger in **both** the `assetrun-` report (890) and the
`assetplan-` delivery (888). Front's listener holds one pending-mention entry
per topic, so both were served: `run-0035` and `run-0036`, 891 and 893, the
same answer to the developer twice. It cost 26 s and 0.26 USD, and it does
not loop — both replies go home.

This was predicted in [report2](report2.md) before the run and kept as the
plan wrote it. The cheap fix is to name the trigger in only one of the two
posts; the honest question is which, and it is not obvious — the delivery is
what the requester was waiting for, and the run topic is where they spoke
last. Deferred to `report.md` rather than fixed inside the proof.

### 2. Front proposed the p8 flow before ever seeing it

871, written before Front had exchanged a word with forge:

> 3. Once it registers a Work, it opens `assetrun-paper-airplane-icon`; I post
>    there to trigger generation.

Front learned that from `tools/agents.md`, harvested from `#agents`
immediately before the run — i.e. from forge's re-posted introduction
(message 868), and from nothing else. The introduction-as-contract holds: a
behavior change travelled to another agent as posted content, with no guide
edited on the consumer's side. Step 2 changing the introduction *was* the
step that taught Front the new flow.

## What p7 asked, and what p8 answered

p7 closed with: "nothing in the system decides when an exchange is over…
Neither the waiting nor the routing, but **ending a conversation**." p8 does
not add a rule for ending one. It removes the reflex that could not end:
**an agent's reply always goes to its own requester, and speaking to another
agent is always a deliberate act.** A conversation between two agents then
ends the way it always did with a human on one end — because one side stops
having anything to send.
