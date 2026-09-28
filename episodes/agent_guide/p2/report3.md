# agent_guide p2 — step 3: text several agents share, written once

## The mechanism

pyagag now ships guide text as package data, `agag/guides/*.md` (pyagag
`72bf6da`, `dfd08db` on `agent-guide-p2`), next to `REPLY_GUIDE` and
`CONTINUATION_GUIDE`:

| file | what it says | appended by |
|---|---|---|
| `board.md` | the board is Zulip reached by `agentchat`, not the filesystem; what is on it; a name you do not know is on it; `agentchat --help` is the index; reading is free, a post runs its addressee and a "how is it going?" restarts their job (as9); `intro`, `channels --prefix`, `topics`; ✔ is finished (fd-wr) | `prompt_with_guide(…, shared=("board", …))` |
| `callback.md` | each serving ends and the conversation does not; ask with `send` where the introduction says, reply, finish; the callback brings the answer back beside the chatlog (as5-8); an ack is not the result; answer their questions with `send` | `shared=("callback", …)` |
| `refs.md` | the references pointer, one text for both of the old forms (the long "`agrefs` reads what the developer has published …" and the short "The developer publishes shared context …"): what a reference is, `agrefs list`, `agrefs --help`, read at the named revision before answering, keep the exact reference with its commit | `shared=("refs", …)`; argue participants by default |
| `entrance.md` | the entrance's fixed half: who asks, answer from the chat and start no work, ✔, list afresh every time (as10, with its trial), close out only when asked, never `send` into this channel | `agag.entrance.entrance_guide`, always |
| `entrance_default.md` | the default vocabulary (`{prefix_line}`, `{request_line}`) | `agag.entrance.default_guide`, for an agent with no `entrance_front/guide.md` |
| `argue_participant.md` | the text `participant_guide()` returned as a Python string | `agag.argue.participant_prompt` |

The API:

- `agag.topics.SHARED_SECTIONS = ("board", "callback", "refs")`, in the
  order they are appended whatever order a caller names them in.
- `prompt_with_guide(lines, guide_text, *, reply, continuation, shared=())`
  appends the named sections right after the role's own guide and before
  the reply and continuation sections. An unknown name is a `GuideError`;
  a missing or empty file is fatal, as a missing guide is.
- `shared_sections(names)` and `shared_text(name)` for the compositions
  that do not go through `prompt_with_guide` (forge's generators and
  cagent's operator take the guide as their whole prompt; autolab's review
  serving appends `review.md` after the sections).
- `participate(…, shared=("refs",))` takes a tuple or a function of the
  invitation, because one account can speak for participants with
  different tools: archsage passes `()` for a sage (it holds `sagetree`
  only) and `("refs",)` for itself.

The choice per role is the agent's code, by what the role holds and does:
`board` for a role that holds `agentchat` and reads the board, `callback`
for one that delegates with `send`, `refs` for one that holds `agrefs`.
`agents.toml` was not used: the config schema (`ag.agent-config.v2`)
validates keys, and a role's sections follow from its grant, which the
code that composes the prompt already knows.

## Who gets what

| agent | role | sections | removed from its own guide |
|---|---|---|---|
| agfront | desk, front, routine_run | board, callback, refs | `shared/board.md`: the agentchat paragraph, the intro/channels bullets and the agrefs bullet (Front's file keeps the developer's assumption, the working directory and the chatlog format). `shared/work.md`: "Each serving ends; the conversation does not" (now `callback`) and the ✔ half of "before posting, read it" (now `board`, with fd-wr's reference) |
| agfront | argue | board, refs | — (it invites by mention, not by `send`) |
| archsage | archsage | board, callback, refs | the agrefs pointer (AS21), the generic half of "when you delegate" (AS16; the `agproject` half stays), "your reply is posted for you" (AS20, which REPLY_GUIDE says) |
| archsage | a sage in an argue | none | — |
| autolab | superdirector | board, refs | the agrefs pointer (WP15) |
| autolab | supercoder | board, callback, refs (before `review.md` on a review serving) | the agrefs pointer (WR15); WR12/WR13 ("the introductions file says how…; post and finish, you will be called again") become one line pointing at the sections |
| autolab | entrance | pyagag's fixed half + its vocabulary | everything but the vocabulary (EA1, EA3's generic half, EA5's resolve half, EA6) |
| autolab | argue | refs (participant default) | the agrefs pointer (AA3) |
| autolab | — | — | `autolab-front/guide.md` deleted: nothing reads it (AF1) |
| agforge | assetplan front | refs | the pointer (FP3) |
| agforge | planner, run (generator) | refs | the pointer (FG7, FR7) |
| agforge | entrance | pyagag's fixed half + default vocabulary | `entrance_front/guide.md` deleted: it was the default vocabulary with forge's prefixes, word for word (EF1, EF2) |
| agforge | argue | refs (participant default) | the pointer (FA3) |
| agobserver | intake | refs | the short variant (OI6); one line stays: `agrefs` is reached through the `run` tool |
| agobserver | argue | refs (participant default) | the short variant (OA3) |
| cagent | front, operator, argue | refs | the short variant, three times |

Every trial reference that was in a moved paragraph moved with it: as9
("how is it going?") and fd-wr (✔) into `board.md`, as5-8 into
`callback.md`, as10 into `entrance.md` and, in its concrete form, into
autolab's vocabulary.

## agfront's `board.md` belongs half in pyagag

The plan asked whether agfront's `shared/board.md` belongs in pyagag. Half
of it does and moved: the board facts every conversational agent needs.
The other half stays in agfront, because it is not true of the other
agents: "the developer speaks to you as someone who knows every project"
(Front's entrance position), the working directory's `threads/` and
`tools/` (Front's servings write them), and the `[name #id] sender <user
id> · <time>` chatlog format (agfront renders its own chatlog with
message ids; archsage and autolab render `[name] text`). Grant differences
(who holds `agproject`, `agrun`) stay in each agent's files, as in p1.

## The duplicate check

Over every `agent/guides/**/*.md` in the six agents plus
`agag/guides/*.md`, lines longer than 40 characters, counted:

| before (step 1) | after this step |
|---|---|
| 8 × each of the 4 lines of the long agrefs pointer; 5 × and 4 × the short one's lines; 3 × the title-line sentence; 2 × two autolab lines; 2 × the entrance's "never `send`" line | 3 × "and the rest of the file becomes the description."; 2 × "Your reply to this conversation will be sent to the developer."; 2 × "The file README_PROJECT.md explains how the folders …" |

What is left: the title-line sentence is the output contract of three
agents' own parsers (autolab's plan, forge's plan, cagent's handoff
files), and the two others repeat inside autolab. All three sit in the
heads step 4 rewrites (WP1, WR1–WR2, FG5); the check is run again after
step 4.

## Tests

Against the pyagag branch installed into each worktree's venv (the locks
are updated at deployment):

| repository | result |
|---|---|
| pyagag | 1133 passed (+ new `tests/test_shared_guides.py`: order, unknown names, once each, participants, a sage with no section) |
| agfront | 194 passed, 1 failed: `test_the_listener_is_the_skeleton…` needs `.local/zulip.env`, as in p1 (worktree only) |
| archsage | 40 passed |
| agautolab | 331 passed |
| agforge | 252 passed, 4 failed: the sibling-path tests of step 2 (worktree only) |
| agobserver | 177 passed |
| cagent | 204 passed |

Tests that pinned the old composition were updated to the new one: agfront
(`whole_instruction(role)` = own files + sections; the whole-input tail),
autolab (the entrance is the fixed half + its vocabulary, said once),
forge (no entrance guide of its own; the fixed half once; the plan front's
tail), cagent (the front prompt's tail).

## Commits (branch `agent-guide-p2`)

pyagag `72bf6da`, `dfd08db`; agfront `17e8ca2`; archsage `305b96c`;
agautolab `2a05870`; agforge `f7add0a`; pj-agdev `c5c0124` (agobserver);
pj-clusterintent `798dc3b` (cagent).
