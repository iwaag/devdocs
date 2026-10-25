# workflow_editor p18 — final report: one conductor run per conversation

Plan: [plan.md](plan.md). Step reports: [1](report1.md) representation,
[2](report2.md) continuation and graph growth, [3](report3.md) recipient
boundaries, [4](report4.md) aghangout, [5](report5.md) transition,
[6](report6.md) deployment and trial.

## Outcome

A facilitated conversation no longer creates a ponderer run for each
response. The room has one **standing conductor**:

- When its nodes are done, the run **rests** (`idle`). At rest it holds no
  job, slot or polling turn, and it reads "Idle · Ready for the
  conversation's next message", or 待機中 in aghangout.
- The room's next words are delivered to the same run, which is woken with
  one `ponder-w<n>` node.
- Facilitation end/start and close/reopen move the same run through
  `open`, `suspended` and `closed`.
- A cancelled conductor is replaced only explicitly, and the replacement
  is linked to it.

**Recipient projects still run their own commissions.**

- A new commission is a new child run.
- Clarifications go to that child.
- Changes go back to the same child, including after the conductor has
  accepted a result: the person's words reopen it.
- A child completes on its own. Rest or closing infers nothing about it.

**Both levels are clear in the deployed UI.**

- aghangout's 「進行のグラフ」 shows the one run as growing stretches.
  Commissions are links to their own runs, messages and steps link to each
  other, and finished stretches fold.
- WFE run views show the idle focus card and status.
- agobserver reads a resting conductor as `idle-input`.

**Existing rooms were transitioned honestly.** One recorded decision per
room: 24 latest conductor runs were established, and 5 retired-argue rooms
were recorded as unreadable. Earlier runs are kept as history, and nothing
was executed or replayed.

## Deployed state

| Component | Release |
| --- | --- |
| WFE (agdevworld) | `9fd7a3f` (pre-p18 `ae7d050`), conversation store v13 |
| agautolab (wfrun guide, executor pin) | `eeddf95` |
| pj-ponderer (conduct guide and workflow) | `5c6a7ca` |
| aghangout | `f7b56c9` |
| agobserver (pj-agdev) | `9e9e767` |
| agpc worker | wfe `9fd7a3f` / executor `eeddf95` (nctl drift converged) |

The recovery point is in ignored notes (`pj-agdev/.local/p18/notes.md`):
backup paths, code and data rollback commands, and the pre-transition
conductor records. A pre-p18 release cannot read store v13 or replay
`standing.*`, so a rollback restores code and data together and accounts
for messages accepted since.

## Identities

- Trial: room `p18-trial` (check actor `p18-check`) → ponderer
  `conduct/run-045` → misc `freework/run-010`.
  - Messages #384–#393.
  - Wakes `ponder-w1`…`w3`, the first under job receipt
    `wake:ponderer/conduct/run-045/w1`.
  - Correction `c1` on the child. Its delegation `d1` was reopened by the
    person's words, and its `satisfied` disposition is kept.
- Transition example: `20261024-160217-forusr` is now continued by
  `run-036`. `run-031`…`run-035` remain as its earlier history.

## Validation

- WFE: 622 tests and both builds pass. New suites cover standing runs,
  their flow through the executor, recipient boundaries, the transition
  and the run-view activity line.
- aghangout: 33 unit tests. The browser checks pass in Chromium and
  WebKit, including a new progression section of 15 checks.
- agobserver: 38 tests.
- The transition was rehearsed on a copy of live data; the live result was
  identical.
- The deployed trial covered several real ponderer turns, a commission and
  its correction, idle intervals, close/reopen, a browser check at two
  widths, the run views and observation (details in report6).

## Remaining limitations

- **`run.json` grows with every turn.** Each wake writes a revision, and
  the record keeps every revision's compiled graph: about 780 KB at 60
  turns and 2.6 MB at 120. Reading stays fast (linear replay), but the file
  size is a known limit for very long rooms.
- **Topics are optional.** Topic labels are the agent's choice. In the
  trial, ponderer labelled only the first stretch, so later stretches read
  「第N段」.
- **Live trial coverage.** Human review was not exercised in the live
  trial. It is covered by the tests, and the p17 suites still pass.
  Concurrency and restart behaviour were likewise verified in tests rather
  than provoked live.
- **Retired rooms.** The 5 retired-argue rooms have no readable conductor.
  If someone speaks there, a new standing run starts.
- **Decision text language.** The decision and outcome texts inside steps
  are the agent's own words, usually English. The surrounding product
  labels are Japanese.

## Where to look

- Example: aghangout `?conv=p18-trial` (under 動作確認; the full address is
  in the ignored notes).
- Contracts: `workflow_editor/docs/runs.md` *Standing runs*,
  `docs/conversations.md` *Progression* and *Transition*,
  `docs/delegation.md` *Correction*.
- Development reference: `devdocs/README_DEV.md` *One conductor run per
  conversation*.
