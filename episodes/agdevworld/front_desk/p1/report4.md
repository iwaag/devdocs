# Step 4 — verification, deployment, and one completed `ghtrends` task

## Tests and build

| suite | result |
|---|---|
| `agdevworld` `npm run build` | passes (tsc + vite) |
| `agdevworld/agentroom` `uv run pytest` | 149 passed (19 new in `test_frontdesk.py`) |
| `agfront` `uv run pytest` | 29 passed (8 new: character_talk selection for direct posts and callbacks, ordinary front routing, guide presence) |
| `agautolab` `uv run pytest` | 219 passed, 1 failed — `test_intro.py::test_the_intro_is_posted_for_this_instance` fails at HEAD before this episode (`'Client' object has no attribute 'whoami'`, from the pyagag roster update); unrelated, left as is |

Covered as the plan asked: character_talk selection for direct posts and
callbacks; ordinary front routing; history restoration (held / read /
unknown, resolved topics included); destination checks (id shape, length,
selfnote, unconfigured); duplicate sends (submit token); uncertain outcome
without retry; in-place resume of a ✔'d conversation.

## Deployment

- `docker compose up --build -d web` → `:8090` 200, new bundle.
- `launchctl kickstart -k` on `com.agdev.agentroom` (35 s sweep, then
  `/frontdesk` live, chat configured) and `com.agdev.agfront-zulip`
  (checked idle first via `/inflight`).
- Front's introduction re-posted to `#agents` with the Front Desk paragraph.
- No plist change: the desk reads with `AGENTROOM_ZULIP_ENV`, writes with
  `AGENTROOM_CHAT_ZULIP_ENV`, both already there.

Browser checks on the deployed `:8090` (CDP driver, `.local/deskshot.mjs`):
images, long replies paged (`1/2` at 1600×1000, `1/5` at 1000×700), history
panel, Japanese IME composition-Enter not sending, emoji intact, resizing.

## The live conversation — `#front` › `front-desk-20260908-1600`

Seven `character_talk` runs, all on claude_code / sonnet-5, from
`agfront/.local/agent/character_talk/run-0001…0007.json`:

| total cost | total run time | runs |
|---|---|---|
| $0.875 | 197 s | 7 |

The exchange, as posted (message ids):

1. **5171** Developer, from the screen: greeting. **5173** Front, in the gyaru
   voice (`やっほー！はじめましてー😊✨ …`).
2. **5174** Developer: what Front does all day. **5176** Front answered in
   character but **leaked a reasoning preface** ("This is just small talk —
   … Front から開発者への返信："). Guide fixed the same minute (the whole
   output is the dialogue; agfront `23012f1`), read per run, no restart.
3. **5177** Developer: *run ghtrends once and see it through; approvals are
   pre-given.* **5183** Front reported it had posted the mission (5180, in
   plain professional English, after its `[selfnote][rootchat]` 5179) into
   `#pj-ghtrends` › `workplan-trend6` and **exited**.
4. **5182** autolab named Front: `failed during superdirector: run_role()
   got an unexpected keyword argument 'extra_meta'`. Front was **called back**
   and reported at home (**5185**), in character, offering three options.
5. The failure was real and outside the plan: agautolab's own `run_role`
   wrapper (`role_run.py`) never accepted `extra_meta` after commit
   `61510bd` made the listener pass it, so **every listener-started autolab
   run on agstudio had failed before the harness started**. Fixed with a
   regression test (agautolab `953127d`, pushed; listener kickstarted).
   This is a Deus Ex Machina moment by the terms doc: the Front Desk found
   it and the Omni Agent repaired it.
6. **5188** Developer, from the screen: *fixed, retry — option 1.* **5189**
   Front to autolab, professional English; **5195** autolab wrote the plan
   and task; **5198** Front started task 1 in `#work-g-13` ›
   `workrun-task1-g-13` and reported (**5200**), then exited again.
7. **5203** autolab: task complete, named Front; topic resolved (**5204**,
   a system notice — not speech). Front called back, reported (**5207**):
   repository, figures, files, commit id — in character, with the API URL
   rendered as a link chip.

Front also posted a plain-language record into `#front` ›
`routine-ghtrends` (**5206**) — the stray-report pattern the routines board
already flags. A guide line now forbids it (agfront `120a872`).

### Verified independently of Front's words

- `agautolab/.local/projects/ghtrends/main`: commit `a99625f` "Add
  microsoft/markitdown trending repo summary (2026-09-08, #2 on GitHub
  trending)" — `index.md` +1 row, `repos/microsoft-markitdown.md` +32 lines,
  one commit; `git ls-remote origin main` = `a99625f`, so it is pushed.
- The summary names `https://github.com/trending`, rank #2, 2026-09-08,
  MIT, and says its figures come from
  `https://api.github.com/repos/microsoft/markitdown`.
- `work-g-13` holds only `✔ workrun-task1-g-13`; autolab's report says
  Plane task Done and the devlog recorded.
- The Front Desk received the result as the latest reply, with the status
  `answered`, and the history panel shows every turn with acks as receipt
  lines.

### The two writing styles, inspected

- To the developer (5173, 5183, 5185, 5200, 5207): Japanese, gyaru, emoji
  in nearly every line, exact names and ids kept plain.
- To agents (5180, 5189, 5198): English, professional, no emoji, states the
  request, the evidence and what is expected back. 5206 (the stray record)
  likewise.

## Fixes made from what the live run showed

- relay: strip the `@**Developer**` handoff mention from shown posts
  (agdevworld `ab23a6e`).
- guide: no reasoning preface (`23012f1`); never post into
  `routine-<name>` (`120a872`).
- scene: Phaser 4 geometry mask on a container child did not clip — the
  history panel now crops per post; link chips clamped to the box; a
  trailing backtick no longer ends up in a bare-URL chip (`e6c1b0c`).
- agautolab: `extra_meta` accepted by the wrapper (`953127d`).

## Remaining issues

- `agautolab1` (the VM) was not redeployed; if it carries `61510bd` it has
  the same TypeError until the playbook runs.
- Zulip's ✔ glyph renders as a wide fallback glyph in the dialogue font.
- At very small widths the page label can touch the last chip row.
- `agautolab/tests/test_intro.py` failure predates this episode.
- Cost per casual exchange is ~$0.06–0.12; the model is front's and can be
  moved in `.local/agents.local.toml` under `[roles.character_talk]`.
