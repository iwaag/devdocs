# Report 1 — pyagag: give agcode a tool seam (additive)

Status: **done**. `pyagag` suite green (191 passed, was 176).

`src/agag/agcode.py` is still one stdlib-only file; it grew from 661 to 750
lines. Nothing about the CLI's behaviour changed — the seam is library-side
only, and a test now pins that.

## What was added

### 1. The tool table is a parameter

```python
@dataclass(frozen=True)
class Tool:
    spec: dict[str, Any]        # the JSON tool spec sent on the wire
    func: Callable[..., str]    # called as func(base, **arguments)
```

- `DEFAULT_TOOLS` — the four built-ins (`read`, `write`, `list`, `run`), in the
  same order as `TOOLS_V0`, carrying the same spec dicts. `run(tools=...)`
  defaults to it, so every existing caller is unaffected.
- `tool_table(tools)` builds the name→Tool mapping and rejects duplicate names
  rather than silently shadowing one tool with another.
- `dispatch_tool(base, name, args, tools)` now takes that mapping. The
  module-level `_TOOL_FUNCS` constant is gone.
- `run()` sends `[t.spec for t in tools]` as the payload's `tools` and
  dispatches against the table. An empty sequence offers no tools at all.

The uniform signature is `func(base, **arguments)` — the working directory
first even for a tool that has nothing to do with the filesystem. One shape
keeps `dispatch_tool` free of special cases, and Step 3's `fetch` / `wait` /
`switch_view` / `show_image` will simply ignore the first argument.

### 2. `READONLY_TOOLS`

`(read, list)`. Permission *is* the tool set here. A read-only door is handed
this preset; there is no permission engine, no deny rule, and — the part that
matters for weak models — no forbidden option visible to attempt. Asking for
`write` under this preset comes back as `unknown tool: 'write' (available:
read, list)`, counted as a malformed call, with nothing written.

### 3. `system_suffix=`

Appended to the pinned `SYSTEM_PROMPT` after a blank line, via a new
`compose_system(working_dir, system_suffix)`. The working-directory sentence
stays first and unconditional — a test asserts a suffix naming a *different*
directory still leaves the real base ahead of it in the prompt. A blank or
whitespace-only suffix changes nothing. This is how today's `AGENTS.md` files
arrive in Steps 3 and 5, read from disk per request rather than fixed at
process start.

### 4. `stop=`

A zero-argument predicate checked between turns. True ends the run as
`outcome: aborted` with a new failure kind `cancelled`, keeping the usage and
turn count accumulated so far. The check sits at the top of the loop, so an
in-flight turn always completes — the run never abandons a request whose
tool results are already applied. `FAILURE_KINDS` grew by that one member.

## Prompt audit

`test_base_prompt_names_exactly_one_directory` renders the base prompt with a
sentinel directory and asserts:

- the sentinel appears exactly once, and
- every path-shaped token in the rendered text *is* that sentinel, and
- no operator-specific token (`agstudio`, the developer's username,
  `localhost`, `http`, `Users`, `home`) appears in the template.

The pre-existing byte-for-byte template pin and the no-ambient-reads source
audit still pass unchanged.

## Test additions (15 new tests)

Tool seam: default set is the V0 four in order; read-only preset offers only
`read`/`list` on the wire; read-only preset reports `write` as unknown and
writes nothing; a custom in-process tool is offered and dispatched with `base`
first; empty tool set; duplicate names rejected.

`system_suffix`: follows the pinned template; blank suffix inert; working-dir
sentence stays first.

Cancellation: stop-before-first-turn aborts with zero backend requests;
stop-between-turns keeps turn 1's usage and returns empty output;
always-false stop is inert.

CLI: `test_cli_defaults_are_the_four_tools_and_no_suffix` pins the wire payload
of `python -m agag.agcode` to exactly what it was before the seam existed.

One existing test changed: `test_dispatch_tool_maps_decode_error_without_raising`
now passes the mapping as `dispatch_tool`'s fourth argument.

## Out of scope, as specified

No MCP, no streaming, no session resume. `agcode.run()` is still one shot and
stateless; Step 5 makes cagent the memory rather than teaching agcode sessions.

## Docs

`README.md` gained a "Calling agcode in-process" section with a worked example
and one paragraph per argument — Tool Giving includes the usage information,
so a consumer does not have to read `agcode.py` to host an agent.
`docs/agent-config-v1.md` needed no change: the config contract is untouched.

## Notes for later steps

- `Tool.func` returning a non-string will reach the wire as-is; nothing
  validates it. If a Step 3/5 tool returns structured data, `json.dumps` it at
  the tool boundary the way `tool_run` already does.
- A tool that raises `TypeError` internally is reported as a malformed *call*
  (it is indistinguishable from a bad-arguments `TypeError` at the call site).
  Custom tools should convert their own errors into a returned error string.
