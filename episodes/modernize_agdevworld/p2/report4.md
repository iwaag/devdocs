# p2 step 5 — documentation and desired state

## `pj-agdev/.local/devenv.md`

The agfront section described the p1 route (`dispatch.md`, first line the
topic, `run-*`, "autolab answers inside the topic it was given"). Rewritten to
the create route: the guide's three branches, `create.md` posted verbatim under
a topic the listener derives (`front-<stem>` gen N → `create-<stem>-N`), forge
as the reader because `create-` is its prefix and it shares `#general`, and the
honest note that nothing comes back. Heading is now `(modernize_agdevworld p1,
p2)`. The launchd, credentials, subscription and `AGFRONT_ZULIP_LOG_ONLY` notes
were already correct and stand.

## `devdocs/README_DEV.md`

Unchanged. "agfront … Responds to any requests from Human and sends messages to
other agents" is still exactly what it does — only the set of agents it can
reach changed, and that belongs in the local env doc, not the agent roster.

## Desired state

`nctl drift` → **`converged=38 unknown=5`, drifting=0**, `agfront` converged.
No edit was needed: p1 declared Front's channels as `[front, general]` and p2
did not change them — the create route reuses the same subscription, which was
the point of choosing `#general` over `#FreeForge`. The 5 `unknown` rows are
the same agpc service-observation staleness p1's `report6.md` recorded, plus
`pj-voxel3dprint`'s stale workspace observation; unrelated and pre-existing.

## Rollout

agfront `bc1dbe7` was pushed to GitHub at step 4's start; the agstudio checkout
*is* the running one, so the listener restart was the whole rollout. The
`lighter_agag_listen` phase then added `1578487` (pyagag bump) on top and
restarted it again, so the running listener carries both. `pj-agdev`'s
submodule pointer is already at `1578487` via that phase's `80982da` — nothing
left to bump here.

## Phase state

All five steps are done. What p2 set out to prove:

- the simplified guide is the whole instruction, unchanged by the machinery
  (three lines, and step 4 gave no evidence for adding a fourth);
- `run-` is gone from Front;
- `create.md` reaches agforge and produces real work (Plane Work F2-10).

Still out of scope, unchanged from the plan: results flowing back into
`#front`, the agdevworld chat panel rewiring, `run-` in agautolab, and any
Front route to development work.
