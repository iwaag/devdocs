# p8 Step 6 — the conversational experience, proven on the deployed entrance

Every trial went through the deployed entrance:
- messages posted with `wfe conversation post`, which is what the Front Desk's route does;
- the person's account named honestly as `Omni Agent (for Developer)`;
- Front served each one with ordinary WFE runs.

Times are message → first reply naming it. Evidence: the conversation store
(`wfe conversation show <id>`), the runs named below, and pj-agdev's ignored
`.local/project-migration-p8/` (notes, screenshot).

## Results

| Trial | Conversation / runs | Evidence |
| --- | --- | --- |
| Existing information | `p8-trial-a`; front `converse/run-001` | "What did playprey's latest run deliver?" answered in 50 s from playprey's run records: deliverables, unverified points, run and output links, and that Front read rather than verified. No specialist work started |
| Two-project request | `p8-trial-b`; front `converse/run-002`; forge `provide/run-008`; agsfx `create/run-002` | One message produced two delegate nodes added by adjustment. Children: forge (64×64 transparent bell PNG) and agsfx (chime < 1 s). Front then:<ul><li>combined both into one review page;</li><li>asked the Developer to choose;</li><li>disposed both `satisfied`;</li><li>completed with links to the files.</li></ul> |
| Mid-flight change | `p8-trial-b` #5 → front `converse/run-003` | "Make the chime lower and softer" arrived while agsfx waited for its confirmation. Front:<ul><li>linked #5 as `steer` of run-002;</li><li>instructed run-002 and answered agsfx's question with the Developer's words;</li><li>resumed agsfx. agsfx made new candidates (A5 880 Hz, E5 659 Hz); the Developer chose D.</li></ul>The original request (#3) and the revision (#5) both stay as said; run-002's instruction i1 names #5. Reply in 74 s |
| Material ambiguity | `p8-trial-c` #8 / #11 | The first glossary draft found no definition of "Shackle". Front marked it unconfirmed and asked; it did not guess. The answer (#11) was applied in the next run |
| New project | `p8-trial-c`; front `converse/run-004`, `run-006`; glossary `freework/run-001` | Asked for a terms project "unless an existing one suffices", Front first kept the glossary in its own `knowledge/` and said why. Told to separate it, Front:<ul><li>created `glossary` with `wfe create` (Gitea `developer/pj-glossary`, freework placed);</li><li>requested its first run;</li><li>had the glossary (21 terms with sources) integrated into `main`.</li></ul>2 min 21 s from message to result |
| Conversation responsiveness | `p8-trial-d` #17, #18 | While cagent `consult/run-006` (Front's child for #17) held the coordinator's default slot, #18 ("what is running now?") was answered in **18 s** by `converse/run-009` in the conversation lane. The reply listed the cagent work, the waiting parent and the held runs. #17's result came later (298 s) with `re #17`; neither reply was misattributed |
| Direct operator work | `p8-trial-f`; front `converse/run-011` | In 32 s:<ul><li>project context block `developer-preferences` set on forge (manager front-agent, editor provenance `front/converse/run-011`);</li><li>glossary `freework` intent changed with the base-checked save.</li></ul>Both read back through the same API the browser uses. Run management (instruct, answer, resume, disposition) is in trial b |
| Knowledge reuse | `p8-trial-e`; `p8-trial-h`; forge `provide/run-009` | "From now on, begin reports with a ≤3-line summary" became the system block `developer-preferences` (rev 1, evidence names message #20). The next results (#22, #29) follow it. A project used directly gets the same block: `wfe context show --workspace forge --workflow freework` lists the system and the forge blocks. forge's gear icon run delivered the SVG the forge block asks for, without being told in the request |
| Specialist question | `p8-trial-h`; forge `provide/run-009` | Front answered forge's result confirmation itself, because the Developer had left the check to it. It opened the PNG and recorded why it accepted; nothing was forwarded to the Developer |
| Recovery | `p8-trial-g`; front `converse/run-012` | During the turn's attempt the executor and the WFE service were both restarted. The executor adopted the live attempt by lane. The same message re-posted with its token was "already posted". Result: one run, one job, one attempt, one reply (18 s) |
| Delivery | `p8-trial-b` #15, `p8-trial-h` #29 | Replies carry links the Developer opens directly. Image and sound play in the browser: `…/wfe/api/projects/<p>/runs/<wf>/<run>/raw?path=…`. The Front Desk shows each reply with its run chip (screenshot `shots/frontdesk-trial-b.png`) |

Response times: 18–74 s for replies that need no specialist work (median 32 s);
4–5 min for results that include a specialist's work.

## Failures found by the trials, and their fixes

1. **A disk-wide search held the conversation slot for ten minutes** (`converse/run-002`). The guide named `docs/delegation.md` without saying where it was, and Front ran `find /`. Omni killed the `find` (intervention). The guide now points at `wfe delegate help` and records why (pj-front `377863f`).
2. **A question to the Developer stayed inside the run** (`converse/run-002` asked its kit confirmation with `wfe run ask` only). Guide fact: a question goes into the conversation too (`d2d0fa5`). The next question was posted as a conversation question (#9).
3. **Ready work beside a wait was held as interrupted.**
   - agsfx `create/run-002` and front `converse/run-002` asked, ended, and were held "interrupted" for report steps left ready. The answer could not continue them, and Observer opened eight recovery runs.
   - The rule's own wording said "ready and nothing waiting", but the code ignored waits. Fixed in WFE `5f55a91`.
   - That first fix changed how existing histories replay, and 8 runs read as inconsistent. In `dde93b8` the rule applies only to `attempt.end` entries marked `waits: defer-ready`, which are written since then.
   - The release `7b16e0c` had also changed the Zulip wording in decision evidence. One record written in that window (forge `provide/run-008`) was repaired, with the Developer's permission, by naming its recorded evidence on the entry (forge `9d11975`). Afterwards every run in every workspace reads as usable.
4. **A delivered file had no link that showed it.** The run-file API returned JSON, and the review pages had been stopped.
   - New read-only raw routes serve a project's or a run's file inline by type (`248726b`).
   - Guide fact: a result is delivered when the Developer can open it from the reply.
   - The next delivery (#16, #29) carried such links.

Checks after the fixes: WFE 420/420 plus the access tests; type check and build pass.
Deployed: `248726b`.

## Interventions (Omni)

- Killed `converse/run-002`'s runaway `find /`. It was a guide defect, fixed; no handoff.
- Repaired forge `provide/run-008`'s decision entry (record format regression of WFE `7b16e0c`, fixed in `dde93b8`); no handoff.
- Posted every trial message, answered as the person (as `Omni Agent (for Developer)`), and restarted services for the recovery trial.

Omni did not answer a specialist, dispose a delegation, create a project or
write a reply in Front's place.

## Cost

Front's attempts in these trials cost $3.87. The children cost:
- forge $0.67;
- agsfx $0.78;
- glossary $0.13;
- cagent: codex, no cost recorded.

Observer's recoveries, caused by defect 3, cost $1.26.

## Remaining limitations

- **One conversation slot.** A Front run that does long work itself holds the conversation lane, and a new turn waits behind it. Measured once: 45 s behind `converse/run-002`'s resume. Raising the lane's capacity needs a second lane executor thread; today one executor serves one attempt per lane.
- **A shell process an agent leaves behind keeps its attempt open** after the harness has finished (defect 1). Nothing ends the attempt on the harness's own end.
- **Definitions Front edits are saved, not committed** (glossary's intent). The same is true of the editor's Save.
- **The Front Desk shows some leftover legacy words** (`#front` in its header, the retired re-voice button). The side panels (progress, contexts) still read the relay.

Experiential feedback from the Developer is pending. Technical verification
does not depend on it.
