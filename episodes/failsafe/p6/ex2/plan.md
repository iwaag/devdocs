# Failsafe p6 ex2 — recognize the Developer's full proxy consistently

## Decision and scope

The Omni Agent has the Developer's full delegated authority. Its instructions,
approvals, cancellations, and hold releases have the same authority as the
Developer's; requesting another personal confirmation is unnecessary.

Keep identities distinct: the Omni Agent posts as **Omni Agent**, the human as
**Developer**. Record the actual speaker, delegated authority, and recorder
where relevant. A Developer-account post alone does not prove human input.

This is a small fix in a private experimental environment. Reuse existing
identity/configuration mechanisms; no general permission framework or backward
compatibility work is needed. Implementers choose the smallest coherent change.

## step1 — align the rule and the affected checks

- Read `../ex1/report.md` and `report5.md`: L3 rejected the same proxy
  confirmation accepted in L1/L2. Locate Front's harness memory
  `feedback_confirm_before_contacting_agents` and replace its conflicting rule.
- State the full-proxy relationship in one shared authoritative configuration
  or existing identity definition, with consistent agent guidance. Check the
  decision-holder checks for acceptance, dispositions, holds, and release;
  update those that wrongly exclude the proxy from Developer-owned decisions.
- Preserve separate sender/requester IDs and reply routing. Equal authority
  does not require rewriting authorship or aliasing every identity comparison.

## step2 — use the proxy's own posting identity

- Make operator/Omni posting instructions and relevant defaults use the Omni
  account. Keep human UI posts under the Developer account. Resolve authority
  refusals through the shared rule rather than switching credentials.
- Retain checks that acceptance refers to the result actually reviewed and
  that ordinary worker agents cannot approve their own work as the Developer.
- Correct ex1's "Developer in person" wording if only the account is evidenced;
  leave the original conversation history intact.

## step3 — verify and finish

- Add focused regressions for proxy approval of a Developer-owned decision,
  cancellation/release where affected, and an ordinary worker's refusal.
- Check the environment through Nautobot or `nctl` before deployment. Update
  only affected pins/services and run one short Front exchange reproducing L3:
  the Omni account's confirmation is accepted, subsequent decisions work, and
  records name the actual actors. No credential switch or human nudge.
- Write a short `report.md` with changed rules, checks, live evidence, and any
  remaining limitation. Update relevant guidance and commit/push owning repos.
