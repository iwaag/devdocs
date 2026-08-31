# agent_notifier — Step 4 report

Completed 2026-08-31; the requested short-job two-serving criterion did not
pass, and this report records the result rather than hiding it.

- The autolab and agforge guides now tell a receiving run to ticket a
  minutes-long generation, record the pending action, and finish.
- A real throwaway autolab mission (M-44 / M-45) submitted a 640×640 SDXL
  job and created its `comfynotify watch` ticket without polling. The first
  supercoder serving lasted 231.0 s and cost $0.4280606.
- The still itself finished in 5 s. The notifier posted the success record
  while that first serving was still writing its report. Its later completion
  reply then resolved the topic, so the notifier post was not the last real
  post and no second serving occurred. The transcript and first-run report
  show that the run did not poll after ticketing.

This is the expected short-job race: the notifier is correct, but a job that
finishes before the agent exits cannot be a callback. Step 5 uses a
minutes-long clip, where the first run can end before terminal state, to
verify the intended two-serving path.
