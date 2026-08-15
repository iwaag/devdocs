# Report 3 — agdevworld → agcode with native tools

Status: **done**. Assistant suite green (129 passed, was 125). Container
rebuilt; the live `front` chat ran on agcode, switched the view and fetched.

## The choice the plan asked me to record

`run_front` calls `agcode.run()` **in-process**, through a new
`chat.run_agcode()`. `run_harness` did not grow a tools parameter.

The reason is that the tools are Python callables. There is no argv, no
config file and no environment variable that can carry a closure to a
subprocess — a tools-capable `run_harness` would have had to re-invent MCP to
get them across, which is precisely the thing this step removes. Going
in-process also deletes the MCP subprocess, the two per-run environment
variables, and the stdout round trip.

`chat.py` stayed legible: `run_front`'s body is one `if agent.harness ==
"agcode"` branch, and everything the branch needs lives in two named functions
above it (`agcode_tools`, `run_agcode`). `run_agcode` returns a `HarnessResult`
with `{**identity(agent), **result.meta}`, so the caller — the record writer,
the 502 path, the browser — sees exactly the shape it saw before.

## The four tools

`tool_service.py` stays the MCP entry point for the `sonnet` profile, and I
did **not** lift the bodies out of it. Instead the file grew two small
functions and one signature change, so both doors call the same
implementation:

- `call_tool(name, args, *, base_url=None, actions_file=None)` — the per-run
  context now arrives as arguments, falling back to the two environment
  variables the MCP subprocess is started with. In-process callers pass it
  explicitly, because mutating `os.environ` in a threaded server is a race —
  the same reason `chat.py` already used `dataclasses.replace` rather than
  `os.environ` for the subprocess path.
- `messages_api_specs()` — the four specs with `inputSchema` renamed to
  `input_schema`. That rename is the entire difference between the MCP and
  Messages API tool vocabularies here; the schemas are the same objects.
- `result_text(reply)` — one MCP reply flattened to the string agcode's `Tool`
  contract wants. The `isError` flag is dropped: the bodies already name their
  own failures ("fetch refused: …", "unknown view: …"), so the flag carries
  nothing the text does not.

`chat.agcode_tools(tool_base_url=…, actions_file=…)` binds those three
together into four `agcode.Tool`s. Each ignores the working directory it is
handed first — none of the four touches the filesystem except the actions
file, which arrives as an absolute path.

Lifting the bodies would have given `fetch` two implementations to keep in
step, one per profile. One implementation with two entrances is the smaller
long-term object, and the new tests assert both entrances agree.

## Other changes

- `agents.toml`: `[profiles.local] harness` → `"agcode"`.
- `_launch_conditions`: the opencode branch is gone. agcode never reaches the
  function; the remaining non-claude branch (the `fake` stub) still runs from
  the project root.
- Deleted `opencode.json`.
- `overlay.py`: dropped `[local.harness.opencode]` and the
  `AGENT_HARNESS_OPENCODE_COMMAND` read. **No `[local.harness.agcode]` block
  was added** — its default command is `sys.executable`, already the
  interpreter importing `agag`.
- `assistant/Dockerfile`: dropped `opencode-ai@1.18.10` from the npm install
  and `opencode.json` from the `COPY`. The image is measurably smaller and one
  npm dependency lighter; the node base stays, because claude_code still
  arrives that way.
- `compose.yaml` + `overlay.py` default: `AGENT_PROVIDER_OLLAMA_BASE_URL` lost
  its `/v1` suffix (see report2 — agcode posts to `{base_url}/v1/messages`).
- `README_DEV.md`, `assistant/GUIDE.md`: harness table, files list, profile
  paragraph, image contents.
- `assistant/uv.lock`: pyagag pin `00a89808` → `b705fd97`.

`AGCODE_MAX_TURNS = 60`. The service deadline is the real bound and is passed
as `deadline_s`; the turn budget is only a runaway backstop. A tight one would
end a legitimately slow answer, because this agent polls (`wait`, then `fetch`
again).

## Tests

`assistant/tests_py/test_chat.py` — five new: the four tools carry Messages
API schemas; `switch_view` appends to the run's actions file; two runs with
different actions files do not interfere (the anti-`os.environ` property);
refusals come back as their own text; `run_front` on agcode never calls
`run_harness`, passes the native model to the wire while keeping the canonical
one in the record, and works in a temp directory that is gone afterwards; a
failed agcode run is still a `ChatFailure` carrying its outcome.

`assistant/tests_py/test_tool_service.py` — four new: the spec rename is a
rename; `result_text` flattens error replies too; explicit context beats the
environment; the environment still serves the MCP subprocess.

One existing test renamed: `test_opencode_runs_from_the_project_root…` →
`test_non_claude_harness_runs_from_the_project_root…`, now exercised with
`fake`.

## Live check

`docker compose build assistant && docker compose up -d assistant`, then one
`POST :8091/api/chat`:

> "Switch the screen to the tasks view, then fetch /api/autolab/nodes and tell
> me the node names it lists."

```json
{"reply": "The screen is now on the **tasks** view.\n\nThe `/api/autolab/nodes`
  endpoint lists two nodes:\n\n1. **agstudio** — reachable (status 200)\n
  2. **agautolab1** — reachable (status 200)",
 "actions": [{"action": "switch_view", "view": "tasks"}],
 "run": {"role": "front", "profile": "local", "harness": "agcode",
         "provider": "ollama", "model": "ollama/qwen3.6:35b-a3b-coding-nvfp4",
         "outcome": "done"}}
```

Both tools fired for real: the `switch_view` action reached the browser
channel (it is in `actions`, which is what `chatPanel.ts` applies), and the
node names came from the live passthrough, not from the model.

The generated in-container overlay was checked and carries no opencode block
and the un-suffixed ollama base URL.

`/records/<id>.agent.jsonl` is now agcode's own verbatim capture — its first
line is `{"record": "meta", "format": "agcode-transcript-v1", …,
"base_url": "http://host.docker.internal:11434", "working_dir":
"/tmp/agdevworld-agent-…"}`, five lines for two request/response pairs. That
is a genuine improvement over what the same path used to hold: opencode's
stdout event stream. Because this door is in-process, `transcript_path` is
agcode's real wire capture, not the harness-stdout capture described in
report2's second note.

## Deus Ex Machina note

Did the agcode migration for agent `front` (agdevworld assistant) — handoff
candidate.
