# agent_standardize p4 plan — agfront learns autolab, up to workplan

Goal: repeat p2's proof with the second, more complex agent. agfront, asked
by the developer in a `front-*` topic, understands from autolab's intro how
to request development work and opens a `workplan-…` topic on the
developer's behalf. **Planning only — nothing may fire a `workrun-`.**

Success criteria:

1. autolab is standardized the way forge was in p1: instance-named Zulip
   bot, its own channel as entrance (all topics swept, placeholder reply),
   and an `intro-autolab-agstudio1` topic in `#agents`.
2. A `front-*` conversation: developer asks for development work in a named
   project → Front proposes, is permitted, opens a `workplan-…` topic in
   that project's `pj-<slug>` channel → autolab's planning flow answers
   there (a mission plan appears).
3. Attributability, same grep discipline as p2/p3: `autolab`,
   `workplan-`, and the channel names appear nowhere in `agfront/src` or
   its guide. Front's knowledge arrives only via the harvested intro.
4. No `workrun-` execution: creating a `workrun-` topic as a planning
   artifact is fine (a topic nobody posts into never fires); *triggering*
   one is out of scope.

Decisions already made ([braindump.md](braindump.md) + episode decisions):

- Instance name `autolab-agstudio1` — the placement that runs the Zulip
  listener. The agautolab1 node has no listener and is **not** instanced
  this phase.
- `workplan-` topics stay in `pj-<slug>` project channels — that is
  autolab's existing contract (project identity comes from the channel) and
  it is not changed here. The instance channel is for questions and
  redirects only; the intro must teach exactly this two-channel shape.
  This is also the episode's first proof that an entrance can *redirect*
  ("I opened workplan-… in pj-<slug>") rather than host the work itself.
- **Plane identity split is deferred.** `.local/plane/autolab.env` is shared
  with the agautolab1 deployment; renaming its display name would mislabel
  that node's attribution. Zulip bot rename only, Plane untouched, deferred
  note in the report.
- Test-only realm still applies: anything left from before may be deleted.

Constraints (deliberately minimal):

1. Secrets in `.local/`; pyagag changes go commit → push →
   `uv lock --upgrade-package pyagag` in consumers (localrule.md).
2. agautolab changes: push to GitHub; redeploying agautolab1 is optional
   this phase (its gateway does not touch any of this), `--limit agstudio`
   keeps the live checkout honest if run.
3. Zero agfront behavior change (criterion 3 enforces it).

## Step 1 — Standardize autolab (the p1 pattern, second copy)

- Rename the agstudio autolab bot to `autolab-agstudio1` (admin
  `developer.env`, PATCH by user id; ids survive renames).
- Identity seed: `.local/instance.toml` + committed example, one `name`
  key, same shape as forge's.
- Own channel `autolab-agstudio1`: create, subscribe bot + Omni +
  Developer, place in the agents folder. Extend the listener's sweep to
  "everything in my own channel, existing prefixes elsewhere" — pyagag's
  callable `topic_filter` already exists (p1 step 3), so this is the same
  two-line filter forge has, plus a placeholder entrance reply pointing at
  the workplan contract.
- Intro machinery: this is the **second copy** of forge's intro command. 
  Preferred: lift the shared part (post file + date/revision stamp to
  `#agents`, `intro-<instance>` topic) into pyagag and have both agents
  call it; acceptable: copy the ~60-line module and note the duplication.
  Implementer's call — but if lifted, forge should switch to the shared
  helper in the same change, not "later".

## Step 2 — Write autolab's intro as the contract

`params/intro.md` (or autolab's equivalent home) must let a reader who has
never seen autolab succeed: what autolab does; ask questions in
`autolab-agstudio1`; to request development work, open a `workplan-…`
topic **in the project's `pj-<slug>` channel**; how to know/choose a
project (list-or-ask guidance); planning vs. execution (`workrun-`) in one
line. Front is the first consumer — write for it, not for humans. Post it.

## Step 3 — Ensure one live test project

The p3 wipe declared everything deletable, so a live `pj-<slug>` channel
autolab is subscribed to may not exist. Check; if none, provision a small
throwaway via autolab's own `project_init` path (Zulip channel + repos +
Plane project). Doing that by hand as the Omni Agent is a Deus Ex Machina —
leave the one-line note. Verify the listener actually sweeps the channel
(subscription is the routing decision; `project_init` handles it).

## Step 4 — Prove and report

1. Fresh `front-*` topic: developer asks for a small planning task in the
   test project by name. Expect a grounded proposal citing the workplan
   contract (from the harvest), permission, then the `workplan-…` topic in
   `pj-<slug>` under Front's identity, and autolab's plan reply there.
2. Confirm nothing posted into any `workrun-` topic (criterion 4) — quote
   the topic listing.
3. Run the criterion-3 grep and quote it.
4. `report.md`: message links for all three channels (front-*, pj-<slug>,
   #agents), the greps, commits, and deferred items — Plane identity split,
   agautolab1 instancing, `workrun-` execution via front, entrance reply
   beyond placeholder, intro-machinery dedup if skipped. Update
   `agautolab` docs and `.local/devenv.md` (bot name, new channel).
