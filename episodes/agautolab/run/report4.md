# run rework — Step 4 report: guide check

Done. `agent/guides/run_supercoder/guide.md` reviewed against the new flow and
two edits made; `agent/guides/run_coding/guide.md` is deleted with the code
path that loaded it.

## Findings and what was done

| Finding | Action |
|---|---|
| Typo: "crete" | fixed → "create" |
| It never said **where** `report.md` goes, so the handler could not find it | the line now names the workspace directory the prompt gives above it |
| It never mentions the chatlog | left alone — the placement lines cover it, exactly as in the superdirector flow |
| `success.flag` | absent from the guide already; dropped from the flow in Step 3. `report.md` alone is the done signal — one signal, not two |

Nothing else was deleted. The final guide:

```
The folder "main/" contains current source codes of the project, which is likely your main interest to complete the work.
The folder "direction/" contains concept documents of the project,
 which you only read when you really need to check the concept of the project.
The folder "devlog/" contains plans and reports of past works, which you only read when you need to log of the development.
Being empty means the project has just started.

Do the work following developer's request.
If the developer agreed that the task was done, create "report.md" in the workspace directory this prompt names above.
```

The handler side of that sentence is the prompt's second placement line:
`Write "report.md" — and any other file this guide asks for — into "<absolute workspace path>".`

## One gap found and deliberately not closed

The guide never tells the supercoder that **its reply is relayed to the
developer** — `mission_superdirector/guide.md` opens with exactly that line
("Your reply to this conversation will be sent to the developer."), and a
`run-` topic is a conversation now, so the same is true here. Adding it was
outside the three edits the plan listed, so it is reported rather than done.
Suggested addition, if wanted:

```
The topic is supposed to be about one task of the current mission.
Your reply to this conversation will be sent to the developer.
```

## `run_coding/guide.md`

Deleted along with `work_prompt`/`run_work` in Step 3. It described the old
`.local/work/` contract (`work.md` in, `report.md` + `success.flag` out),
none of which survives. The `coding` role itself is kept per the plan.

## Verification

`uv run pytest -q` — 163 passed. Guide loading is exercised by
`test_a_serving_runs_the_supercoder_in_the_project_with_its_workspace`
(the prompt ends with the guide text) and `test_guide_refuses_to_start_without_the_file`.
