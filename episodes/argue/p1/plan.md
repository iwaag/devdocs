# Argue p1 — from human desire to an actionable plan

## Goal and scope

Introduce a Zulip conversation where a human develops an initially vague desire with Front, archsage, domain sages, and other existing agents. Front facilitates; archsage designs the knowledge and research needed; sages answer from their assigned study trees. The session reaches its purpose when an actionable plan is recorded or the agreed project/workplan is established and linked back to the conversation.

This phase ends at planning and setup. It does not execute the resulting studies/development, wait for their results, refresh knowledge from those results, or bring them back into argue. User-initiated chains of later argue sessions are a future design. A useful plan alone is a valid outcome; creating a project is not mandatory.

This is a breaking redesign in a private experimental environment. Backward compatibility, legacy sage migration, production security hardening, and preservation of disposable experimental state are unnecessary. Choose package locations, schemas, storage, naming, and implementation order freely; replace obsolete paths instead of building compatibility layers. Keep credentials and machine-specific facts in ignored files. Favor agent judgment and useful tools over a scripted discussion sequence.

## Step 1 — Establish the conversation contract and shared participation

- Front can open an argue topic from an ordinary conversation, link the two, and invite the human to state their desire. Choose the channel/topic convention and publish it in the relevant introductions.
- Record the human's original desire by message ID and author. Front may help formulate it, but its own summary is not the human's submission. A human-posted statement or explicit human adoption of a draft is sufficient; use the existing identity/entry path rather than introducing a new authentication system. While it is missing, Front asks when the conversation resumes, without a timer or repeated idle model runs.
- Give Front the automatic conversation route. Other agents participate when explicitly addressed and answer in the same argue topic. Make this available to existing in-system agents, including cagent and Observer, without replacing their ordinary work roles with discussion behavior.
- Put the common participation guide and mechanics in `pyagag`: read the conversation, answer the question in service of the human's desire, and identify useful next steps. Add concise role context for Front, archsage, and each specialist; their capabilities and expertise remain distinct.
- Treat a mention as a deliberate request for a contribution. Do not automatically mention the previous speaker on every reply. Track outstanding requests by message ID and logical recipient so later posts, multiple recipients, and listener restarts do not erase unanswered invitations or routinely replay completed ones.
- Reuse the mirror and durable intake. Keep conversation identity and completion recoverable from Zulip; choose a small amount of metadata rather than a second workflow platform. Resolving an argue ends its discussion dispatch, without resolving the projects or plans it created.

Verify with focused tests: human desire versus an agent draft; ordinary human turns; two addressed participants followed by another post; restart with an unanswered invitation; and a specialist reply followed by Front without an automatic mention loop.

## Step 2 — Build archsage and consolidate sages

- Add archsage as one deployed agent with one Zulip account. Use a configurable frontier-model profile for research design and difficult synthesis; select the available backend at implementation time.
- Keep sages within its workspace, for example `sages/<name>/guide.md` and `sages/<name>/mainstudy/`. A shared execution profile plus each sage's domain guide and knowledge tree should be sufficient. Archsage can inspect all trees and verify a specialist's cited sources.
- Start by moving the existing arxiv knowledge-answering behavior into this structure. Retire the standalone arxivsage listener and obsolete deployment/registration when the replacement works; old bot identity and entrance compatibility are unnecessary.
- Provide direct specialist addressing through the archsage account, for example an actual archsage mention followed by `sage:arxiv`. Parse the recipient without a model call. A request for archsage invokes archsage; a request for a sage invokes only that sage. Publish the available names and addressing syntax so Front can discover and use them.
- Add the speaking role's header in the posting layer. Record logical speaker, triggering request, actual model/profile, and knowledge revision in run evidence. Dispatch between roles inside archsage directly when needed: its own Zulip posts are ignored by today's listener and cannot be relied on to wake another role on the same account.
- Scope each sage's supplied context and knowledge tools to its assigned tree. A folder or working-directory change alone is not access isolation. Use a simple bounded reader or existing harness controls where practical; no new container/OS isolation platform is required for this experiment. Document the actual boundary and keep archsage's broader access explicit.
- Let archsage define another sage and its study target with lightweight configuration and a guide. An empty knowledge tree is valid for a newly planned study: the sage should explain the gap, not invent findings. Test addition of a second domain without writing another listener or deploying another account.

Verify: direct sage invocation costs no archsage model run; two sages receive their respective knowledge context and retain distinct identities despite sharing a sender; archsage can inspect both; a missing source is reported honestly. Existing citations and knowledge-revision recording are useful starting points.

## Step 3 — Turn discussion into a plan or project setup

- Front calls archsage after the human desire is established. Archsage identifies useful existing knowledge, research questions, and missing study/sage domains. It chooses the substance from the conversation rather than a fixed arxiv/realworld checklist.
- Front facilitates follow-up questions and can call a known sage directly. Reuse archsage's recorded analysis; call it again when a new direction, missing domain, or conflicting evidence warrants its judgment. Avoid fixed round counts or an expensive archsage routing pass on every turn.
- Let Front propose concrete study topics, new projects, or workplans in existing projects. Once the human has chosen a direction, record the plan and create the agreed planning artifacts using existing tools and agent entrances. Reuse authorization already given in the conversation; do not add a confirmation round for every setup action.
- Keep the initial study/sage definition and later execution separate. A study definition or empty project can be established now; running research to populate it is outside p1. Inspect the target entrance before posting: in this system, some posts start work rather than merely record a plan.
- Finish with a visible outcome containing the original desire reference, chosen direction, concrete next work and intended owner/location, unresolved questions, and links to any created artifacts. Mark the session complete when that outcome is delivered. This means planning is complete, not that the human's ultimate desire has been achieved.

Verify: an open-ended desire becomes a useful plan; a knowledge gap becomes a study definition; an existing-project option becomes a linked workplan; plan-only completion works without starting downstream execution. The session can wait for human input without background spending.

## Step 4 — Deploy, demonstrate, and report

- Check the current environment through Nautobot or `pj-clusterintent/nctl`; use the ignored environment notes to locate services. Update affected dependency pins, introductions, launch configuration, and desired registrations together. A single archsage service/account owns its logical sages; avoid registering every sage as a separate deployed process.
- Run focused routing/role tests and relevant checks in changed packages. Demonstrate a bounded live conversation: human desire, archsage analysis, direct sage answer, another existing agent's contribution, and completion with a plan or project/workplan link. Use fixtures for the human-only input checks, or a real human submission; an implementation agent's test post is not evidence of human authorship.
- Include one restart with an outstanding invitation and demonstrate that a completed conversation stays quiet. Record model-run counts by logical role and enough API/run evidence to spot redundant calls; no benchmark campaign or arbitrary cost threshold is needed.
- Update developer guidance and ignored local environment notes. Write `report.md` with the delivered behavior, demonstration links, costs, and remaining limitations. Commit and push changed repositories and necessary submodule pointers.

## Implementation hints

- `pyagag/src/agag/listen.py`: `_consider`, `owed`, and `recover` use owner/mention routes, sender identity, and last-speech checks. The queue currently coalesces by conversation and route. Multi-party invitations and same-account sages need explicit treatment; adding a topic prefix alone is insufficient.
- `pyagag/src/agag/topics.py`: `serve_topic` defaults to `handoff=True`, which adds a mention to the reply. `handoff=False` is already available. `conversation_context` provides bounded prompt inclusion with full chat files alongside it; this helps avoid a run answering without reading the conversation.
- `pj-agdev/agfront/src/agfront/listener.py` and `zulip_listener.py`: ordinary callbacks follow `rootchat` back home. Argue participation answers in argue itself, while links to its opening conversation remain provenance. Front also has recovery hooks for self-opened conversations worth consulting.
- `arxivsage/src/arxivsage/listener.py`: existing knowledge citations, study-gap handling, profile selection, and `knowledge_revision` recording can be reused. `service/sync_knowledge.sh` is an explicit refresh today; automatic study-to-sage refresh is outside this phase.
- `pj-clusterintent/cagent/src/cagent_api/zulip_window.py` and `pj-agdev/agobserver/src/agobserver/listener.py` need participation adapters; not every current agent has a general mention handler. Shared discussion guidance can be reused without invoking their normal change/watch intake.
- `better_zulip_call/p1/report.md`: mirror reads and intake are reusable, but serving-time reads still include direct Zulip calls. Prefer available mirror context; it does not reduce the cost of sending long histories to models. Keep the original desire and decisions reachable when introducing summaries.
- Zulip mentions address users or groups, not arbitrary virtual-agent names. A real archsage mention plus a logical sage selector is sufficient for p1; independent autocomplete identities are optional.
