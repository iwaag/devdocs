# Workflow editor fix3 — one way to ask, and catalog nodes as far as possible

## Evidence

Discussion `20261022-201638-8q25lf` ("image subtract", 2026-10-22).

- `gimmickimg/freework/run-004` was delegated by `ponderer/conduct/run-020`
  (d1). It removed the mushrooms with LaMa and asked ponderer-agent whether
  the result was enough (child q1).
- ponderer took the question up on its `delegate-try` node and asked the
  person with `wfe run ask` (q1 to iwaag). That recorded a WFE question and
  notified the run topic, but posted nothing in the discussion.
  - `wfe discussion show` and `state` said "Waiting for the person:
    nothing".
  - The API's `status.awaiting` was `[]`.
  - The run itself was `awaiting-person`. Each side waited for the other.
- In a discussion there were three ways to ask:
  - `wfe run ask` alone (not posted);
  - for a participant, `wfe run ask` plus `wfe discussion contribute
    --kind question` (posted only if the second command was run);
  - for the conductor, `wfe discussion talk` (posted, but only on a talk
    node).

  The guide's "a question to the requester is `talk`" did not fit a
  question that arrived on a delegate node.
- Separately, all four gimmickimg freework runs (001–004) did all their
  work inside freework's single talk node. That included an agreed
  implementation and a presentation, although the catalog's `present`
  template was in the prompt. The catalog's wording ("they generally expect
  … choose whether and where") read as optional. ponderer's nodes are fine
  grained only because every discussion command needs `--node`.

## Decisions (Developer, 2026-10-22)

1. Adopt one way to ask: a question is `wfe run ask`. WFE posts the open
   question of any run that speaks for a discussion. `discussion talk` and
   `contribute --kind question` go away.
2. Catalog activities are performed as nodes of their own "as far as
   possible". This is not an absolute rule, because that would close off
   the flexibility of a non-deterministic process.
3. Rescue run-020's question.

## Changes

- WFE (agdevworld):
  - `DiscussionStore.postQuestions`, called after `wfe run ask` and in the
    executor's reconcile. On the conductor's talk node, the question is the
    talk activity.
  - `status.awaiting` names whom the run asked.
  - The CLI refuses `talk` and question contributions.
  - Docs: discussions.md *Questions*, contract.md, catalog.md.
- Catalog meaning (`shared/catalog.ts`, `wfe run nodes` help) and the wfrun
  guide: "as far as possible, not inside another node such as a talk; not
  an absolute rule".
- ponderer guide: a question is `wfe run ask` on whichever node waits.
