# failsafe p6 ex2 — step 3: verify and finish (in progress: live trial not run)

## Regressions

- **pyagag** `tests/test_failsafe_p6ex2.py`, 15 tests:
  - the statement and its absence;
  - the chatlog label;
  - proxy acceptance of the Developer's mission: plain, reserved, and in
    person;
  - no acceptance without the statement;
  - no acceptance for an ordinary agent or for the worker;
  - hold release by the proxy, by the principal, and in person, with an
    ordinary agent refused;
  - a hold on the proxy's own words;
  - the proxy's cancellation, which names the actual speaker;
  - a question to the Developer answered and withdrawn by the proxy, and
    not by an ordinary agent.
- **Full suites on pyagag `df0b635`**:
  - pyagag 1123;
  - agfront 193;
  - agautolab 331;
  - agforge 265;
  - agobserver 177;
  - agentroom (relay) 363;
  - archsage 39;
  - cagent 204.

## Environment and deployment

- **Before deployment:** `nctl drift` converged=46, error 0.
- **pyagag `df0b635`** is locked, synced and committed in:
  - agfront `3507e4a` (guides `12ce0f1`);
  - agautolab `b9dedad`;
  - agforge `9e1eacd`;
  - agdevworld `48672c2`;
  - agobserver (in pj-agdev `57d6a25`);
  - archsage;
  - pj-clusterintent (cagent).
- **Restarts at 07:17:59Z**, each with `launchctl kickstart -k`:
  - every listener;
  - autolab's gateway;
  - forge's service;
  - the relay;
  - cagent-api and cagent-zulip.

  All came back running. Startup recovery was the same as at ex1's 05:51Z
  restart (the same 50 old ✔ mentions ignored), with nothing new queued.
- **Not redeployed:**
  - comfynotify keeps its old pin; it makes no decision checks;
  - agautolab1 (VM) runs only its HTTP gateway, as in ex1.
- **Host statement:** `~/.config/agag/people.toml` is in place for every
  process on the host.

## Live L3 reproduction: not run

- The Omni Agent posted nothing to `#front` for this trial.
- The request post as the Omni Agent, on its own credential, was refused
  by the Omni Agent's session classifier (*Self-Modification*).
- Posting it needs the Developer's permission for this session, the same as
  the step-1 write.
- The planned trial is `#front › front-desk-20260928-p6ex2-proxy`:
  1. The Omni Agent requests one mission of one task in pj-robustp1, not
     started, and a hold on its start on its own post.
  2. It then releases the hold and scraps the plan.
  3. Front records `cancelled` on the Omni Agent's post.
- **Expected:**
  - no request for the Developer's personal confirmation;
  - the hold and the disposition name `by 9 (Omni Agent)`;
  - no credential switch and no human nudge.
