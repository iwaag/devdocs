# Explicit reply p1 — phase report

Plan: [plan.md](plan.md); advice: [advice.md](advice.md); step reports:
[1](report1.md) [2](report2.md) [3](report3.md) [4](report4.md)
[5](report5.md). Executed 2026-09-20 by the Omni Agent.

## What changed

**pyagag** (`7452782` … `d20720c`), the shared listener and topic
lifecycle:

- `agag.serving`, `agag.delivery`: a serving is a journaled record
  (received → acked → executed → prepared → delivered; `failed`,
  `interrupted`) in `listener.sqlite`; the reply is on disk before it is
  sent; an ambiguous send is settled by read-back; delivery is retried with
  bounded backoff, never the run; exhausted retries stay visible as
  `failed` and are re-armed by the next post. One completion rule —
  unprocessed input past the last delivered serving's boundary — after a
  reply, before a resolve, at recovery and in `owed()`; the bot's own ack is
  never evidence.
- `agag.reply`: the `ag-reply` mark. Only marked text is posted; several
  marks are one post; code fences survive; missing/empty/unclosed marks are
  repaired once and otherwise posted as a visible failure; machine blocks
  and system notices are kept apart; the outcome is written beside the run
  identity. Described once after every conversational guide.
- Handoffs bound to requests: the requester from the processed input, an
  anchor in `Conversation`/root notes/`AGENTCHAT_HOME_ANCHOR`, the reply's
  destination located by id at delivery (renamed/resolved/gone handled out
  loud), an unreadable reply conversation kept pending, the served mark
  written by the listener after confirmed delivery and bound to the mention
  processed.
- `agag.continuation`: the carry-forward view (new input, the agent's
  `ag-continue` summary with newer posts overriding it, outstanding
  requests by state, an interrupted previous serving, what is omitted),
  refreshed for every serving including callbacks.

**Consumers**: Front (all four roles, callback by anchor, continuation
view), autolab (director, bmining), forge (plan front), cagent (front),
archsage (council and sages), Observer (argue participation), the shared
entrance and generated-agent template; every pin at `d20720c`, every
listener on this host restarted on it.

## Evidence

pyagag 683 tests; consumer suites green on their synced venvs; the
end-to-end fixture: 0 lost, 0 duplicated, one run per serving, 0.22 s
recovery. Fixtures reuse the real #7222/#7230/#7262 posts.

## Remaining gaps and pending live checks

See [report5.md](report5.md): three proposed live posts await
authorization; the `agautolab1` VM is not redeployed; the status file
carries no queue counts; `agentchat reply` is deferred as the advice
decided.
