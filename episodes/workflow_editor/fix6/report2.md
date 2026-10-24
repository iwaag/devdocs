# fix6 step 2 — the assessment where the parent receives and reports

Plan: [plan.md](plan.md) step 2. Code: agdevworld `4888ea3`.

## Where one assessment is used

| Place | How |
| --- | --- |
| `wfe delegate show` | the latest result's assessment under the delegation (checks, facts, expectations, repairs, integrations, "now", unavailable) |
| Take-up of a result | printed once with the take-up, followed by the parent's next moves |
| The parent's next attempt | brief and continuation, beside each result not taken up yet (`ExecStore.assessedLines`) — not repeated once taken up |
| `wfe delegate dispose --disposition satisfied` | `DelegateStore.dispose`: under the parent's lock, the result (latest, or the one named, with `--manifest`) is assessed; reminders refuse before the disposition is recorded |
| The parent's outward report | `wfe conversation reply|contribute --kind result|synthesis [--completing]`: `adviseRun` with `assess` |
| The completion that completes the parent | `wfe run complete`, `adjust --completing`, HTTP `ops`/`adjust`: fix5's advice with `assess` |
| HTTP | `POST …/delegations/<d>/dispose|revise` (`yes`), `GET …/delegations/<d>/assessment`, `GET …/delegations` with each latest result's assessment |
| Run view | the delegation card shows the assessment and corrections; the child's *Related runs* shows a correction |

At report time every delegation's latest result is assessed: open, and
judged satisfied (an earlier `--yes` at disposition is no exemption);
rejected, abandoned and orphaned ones are not. A corrected result is a new
event and is assessed anew, so the old warning disappears.

## One response, no partial effects

- Reminders are `delegated-presentation-missing`,
  `delegated-review-unrecorded`, `delegated-presentation-serving`, each
  naming the delegation, child run, result event and manifest digest. They
  merge with the parent's own reminders (the dedupe key is now code + refs,
  so two delegations are two reminders).
- Disposition: assessed and recorded inside one lock; a refusal records
  nothing. A retransmission (same receipt) is not assessed again.
- Run completion: evaluated by `adviseOrRefuse` inside the operation's lock
  before the write (fix5's path).
- Conversation posts: evaluated under the run lock before the post (fix5's
  path); a post that also completes a node is one refusal.
- `--yes` (HTTP `yes`) passes reminders for that command only.
  `result-superseded`, `result-returned`, `delegation-closed`, attempt
  ownership stay errors.
- Remote workers forward `wfe delegate …` and `wfe conversation …` with
  their flags (`cli/remote.ts`); no separate path was needed. HTTP clients
  get 409 with `{code: "reminder", reminders, assessment}`. No user-facing
  client performs dispositions today; the run view only displays.

## Repairs offered

Inspect the child's evidence (`wfe delegate show|file`); return the result
to the same child (`wfe delegate revise`, step 3); relay the person's actual
answer (`wfe run answer … --from <person> --source-message …`, then
`--because`); ask them (`wfe run ask … --to <person>`); continue with
`--yes`. For a page still serving: ask the child for its cleanup or relay
the person's request to keep it — never stop the child's page from the
parent. Nothing asks the parent to act as the child or to record a review
nobody gave. Incoming human answers are never refused.

## Checks

`test/correction.test.ts`:

- the brief and take-up carry the assessment; `satisfied` is refused with
  `delegated-presentation-missing`, records nothing, and the last line
  carries the omission and `wfe delegate revise`; a wrong manifest is
  `result-superseded` even with yes; `--yes` records it; the satisfied
  delegation is still assessed for the report;
- a parent without work nodes is reminded when its completion completes the
  run; nothing recorded; `--yes` passes once;
- CLI (`wfe delegate show`, `dispose` exit 1, then `--yes`) and HTTP
  (`…/dispose` 409, `…/assessment`, `yes`, retransmission recorded once,
  `…/revise` on a closed delegation 422).
