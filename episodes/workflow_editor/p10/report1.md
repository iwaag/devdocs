# p10 step 1 — Delegation identity and decision ownership

## What was done

- **Contract**: `agdevworld/workflow_editor/docs/delegation.md` (new). It
  specifies the delegation between a parent run and a child run of another
  managed workflow:
  - **who owns what**: the parent's agent, the child's agent, the system and
    the person;
  - **the definition**: a portable target `project` + `workflow`, and
    `accepts: person | parent`, the scope for the child's result decision;
  - **identities**: the creation-qualified parent uid, the delegation
    `d<n>` with uid `<parent uid>~d<n>`, the child uid, and transport
    references that are not identities;
  - **one child per delegation, one open delegation per node**: a replacement
    is explicit new work with a `predecessor`;
  - **the records**: the parent's `delegations.<d>`, and the child's ordinary
    request input with `entrustedBy: {kind: delegation}`, with the original
    human input kept apart from the constructed request;
  - **dispatch resolution**: an explicit target workspace (an ambiguous one
    is refused), a target that accepts requests, and cycle checks over
    definitions and over the active ancestor chain;
  - **ten facts kept apart**: dispatch intent through child integration and
    parent completion;
  - **a decision-ownership table**;
  - **a 17-row transition table** (below);
  - **delivery and crash points**: binding by the delivered message id,
    never by text;
  - **limitations**.
- **Definition model** (agdevworld): delegate nodes can now carry `project`
  and `accepts`. The new fields are read and written in YAML, and when
  declared they enter the definition digest and the compiled node. Without
  them, the digest and the compiled graph are byte for byte what they were,
  so no approval or frozen run graph changes.
- **Validation**:
  - New checks: `delegate-project-unknown`, `delegate-accepts`, and a
    `delegate-no-intake` warning.
  - `delegate-target-unknown` and `delegate-self` now work per project.
  - `delegate-recursive` follows targets across the registry's projects when
    the context supplies them (`ValidationContext.project/projects`).
  - Workspace summaries pass the project id and each workflow's intake.
- **Execution guards unchanged**: `delegate-unsupported` still refuses every
  run that has a delegate node, at preflight and in the reducer, as the plan
  says. Step 3 lifts them together with the warning.

## Completion evidence: the transition/ownership table

`docs/delegation.md` *Transitions* covers:

- normal delivery (rows 1–2, 17 for a failed or uncertain delivery);
- questions (3);
- answers, and stale answers (4–5);
- forge's plan agreement (its `_plan` question is row 3; the answer, row 4,
  is the child's agreement under its existing rule);
- forge's result confirmation `confirm` with `confirms: result`, and its
  acceptance or rejection (6–8);
- results, and the parent's receipt and disposition (9–11);
- failure (12), holds of the child and of the parent (13, 15), and
  cancellation of either (14, 16).

*Decision ownership* says:

- which answers are within the parent agent's scope;
- that result acceptance needs `accepts: parent`, or the parent's person's
  taken-up answer as `because`;
- that the child also refuses an out-of-scope acceptance (`delegate-scope`);
- that a `satisfied` disposition is neither the child's acceptance nor its
  integration, nor the parent's own result decision.

## Checks

- `test/delegation.test.ts` (4 tests):
  - the YAML round trip and the compiled node;
  - registry validation (unknown project or workflow, accepts, no-intake,
    no registry, self);
  - recursion across projects, direct and through a third workflow;
  - the misplaced-field warning, and unchanged digests without the fields.
- Full suite: 314 of 315 passed, and the type check is clean.
  - The one failure is `sqlite.test.ts` "notify store: a short write lock…",
    a timing bound under suite load. It passes 3/3 alone and also passes on
    the tree without these changes.

## Decisions taken here

- **`accepts` defaults to `person`.** Consuming a child's artifact never
  authorizes the child's integration by itself. A workflow author who wants
  the parent agent to accept says `accepts: parent`.
- **Transport is the child workflow's ordinary Zulip intake.** A delegation
  post binds by the message id that the notifier's journaled delivery
  recorded. The request text the child receives is the parent's
  `request.md`, checked by hash, not the post's wording.
- **The parent agent answers through `wfe delegate …`.** It does not read
  Zulip. People see the same exchange in the child's request topic.

## Next

Step 2: the dispatch, bind, relay and answer operations through shared
server logic, the CLI and the API; intake recognition of delegation posts;
notifier conversations for delegations.
