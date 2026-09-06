# Step 1 — assembled panes

Mounted the existing conversation graph and chat panel in the four-pane shell
at `/?parts=shell`. `operationDashboard.ts` owns routine selection and the
selected fire message ID; chat consumes its detail payload without maintaining
another fetch timer. Routine selection changes all panes, and session selection
highlights the corresponding message-ID span in the fire conversation. Manual
activity is explicitly not an identified scheduled run.

Routine cards show the relay state, time since fire, next/overdue schedule
evidence and the overdue-ack annotation without changing the relay state.
The standing request is available in an expandable full-text disclosure.
Chat remains the existing Developer write door with its deliberate send button.
Selection changes disable the composer until destination status is available;
late reads cannot replace a more recent selection.

`nctl status --json` passed connectivity, authentication, catalog, GraphQL and
worker checks. `npm run build` passed (existing Phaser size advisory).
Inspected the assembled desktop screenshot of the real 28-conversation outlier:
all four panes, 29 graph cards including origin, and existing history render.
Evidence is ignored at agdevworld `.local/p5/step1/assembled.png`. No post made.
Refresh lifecycle and graph/mobile refinement follow in steps 2 and 3.
