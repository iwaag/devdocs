# fix6 step 4 — dispatch context and reminder wording

Plan: [plan.md](plan.md) step 4. Code: agdevworld `4888ea3` (CLI help,
docs, brief, wording), agautolab `15258d7` (wfrun guide).

## Dispatch and the child's brief

- `wfe delegate request` prints, as information: who accepts the child's
  result (`accepts: person` — "relay their own answer, not yours" — or
  `parent`), that the parent relays the child's questions and
  presentations to the person, and the node catalog the child will be
  given as it is now, with each template's expectation. It says the result
  is assessed against the snapshot the child actually receives. Nothing is
  required or refused because of it.
- The child's brief names the person the parent acts for, says the request
  is delivered exactly as the agent wrote it (the person's own words beside
  it), that what the child presents reaches that person through the
  requesting agent with its link, that a report of file paths to the agent
  is not a presentation to the person, and who accepts the result. The
  request itself is unchanged.
- Open corrections appear in the child's brief and continuation in full
  (why, what to add, how to add work, `--no-change`).

## Reminder wording

- The first line names every reminder code: `Not recorded — 2 reminders
  before completing build: presentation-missing, cleanup-unplanned.`
- `presentation-missing` states the facts WFE has: which nodes (and that a
  node was added by the run with files), the viewable files ("viewable
  files: …mp4, …png"), no presentation planned, and the catalog's
  expectation. The repair is one concrete command line.
- The "if the work only produced records that need no viewing, continue
  with --yes" sentence is gone, as is the self-check's own `--yes` line.
  The common explanation stays: nothing recorded, a reminder is not proof,
  repair and repeat or repeat with `--yes` for this one command.
- The last line is `In short: <code>: <omission> → <repair> | …` for every
  reminder, so `| tail -3` (and even `tail -1`) still says why to act and
  what to do. Each reminder carries `summary` in the structured detail.
- Delegated reminders end with e.g. `d1 (make/run-001) completed build with
  out/icon.png and presented nothing → wfe delegate revise use/run-001 d1
  --instruction "present … through a present node, then plan its cleanup"`.

Checked by `test/correction.test.ts` (three reminders at once: the last
three lines carry every code, a file name, each repair command and
`--yes`; the escape sentence is absent; the CLI's and HTTP's refusals end
with the delegated summary) and the updated fix5 tests.

## Help, documentation, views, guides

- `wfe delegate help show|take-up|dispose|revise|request`, `wfe run help
  correction`, `wfe present` (`keep`), `wfe conversation` (reply), the
  REMINDERS paragraph of `wfe run` help.
- `docs/delegation.md` (*Assessing a result*, *Correction*, records,
  states, operations, HTTP, error codes), `docs/runs.md` (reminder codes and
  roles, the tail line, *Corrections*, execution summary),
  `docs/presentations.md` (*Kept on request*).
- Run view: a delegation's card shows the latest result's assessment
  (each check with its status chip, facts, the frozen expectations,
  integrations, what could not be read), results returned for correction
  and "next move" while correcting; a child shows its corrections under
  *Related runs*.
- wfrun guide (agautolab): the tool line for `wfe delegate`; the
  reminders' "In short" line; the parent relays the child's page link and
  the person's own answer; a returned result is assessed where it is
  received, with run-001…004 / conduct/run-031…035 as the trial reference;
  `wfe delegate revise`; a child acts for the person its requester relays
  to; a returned result is added to as nodes.
- ponderer (`conduct.md`, `guide.md`) and Front (`converse.md`,
  `discuss.md`) were read for reporting instructions that contradict this:
  none (they say the child's result continues the run and is presented
  with `--kind result`). They were not changed, and no checklist was
  copied into them.
