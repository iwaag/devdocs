# Step 2 — planning and applying the state changes

pyagag `0e38c25`; agdevworld `b602527`; agautolab `111ed7f`; pj-agdev
`c4538b5`. Nothing writes yet outside tests — the live run is step 4.

## What exists now

**The rule is shared, not copied.** `agag.plane` now owns `sub_works`,
`reason_not_completed` and `ALREADY_COMPLETED` — moved out of
`agautolab.mission_done`, which keeps both its names as thin aliases so
every caller and test is unchanged. The Front Desk asks the same rule about
one named Work instead of running an agent's whole-board CLI. Both locks
(`agautolab`, `agentroom`) are upgraded to the new pyagag.

**Two routes**, in `agdevworld/agentroom/src/agentroom/close.py`:

- `GET /frontdesk/<id>/close-plan` → `ag.frontdesk-close.v1`. Ordered
  `actions`, each with `kind` (`work`/`topic`/`channel`/`conversation`), a
  stable `key`, a `state` (`ready`/`done`/`blocked`/`kept`), the `reason`
  and the `detail` it was decided from; plus `counts`, `excluded`, `gaps`,
  `status` (which credentials this relay has) and a `fingerprint`. It writes
  nothing.
- `POST /frontdesk/<id>/close` `{fingerprint}` → the same payload with
  `results` (per target: `applied`/`already`/`failed`/`skipped`) and
  `partial`. **The body carries the approved fingerprint and nothing else**:
  the server closes what it re-derived, never what a browser named — a test
  posts a list of foreign topics and channels and watches them be ignored.
  `409` when the plan has changed since the preview, carrying the fresh plan,
  with **nothing written**.

**How each target is decided.** A Work by the shared rule: every live
Sub-Work completed → ready; already Done → `done`, a successful no-op;
cancelled → `kept`, untouched; unfinished children, or a standalone Work
with none, → `blocked` (never manufactured). A Sub-Work the walk never
reached is carried as `unreached_children` and still counted. A topic is
resolvable only if it was actually read — `note-only` is a gap, not a
finished conversation. A channel is archivable only under step 1's rule.

**Order is fixed: Plane → topics → channels → the Front conversation.**
The Front topic is last and is skipped when anything above is blocked or
failed, because a ✔ there is the claim that the whole thing is finished.
That is what makes "partially closed" honest rather than invisible.

**Retrying is re-running.** Nothing rolls back and nothing retries
automatically; every action is idempotent in the realm's own terms, so a
second click after a partial failure finishes what is left and repeats
nothing. A per-conversation lock serialises two clicks. A small in-memory
record (20 operations) exists for the screen, not for correctness. The
preview's fingerprint covers *which targets and what state each is in* — a
new post in a related topic is not a different decision; a Work cancelled
since, or a target that has appeared, is.

**Credentials.** Reads use `AGENTROOM_ZULIP_ENV`; Zulip writes use
`AGENTROOM_CHAT_ZULIP_ENV` (the Developer — already the credential that
posts here, so no new identity); Plane uses a new **`AGENTROOM_PLANE_ENV`**,
a path to an ignored file, added to `devenv/launchd/com.agdev.agentroom.plist.in`
and pointed at the admin key, because the per-agent `autolab` key is HTTP
403 on the Ghtrends states. Unset or refused is reported in the preview's
`status`/`gaps`, not discovered on submit. Every payload also carries the
sentence *"this closes work; it does not stop a running agent"*.

## Verification

`agentroom` `uv run pytest -q` → **214 passed** (24 new in
`tests/test_close.py`, over step 1's fixtures): the fixed order; each verdict
of the shared rule including a cancelled Work that is then not written; an
unreadable topic blocking; a blocked target keeping the Front conversation
open; one topic's rename failing while the Work and the other topics still
close; a channel archive refused per target; a retry finishing the rest and
writing Plane once; closing an already-closed conversation writing nothing;
a stale fingerprint refusing with no write; two concurrent clicks archiving
once; and the four HTTP shapes (200, 200-with-ignored-body, 409, 400).
`pyagag` → **465 + 5 new**; `agautolab` → **219 passed** on the upgraded pin.

Live **read-only** previews (Developer Zulip credential, admin Plane key,
empty engine memory):

| conversation | ready | done | blocked |
|---|---|---|---|
| `front-desk-20260908-161951` | archive `#work-g-15` | G-15, G-16, both topics, the ✔'d conversation | — |
| `front-desk-20260908-1600` | **G-13 Done**, ✔ `workplan-trend6`, archive `#work-g-13`, ✔ the conversation | G-14, `workrun-task1-g-13` | — |
| `front-desk-20260908-ex1` | ✔ `assetplan-…`, ✔ `assetrun-…`, ✔ the conversation | F2-28 | — |

No gaps, no Plane errors, no exclusions in any of the three.

## Notes for step 3 and 4

- `AGENTROOM_PLANE_ENV` is in the plist template but the **running relay has
  not been reloaded**; the live job still serves the pre-p3 code.
- Nothing has been executed against the realm yet. `front-desk-20260908-1600`
  is the natural live target in step 4: it exercises a Plane transition, a
  topic resolve, a channel archive and the Front ✔ in one run.
