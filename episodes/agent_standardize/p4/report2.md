# agent_standardize p4 — Step 2 report: the introduction is the contract

AI-generated (Omni Agent, 2026-08-21).

## What changed

`agautolab/params/intro.md` is autolab's self-description, written for an
agent that has never seen autolab. It is posted, and it is the only place
Front can learn any of this from — nothing about autolab is in agfront.

It answers, in this order:

- **What autolab does.** Give it a mission in words; it reads the project's
  source, concept documents and past work logs and writes back a plan and its
  task split. It carries the tasks out too, but only when told to.
- **Where to write.** Development work goes in the project's own `pj-<slug>`
  channel, in a `workplan-…` topic. *With the reason*: the channel is the only
  thing that says which project the work is for, so the same topic elsewhere
  has nothing to plan against.
- **Where not to.** `{instance}` — autolab's own channel — is for questions
  about the instance, and starts nothing.
- **Choosing the project.** Usually the requester names it and the channel is
  `pj-` plus that name. Otherwise: look at what `pj-…` channels exist, or ask
  whoever asked. Explicitly *do not guess* — a `pj-` channel this instance is
  not subscribed to is a project nobody asked it to work on, and a topic there
  is a message into an empty room.
- **Planning is not execution.** A `workplan-…` topic plans only; autolab may
  answer with questions instead of a plan; nothing runs until the requester
  clearly says so. The `work-<label>` channels and `workrun-task<N>-…` topics
  are opened by autolab itself, and **posting into one starts real work.**

That last sentence is deliberate. Front is about to read this, and the phase
forbids firing a `workrun-`. Rather than withholding the word — which would
leave Front to discover the surface without knowing what it costs — the
introduction names it and says what it does. Tool Giving, including the usage
information.

## The host label came out of the tracked files

Writing this intro surfaced a contradiction p1 left behind. `instance.py` says
the instance name is local-only *because the label is the host*, and
`devdocs/README_DEV.md` forbids local machine information in non-ignored
files — yet `agforge/params/intro.md` had `agforge-agstudio1` committed in it.

`agag.intro` now replaces `{instance}` with the instance's name as the post is
made. Both introductions use the placeholder, so neither tracked file carries a
host label, and one introduction serves any instance of that agent — which is
the shape a second instance needs anyway. pyagag `9bc24ac`, and the same
change in agforge in the same commit, not later.

## Evidence

| message | topic | sender | revision |
|---:|---|---|---|
| 707 | `intro-autolab-agstudio1` | autolab-agstudio1 (11) | `36ac829` |
| 708 | `intro-agforge-agstudio1` | agforge-agstudio1 (13) | `1107c74` |

Message 707 renders `autolab-agstudio1` where the file says `{instance}`, and
contains no `{instance}` — the substitution is proven by the post itself, not
only by a test. agforge was re-posted because its source file changed; the
topics are append-only history, so the newest post is the current contract.

The Zulip web links are intentionally not recorded: the realm address is local
machine information.

## Verification

- pyagag: **264 passed** (one new case: a file without the placeholder is
  posted unchanged).
- agforge: **189 passed**; agautolab: **178 passed**. Both intro tests now
  assert that `{instance}` never survives into a post.

## Delivery

pyagag `9bc24ac` · agforge `1107c74` · agautolab `36ac829` — all pushed to
GitHub. pj-agdev `ee01634` records the submodule revisions.

## Not done in this step

- The entrance in `autolab-agstudio1` still answers every question with the
  Step 1 redirect, so "ask in this channel which projects exist" is a promise
  the intro does not make and the code could not keep. The intro sends that
  question to the requester instead. Worth closing later; not this phase.
- Whether a live `pj-<slug>` project exists for Front to be pointed at is
  Step 3.

Wrote autolab's self-description for it — handoff candidate.
