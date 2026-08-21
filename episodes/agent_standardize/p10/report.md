# agent_standardize p10 — the entrance answers about the work

An instance's own channel used to be a placeholder. Asked anything, forge
recited its topic prefix and autolab recited a redirect; neither could say a
word about the work it had actually done. It now surveys its own plans and
their execution stage by reading Zulip, explains them, and — when told to —
resolves the finished conversations and marks the Plane side Done. First
driven by the Omni Agent, then by agfront through nothing but the
introductions.

Step reports: [report1](report1.md) · [report2](report2.md) ·
[report3](report3.md) · [report4](report4.md).

## What changed

**`agentchat` grew two commands** (pyagag `db01afc`). `channels [--prefix]`
prints `<name> — <description>` per line — the description being where a
channel derived from one piece of work names that work, in a sentence a
person wrote. `resolve <channel> <topic>` is Zulip's `✔ ` rename, taking the
name the caller knows on either side of it. Checked live: a bot may resolve a
topic another bot opened, so this needed none of the escalation p1's
archiving did.

**autolab grew `python -m agautolab.mission_done`.** A task's Sub-Work is
closed by the run that executed it; nothing ever closed the Work above them,
so p9 finished a mission and left `S2-30` in `unstarted` with four completed
children — and five earlier missions in the same state. Deciding "every child
is Done" is counting, so it is a deterministic command, not a prompt. It is
also the entrance's *only* Plane operation: everything else it needs is in
the chat, and the mission Work's state is the one thing that is not.

**Both entrances became ordinary servings of `roles.front`** through the
shared skeleton, with a terse guide each (8 lines) and `agentchat` on PATH.
forge needed the chat handover autolab has had since p6; it had none. Nothing
about the entrance executes anything — a `workplan-` name in autolab's own
channel is still not a request to run anything.

**Every role runs on `sonnet`.** autolab's `roles.front` was the last
committed `local` one, and its `nested_harness` requirement belonged to the
in-process backend rather than to the role. The `local` profile stays defined
and unused: Agent ≠ Model, and every run records its backend anyway.

## What it proved

Both entrances surveyed their own boards correctly and, when told to, closed
them out. p9's open item (`S2-30`) is Done, closed by the agent that owned
it. Front reached both entrances knowing only the introductions, instructed
them, and the entrances did the work — `R-4` was found, resolved and marked
Done on Front's instruction with zero Omni posts in the loop. The
attributability grep is clean: no entrance channel name and no topic prefix
of either agent in `agfront/src` or its guide, bar one explanatory comment
nothing reads.

Nine entrance questions cost $2.52. Front's six runs cost $1.65.

## What it cost to learn

Two live defects, both in the guides rather than the code, and both found by
running the thing rather than by reading it.

forge **answered twice** — it posted its survey with `agentchat send` and then
the skeleton posted its closing message. One line fixed it.

autolab **surveyed one project and reported on all of them**, missing the
only project holding finished, un-closed work, and leaning on its own earlier
answer still readable in its channel. The guide caused it: "look up only what
was asked" is right about *depth* and wrong about *breadth* when the question
is "where do all your plans stand". Front then reported the wrong conclusion
faithfully — which is how it surfaced at all.

The third thing is the one worth keeping. That defect took a purpose-built
fixture to notice, because **an answer that skipped a project reads exactly
like one that found nothing in it**, and the entrance kept only a cost
report. Both entrances now write a streamed `transcript.jsonl` beside their
chatlog, which needed `run_harness(stream=True)` in pyagag (`1db9150`):
streaming had been reachable only through `on_event`, the live-progress seam,
so a caller who wanted the record rather than the progress silently got a
dollar figure. Failure Farming only pays if the failure leaves a readable
trace.

## Open

Carried in [report4](report4.md) §8: proactive tidying (deliberately not
this phase), scan cost as projects accumulate, whether the entrance should
answer humans and agents differently, Front's one-reply-per-callback noise,
and the fact that an agent cannot correct its own channel's description —
the forge channel still advertised the `create-` prefix retired in p3, and
only the developer account could change it.
