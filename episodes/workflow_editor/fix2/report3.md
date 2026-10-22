# workflow_editor fix2 — Step 3: contracts

## Where each fact lives

| Fact | Place | Revision |
| --- | --- | --- |
| Intake is on unless `intake: {zulip: ""}`. New files show the setting. Round trips and the digest keep the two apart. A wrong type is unreadable. An opt-out is about what the workflow's runs are for, not whether the project takes work | `docs/intake.md` (the one statement) | agdevworld `0d49456` |
| The YAML field's meaning, `intake-mode`, `delegate-no-intake`, and intake always being in the definition projection | `docs/contract.md` | `0d49456` |
| A target accepts requests unless it opts out | `docs/delegation.md` (dispatch resolution, validation table) | `0d49456` |
| A refusal concerns the target's definition or this host, not whether the project takes work. `wfe request` still reaches the workflow as a plain request, and nothing then brings its questions or result back | `wfe delegate request --help` (the tool's usage) | `0d49456` |
| Front: a refused delegation is not a project refusing the work, and work entrusted another way stays Front's to bring back (cites run-045) | Front's guide `agent/guide.md`, *What counts as done* | front `8cb5923` |
| System overview: every workflow has the intake unless it opts out; converse/discuss do; why run-045 failed | `devdocs/README_DEV.md`, delegation section | devdocs (this commit) |

The shared run guide (agautolab `wfrun/guide.md`) already says that the
child is created by the target workflow's intake, and it does not repeat
the refusal rules. It is unchanged, because the refusal is the tool's
fact and its `--help` holds it.

## Front's handling of the incident

Front had already used the right fallback (a plain request) and had
recorded it honestly in its run (`run.adjust`: "dispatched as a plain
request … instead"). It went wrong in two ways:

1. **It presented a capability as a policy.** Front told the Developer
   that "aghangout does not accept delegations from other projects". The
   refusal said only that two workflow files lacked an intake declaration.
   The fix removes the cause. The `--help` text and Front's guide now say
   what a refusal is and what it is not.
2. **It handed back responsibility.** Front told the Developer: "I can't
   follow the result; check aghangout's run when it is done." Front's
   guide already said that the request is served when the Developer has
   the result. The new bullet names this case: work entrusted another way
   stays Front's to bring back. It prescribes no command sequence. Front
   decides how (another target, a later check, or telling the Developer
   plainly what will and will not bring it back).

No other guidance was added.
