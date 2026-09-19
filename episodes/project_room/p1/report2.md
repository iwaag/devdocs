# Project Room p1 — Step 2 report: conversation reads and posting

Plan: [plan.md](plan.md), step 2. Code: `pj-agdev/agdevworld/agentroom`
(`src/agentroom/projecttalk.py`, `tests/test_projecttalk.py`, routes in
`server.py`, wiring in `main.py`, README section under `/projects`;
`projectroom.py` gained `work_row`).

## Delivered

A project-specific talk door on the relay, on the same Developer credential
and text checks as the rooms' writes (`chat.py`):

- `GET /work/<anchor>` — the mission or task wearing that anchor, located
  where the message is now: visible history (no selfnotes), the record row,
  the project stub with its argue anchor, and a `destination` block naming
  the channel/topic, whether it is resolved, its role (`planning` for a
  mission, `execution` for a task) and the responsible instances read off the
  rosters (`autolab-agstudio1` on this realm), with a one-line label.
- `GET /projects/<key>/topics/<topic>` — an unrecorded conversation by name:
  a document (`role: none`, not postable, plus a `front` block pointing at the
  Front Desk and the origin argue), a setup or a plan with no `[mission]` note
  (`role: planning`). A recorded conversation answers with its anchor instead,
  so a reused topic name never shows another mission's history.
- `POST /work/<anchor>/post {text, token, resume?}` — a plan comment goes into
  the planning conversation, a run comment into the execution conversation.
  A ✔'d target answers 409 `needs_resume` unless `resume: true` is sent; then
  it is un-resolved first and the result says `resumed: true`.
- `POST /projects/<key>/topics/<topic>/post` — the same into a setup or an
  unrecorded plan. A document is refused (403) with the `front` path.

The status of a project conversation is judged against **who serves it**: a
post by a responsible bot is `answered`, its ack `received`, anybody else's
post (Front's setup ask included) is `waiting`. The argue room's rule, where
Front is the answerer, does not apply in a project channel.

`/chat` is unchanged: it still accepts only routine conversations.

## Checks

`uv run pytest tests/` in `agentroom`: **323 passed** (10 new, mocked poster,
no Zulip). Cases from the plan:

- exact destination after rename: a mission retired aside to
  `✔ retired-workplan-one-m<id>` while another mission takes `workplan-one`;
  a comment on the new anchor lands under the name, one on the old is refused
  as resolved and, with `resume: true`, is un-resolved and lands under the
  retired name; a plain rename of a task topic is followed without a resume;
- reused topic names: reading `workplan-one` by name answers with the anchor
  and refuses to post by name;
- duplicate submission: the same token returns the first result with
  `duplicate: true` and one post;
- uncertain delivery: a raised send is `uncertain` with a "read before
  sending again" note, and a repeat of that token repeats the uncertain
  result rather than posting;
- completed target: a resolved task needs `resume: true`; deleted target: the
  anchor is absent, nothing is sent, and the read says so;
- documents: refused with the Front Desk and `?view=argue&argue=<anchor>`
  paths; a stranger topic (`notes`) is refused too;
- refusals that cost nothing: missing token, empty text, a hand-typed
  selfnote, over-long text, an unconfigured chat credential;
- stale mirror: status `unknown` with `stale_state`, history still returned;
- routes: 200/404/409/403/503 through the HTTP door; repeated reads make no
  Zulip call.

No live post was made in this step; the live round trip is step 5.

## Notes

- `resume` is opt-in at the API so a room cannot reopen finished work by a
  stray Enter. The room (step 3) will surface the 409 as a confirm.
- Responsible agents are derived from roster ownership, so a `workplan-`
  topic in a project channel is attributed to every autolab instance whose
  prefix matches, the same way the ops board attributes it.
