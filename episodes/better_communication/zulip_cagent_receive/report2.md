# zulip_cagent_receive — Step 2 report: `incident.py`

Date: 2026-08-12. Status: **complete**. `cagent/window/incident.py` records one
incident report as one local file and lists recent ones. Stdlib only, no
dependencies, runnable straight from the superproject root.

## The shape

The plan's suggested shape was kept, with the reason it survived: an agent
writes these and a human reads them, and nothing else parses them.

```text
.local/cagent/incidents/<UTC timestamp>-<slug>.md   (git-ignored via /.local)

---
id: 20260812T063958Z-cagent-said-agpc-was-reachable-and-converged
time: 2026-08-12T06:39:58Z
reporter: zulip:9 Omni Agent
source: zulip-dm
ref: message 38
---

<the report verbatim, as the reporter worded it>
```

Interface:

```bash
uv run cagent/window/incident.py -i "description"            # prints the path
uv run cagent/window/incident.py -i "..." --reporter "zulip:8" \
    --source zulip-dm --ref "message 41"
uv run cagent/window/incident.py --list        # 10 most recent, newest first
uv run cagent/window/incident.py --list 5 --json
```

Decisions worth naming:

- **The body is verbatim.** No summarizing, no schema. The window is told to
  record what it was told, so a later reader sees the complaint, not the
  agent's paraphrase of it.
- **The slug is derived, the id is the filename stem.** Sorting by filename
  is sorting by time, which is why `--list` needs no index file.
- **Same-second collisions get a `-2` suffix** rather than overwriting.
- **Frontmatter values are flattened to one line**, so a multi-line
  `--reporter` cannot forge a second `---` fence or an extra key.
- `CAGENT_INCIDENT_DIR` overrides the directory; it exists for tests, and
  the default is `<superproject>/.local/cagent/incidents`, next to the rest of
  cagent's local runtime state.
- `--list` on an empty or absent directory prints `no incidents recorded`
  rather than failing, because the window will ask that question of a fresh
  install.

## Verification

Two incidents recorded by hand from the superproject root — one phrased as a
wrong-answer report about a node's reachability, one about a wrong address in
a drift table — each printing its path, and `--list` showing both, newest
first, with reporter/source/ref and the first body line. (The recorded bodies
name real hosts and addresses, so they stay in the ignored directory and are
not quoted here.)

`git check-ignore -v .local/cagent/incidents` → `.gitignore:2:.local`.

`cagent/tests/test_incident.py` (9 cases, loaded by path since the script is
deliberately outside the `cagent_api` package) pins the slug bounds, the
write/read round trip, the frontmatter-injection flattening, the same-second
collision, `--list` ordering and truncation, the empty-directory answer, and
the CLI's refusal of an empty report. `uv run --project cagent pytest`: 9
passed for this file.

## Notes

- Nothing here is wired to the window yet; Step 3 gives the window the
  permission to run exactly this script and the guide sentence that tells it
  when to.
