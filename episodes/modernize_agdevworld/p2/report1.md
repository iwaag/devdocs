# p2 step 1 — the guide, committed as the phase's source of truth

`agfront/agent/guides/front/guide.md` went from 24 lines to 3, and that
version is now committed (`159b92b`). Nothing was added to it.

## What it says now

| The last message is | Front does |
|---|---|
| casual chat, or a request a text answer satisfies | just reply |
| a request for image, video, music or speech audio | describe it in `create.md` |
| coding, tool use, deep research | politely say it can't |

## Why nothing was added

- The media list matches agforge's four toolsets
  (`toolset-{image,video,music,speech}.md`) one for one, so Front's branch and
  forge's actual capability already agree. A translation table would only be a
  second place to keep in sync.
- No channel, no topic name, no dispatch mechanics. Those became the handler's
  decision in step 2 — the agent writes content, the machinery routes it.
- The plan's optional line ("say the answer appears elsewhere") was **not**
  added. It is deferred to step 4: if the live runs show Front promising a
  reply into `#front` that it cannot deliver, that is evidence and the line
  goes in; otherwise it would be Anxiety-Driven Guidance and the guide stays
  at three lines.

## Removed with it

The `run-<something unique>` row, the p1 dispatch-table framing, and the
"Nothing comes back to this conversation" paragraph. Front no longer has any
route to autolab's `run-` topics. The `run-` handler itself stays in
agautolab — the Omni Agent still fires it by hand — as planned.
