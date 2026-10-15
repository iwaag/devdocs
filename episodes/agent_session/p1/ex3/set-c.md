# Set C — local save and load of a match

Order: fresh first, then run. Both arms started from `7669fdb`. Evidence:

- `pj-agdev/.local/agent-session-p1-ex3/step3/C-fresh/` and `C-run/`;
- `step4/C-fresh/`, `step4/C-run/` and `step4/corrupt-*.txt`.

The branches are `trial/agent-session-p1-ex3/c-fresh` (`553afcf`) and
`c-run` (`62514f9`).

| | fresh | run |
| --- | --- | --- |
| Reported cost | **$5.57** | **$6.08** (1.09×) |
| Attempts / steps | 5 / 196 | 5 / 140 (one session `cac7b8d6`) |
| Attempt wall time | 1,487 s | 1,196 s |
| Plan / implement / follow-up / repair / close-out | 0.55 / 1.64 / 1.47 / 1.41 / 0.50 | 0.56 / 1.76 / 1.67 / 1.39 / 0.70 |
| Context at attempt starts | 44 k each | 44 → 95 → 162 → 197 → 221 k |
| History reads after the plan | 4 (44 k chars) | 0 |
| Outcome | accepted after one repair | accepted after one repair |

**Designs.** The two designs differ:

- **fresh**:
  - `serializeGame` / `deserializeGame` with field-by-field validation of
    players, entities, orders and queues;
  - the bot's closure state exposed through `getInternal` /
    `setInternal`;
  - Save/Load in a new HUD row (the HUD grew by 30 px).
- **run**:
  - `serialize` / `deserialize` of the raw state;
  - the bot's threshold and attackers moved onto `bot.state`, so the whole
    state is saved;
  - Save/Load at the top right of the field, with a `storage` listener.

**Defects at first evaluation** (one repair each):

- **fresh**: a save taken while a worker was helping an unfinished
  building could not be loaded. `cmdResume`'s build order has no x/y, and
  the validator required them. The save was shown as "保存データなし", and
  the earlier good save had already been overwritten.
- **run**: partly corrupted saves were accepted. An unknown kind in a
  production queue crashed the loop after load, and a missing ore value
  produced NaN.

**Quality.** After repair, both passed every C criterion:

- a round trip of a 4-minute two-bot match with research and a queue in
  progress;
- 90 s of continued simulation identical to the original;
- `byId` rebuilt and bot state restored;
- five corrupt inputs rejected with the live match kept;
- the browser reload restore exact;
- the saved time shown and Load disabled without a valid slot.

**Reading.** This is the largest task, and here the retained arm cost 9 %
more:

- Its later attempts ran on prefixes of 162–221 k. That is 22.3 M cache
  read, against 15.7 M for fresh.
- It needed 56 fewer steps.

The difference is within what the two designs and repairs alone could
cause.
