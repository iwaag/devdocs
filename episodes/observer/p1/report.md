# observer p1 — report

## What was asked, and what exists

The braindump wanted an agent that does the non-deterministic, lightweight,
long waiting: hold a condition written in ordinary words, check it cheaply and
often on a **local** model, and write to Zulip when it holds. It was undecided
between a command in each agent's own working topic and a dedicated channel
with a topic per watch.

**`agobserver-agstudio1` exists and does that**, on the dedicated-channel
shape. One topic in its own channel is one watch — a condition in words, what
to look at, where to notify. It looks every 60 s with this host's ollama
(`qwen3.8:27b-mxfp8` through `agcode`), posts **once** into the conversation
the requester named, and finishes. A ✔ on the watch topic cancels it. The
whole phase cost **zero paid tokens**.

Proved live on 2026-09-13: a file watch with an explicit completion signal, a
natural-language conversation judgment that stayed quiet through an unrelated
progress post and fired on an unanswered request, and a watch **requested by
Front**, whose notification resumed Front's own conversation through Front's
ordinary listener.

Reports per step: [1](report1.md) (the agent and the request contract),
[2](report2.md) (waiting as short evaluations), [3](report3.md) (delivering
once), [4](report4.md) (deployment, demonstrations, numbers).

## The decisions that carry it

**The channel shape won, and the braindump's hesitation was right to
hesitate.** A watch in each agent's own working topic would have made the
request a command in a conversation that belongs to somebody else's work; a
watch in Observer's channel is a request like every other agent's, and it is
the requester who says where the answer goes. What was lost is convenience —
you must name a destination — and that turned out to be the field the local
model got wrong, which is a good trade for having noticed.

**A watch is a message id.** `[selfnote][watch]`'s own id *is* the request
(`w6676`), with `[accepted]` and `[state]` beside it and one visible post
saying the same in words. Nothing reads a topic name, so a rename, a ✔ or a
reused name never redirects one watch's record into another's — the rule the
rest of the realm arrived at the hard way, adopted here before it could cost
anything.

> **Correction (`p1 ex1`, 2026-09-13).** The rule was adopted for the record
> and not for everything that used it, and the paragraph above overstated
> what p1 shipped. Three places still read a name: the accepted destination
> was stored as the requester's text and re-resolved by name at send time, so
> a renamed destination whose name had been reused was notified *by its
> impostor*; cancellation compared cached topic names, so renaming a watch
> topic before resolving it hid the ✔; and `reconcile()` skipped records whose
> state already matched, which made the stale name permanent. Fixed in
> `ex1/` — see `ex1/report1.md`, `ex1/report2.md` and the live evidence
> `w6745` / `w6756` in `ex1/report4.md`.

> **Correction (`p1 ex1`).** p1 also treated *uncertainty* as an answer where
> it mattered most: a destination that could not be **read** produced a
> terminal `undeliverable`, and the read-back that guards against an ambiguous
> send returned the same value for "no delivery there" and "I could not look".
> A met watch could therefore lose its one notification to a timeout, or
> produce two. Delivery now stops only on Zulip's own "gone or ✔"; see
> `ex1/report3.md`.

**Three answers, not two.** `met`, `not_met`, and `unable` — a statement about
the *look*, not about the world. A target that cannot be read is never
reported as a condition that has not held yet. Every failure path lands there:
a non-zero exit, the deadline, a verdict that cannot be parsed. This is the
single distinction the design turns on.

**The acknowledgement names nobody.** The requester is waiting for the
notification; naming them in the acceptance would buy them a paid run to read
"understood". Measured: Front's listener served that conversation exactly
twice — the request and the notification — with nothing in between.

**Routine polling is silent.** Nineteen looks produced two posts in watch
topics: one delivery-finished line, and one "I have not been able to look at
this three times in a row". A watch that reports every look is a watch nobody
leaves running.

**Delivery is once, and the reasons are ordered.** The store's delivery record
is the fact; the watch id inside the post is what a read-back recognizes after
an ambiguous send; the state note is written *after* delivery, so a crash
between them leaves the watch owed rather than silently finished. The
remaining crash window is named in the module rather than engineered away.

**Losing the local store loses memory, never work.** The request is the Zulip
topic. Deleting `.local/watches/` entirely and reconciling recovered the
active watch and skipped the cancelled one, with no fresh post involved.

## What a local model turned out to be good and bad at

Good, across 19 looks with **no incorrect judgment**: honouring the part of a
condition that says *how to tell* (a `.part` file present is not a completed
download, five times running); declining to guess through a permission error;
distinguishing an unanswered request aimed at a named recipient from progress
reports and from a question already answered — quoting the message id, the
sender and the recipient in its evidence.

Bad, once, and instructively: asked to read a request with no destination in
it, intake **filled the field from the nearest phrase** ("Tell me") rather
than reporting the gap. Small models complete forms; they do not naturally
refuse them. The deterministic parser caught it. The lesson generalizes: give
the model the judgment and keep the *contract* — where a post goes, who is
named, what the reply says — in code.

The other half of the braindump's warning held too. An evaluation is given the
accepted request, the previous observation, its own tools and the observe
guide, and nothing else — no chatlog of its own topic, no other watch, no
development guide. Median look: **15.2 s**.

## Where it is

- `pj-agdev/agobserver` — a plain directory in `pj-agdev`, not a submodule, so
  it has no repository of its own. That is the one decision most likely to be
  revisited; it was taken to avoid needing a GitHub repository created by hand
  before the first commit, and reversing it costs a `git mv` and a submodule
  entry.
- launchd `com.agdev.agobserver-zulip`, `AGOBSERVER_INTERVAL_SECONDS=60`.
- Nautobot: service, placement, `desired_workspace pj-agdev`, agent (zulip
  user 23). `nctl drift` converged=46, error 0.
- 36 tests. `devdocs/README_DEV.md` has the cross-project section; the ignored
  `pj-agdev/.local/devenv.md` and `pj-clusterintent/.local/localenv_memo.md`
  have the machine half.

## What p1 deliberately did not do

Topic-local command shortcuts, recurring watches, editing a condition,
adaptive or event-driven scheduling, distributed observation, a dashboard, and
replacing the ComfyUI notifier — all deferred by the plan and all still
deferred. Interrupting an evaluation already in flight is also still out: a ✔
is honoured before the next look and re-checked before notifying, and an
in-flight look is allowed to finish.

## What the next phase should look at first

1. **Sequential evaluation is a ceiling.** N watches make the effective
   interval N × ~15 s; at 60 s that is about four before a watch is looked at
   less often than it was promised, and nothing warns about it.
2. **Nothing prunes `.local/topics/`** — one workspace per look, ~1400 a day
   per long-lived watch.
3. **Observer registers no mention route**, and that is the only reason Front
   naming it in a reply does not start a loop. Anyone adding `on_mention`
   should read report 4 first.
4. **A very long previous-observation chain is unmeasured.** The longest watch
   here lived minutes.
5. **`unable` forever has no ceiling.** A permanently unreadable target is
   reported once at the third failure and then watched silently for as long as
   the requester leaves it open.

*Deus Ex Machina note: the Omni Agent built agobserver end to end — an agent
an in-system agent could in principle have been asked to build. Handoff
candidate.*
