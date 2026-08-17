# p2 plan — Front dispatches to `create-`, not `run-`

Goal (reconcile): the simplified `agfront` guide is the whole instruction, and
the machinery matches it. Front answers casual chat itself, writes `create.md`
when the Developer wants a media asset, and politely refuses coding / tooling /
research. The `run-` dispatch route disappears from Front. Proof is three
conversations in `#front` (below).

Backward compatibility with p1's `dispatch.md` is not required. The `run-`
topic route itself is *not* deleted from agautolab — the Omni Agent still fires
it by hand; only Front's path to it goes.

## Facts discovered during planning

- The simplified guide is already on disk, uncommitted
  (`agfront/agent/guides/front/guide.md`, 3 lines). It names `create.md` and
  says nothing about topics, channels or dispatch tables. That silence is the
  design: the agent writes *content*, the handler decides *where it goes*.
- Its media list — image, video, music, speech audio — is exactly agforge's
  four toolsets (`agforge/agent/toolsets/toolset-{image,video,music,speech}.md`).
  So Front's own branch and forge's actual capability already agree; nothing
  needs to teach Front a narrower or wider list.
- `create-` is agforge's request prefix
  (`agforge/src/agforge/zulip_listener.py`, `REQUEST_TOPIC_PREFIX`). Any
  non-forge post in a `create-*` topic in **any channel forge is subscribed
  to** starts the front → generator pair. The topic body *is* read, unlike
  `run-`; so the create request must be a real description, not a button.
- Checked 2026-08-17 via the API: `forge-bot` is subscribed to
  `FreeForge, general, ops, sandbox, pj-*`; `front-bot` to `front, general`.
  **`#general` is already the overlap** — no subscription change is needed, and
  Front's outbound channel constant stays as it is.
- No bot loop, by filter: forge sweeps `create-`/`runcreate-`, Front sweeps
  `front-` only. `agautolab` is also in `#general` and also reacts to
  `create-` topics, but only when the last message **mentions it by name**
  (`agautolab/src/agautolab/zulip_listener.py`, the mention gate). Front must
  not mention autolab in a create request.
- Nothing comes back to `#front`. Forge answers inside the `create-` topic.
  Same honesty rule as p1: the reply into the `front-*` topic says what was
  sent and where, and does not promise follow-up.
- The listener's own dispatch machinery is ~30 lines
  (`agfront/src/agfront/zulip_listener.py`: `DISPATCH_FILE`, `parse_dispatch`,
  `handle_dispatch`). `parse_dispatch` exists only because `dispatch.md`
  carried a topic name on line 1. With `create.md` the topic is ours to name,
  so that function is deleted rather than adapted.
- nintent already declares Front's channels as `[front, general]`
  (p1 `report6.md`). Keeping `#general` means the desired state needs no edit —
  `nctl drift` staying at `drifting=0` is the check, not a change.
- launchd unit `com.agdev.agfront-zulip` is loaded and running (pid seen
  2026-08-17), alongside `agforge-zulip` and `agautolab-zulip`. Reload with
  `launchctl kickstart -k`; no plist change this phase.

## Steps

Write `report<N>.md` beside this plan after each step.

### Step 1 — the guide, finished and committed

The simplified guide is the source of truth for the rest of the phase, so it
lands first. It is now complete as written — the media list (image, video,
music, speech audio) matches forge's four toolsets — so this step is mostly
commit-and-move-on. One optional line to consider:

- Say that whoever creates the asset answers elsewhere, so Front tells the
  Developer that instead of implying the result will land in `#front`. Add it
  only if step 4's live runs show Front promising a reply it cannot deliver —
  otherwise this is Anxiety-Driven Guidance and the guide stays at three lines.

Do **not** re-add a dispatch table, a channel name, or a topic-naming rule.
Those are the handler's job now, and re-adding them would walk back the
simplification this phase exists to serve. Commit the guide on its own.

### Step 2 — listener: `create.md` out to a `create-` topic

In `agfront/src/agfront/zulip_listener.py`:

- `DISPATCH_FILE = "dispatch.md"` → `CREATE_FILE = "create.md"`; delete
  `parse_dispatch` and its `ListenerError` branches.
- `handle_dispatch` → `handle_create`: no file, no post (a Front that only
  chatted or refused is the normal case). A file with an empty body is still
  an error — an empty post into a channel other agents watch is noise.
- Topic name is derived, not written by the agent. Proposal:
  `create-<front topic minus its prefix>-<generation number>` — e.g. the front
  topic `front-20260817-advance` generation 1 →
  `create-20260817-advance-1`. Unique per generation (a second request in the
  same conversation gets `-2`), and it reads back to the conversation that
  caused it. `topic_write(topic, body, channel=DISPATCH_CHANNEL, client=…)`
  is unchanged.
- The body is the file, verbatim. Front's *answer* is relayed verbatim too and
  never parsed, as in p1 — no mention of `@**autolab**` should ever be
  synthesised here (see the mention gate above).
- Reply section: `asked forge in #general > <topic>; the reply will appear
  there`.
- Update the module docstring: it currently explains `dispatch.md`, `run-`,
  and "whoever picks the dispatched topic up". Name forge and `create-`.

### Step 3 — role table and tests

- `role_run.py`: the `ROLE_ALLOWED_TOOLS["front"]` comment names `dispatch.md`
  as the one file `Write` exists for. Update the name. The tool list itself
  does not change — Front still routes and still gets no shell.
- `tests/test_zulip_listener.py`: the suite pins exactly what agfront decides,
  so it changes with it — one post into `#general`, the derived topic name,
  absence of `create.md` is not a failure, empty body is. Keep the sibling
  rule: nothing asserts what an agent said. Run with `profile = "stub"`.

### Step 4 — live reconcile, three conversations

From the Developer account in `#front`, one `front-*` topic each:

1. **Casual chat** ("こんにちは" / a question Front can answer in text) →
   Front replies in the topic and **nothing is posted to `#general`**. The
   negative is as much the proof as the positive; check `#general` and the
   listener log.
2. **Asset request** ("こういう画像/動画がほしい") → one `create-*` topic
   appears in `#general` carrying the description, and **agforge acks and
   serves it**. Forge asking a clarifying question back inside that topic
   counts as proof of route; a finished asset is a bonus, not the bar.
3. **Complex request** (coding / deep research) → a plain refusal, no post.

Evidence: topic links or screenshots into `agfront/.local/evidence/`,
summarized in `report4.md`. Restart the listener first
(`launchctl kickstart -k gui/$UID/com.agdev.agfront-zulip`) and confirm the
profile is `sonnet`, not the stub used in step 3.

### Step 5 — documentation and desired state

- `devdocs/README_DEV.md`: the agfront line ("sends messages to other agents")
  still holds; adjust only if step 4 taught otherwise.
- `pj-agdev/.local/devenv.md`: its agfront section describes the `run-`
  dispatch — rewrite that paragraph to the create route.
- `nctl drift` → still `drifting=0`; no desired-state edit expected because
  the channel set is unchanged. If it drifts, that is a finding for the
  report, not a silent fix.
- Commit and push per `localrule.md`; the agfront checkout on agstudio is the
  running one, so a push plus a listener restart is the whole rollout.

## Out of scope (later phases)

- Results flowing back from a `create-` topic into `#front`.
- Rewiring agdevworld's chat panel onto `front-*` topics.
- Removing `run-` from agautolab; the Omni Agent still uses it.
- Any Front route to development work (missions, project starts). The
  simplified guide refuses those on purpose — restoring one is a new braindump,
  not a leftover of this one.
